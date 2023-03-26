# PTO Schedule

Jan 12-13 (2)
March 29-31 (2)
May 19 (1)
June 20-21 (2)
September 11 (1)
November 13-22 (8)

Total: 16 days PTO used

# IDCS

Useful information about the differerent Administrator Roles: https://docs.oracle.com/en/cloud/paas/identity-cloud/uaids/understand-administrator-roles.html

# Multi tenant Kubernetes

High level required steps:
* Need to [remove](remove.md) all references to the `DEFAULT` namespace except for `PssNamespaceUtil`.
	* This means when we change the logic of getting the system namespace, it is enforced throughout the entire codebase. 
	* Low-Medium code changes required. 
 * Create system namespace at the top of `SystemResourceService`
	 * Add label/annotation that indicates the `nodeSelector` value to be used by the webhook applications.
 * Validate char limits on
	 * subscriber (10)
	 * env class (5)
	 * system (10)
	 * service names (27)
 * Create calico network policies for each system namespace
 * Make the required changes to the `releaseName` strategy
	 * Hard to be backwards compatable... If we're on the old namespace use the old 
 * Write and deploy two Admission Webhook applications
	 - Need to write a Mutating Admissions Webhook
		- Adds the `nodeSelector` if not present based on the namespace.
	- Need to write a Validating Admission Webhook
		- Validates the `nodeSelector` is added to any pod scheduled in a PSS namespace.
- Migration
	- Provision the entire system namespace and `default` namespace for non-migrated systems. 
	- Manual DNS change to point to the new load balancer. 
		- Use the same database under the hood.

## Naming
Can try to move the namespace but it's really nasty... Basically updating the Helm secret then finding all K8 resources and updating the `metadata.namespace` value manually (could _probably_ do it in code)..
https://blog.random.io/moving-helm-release-to-another-kubernetes-namespace/
Can't stand up everything in the new namespace because Helm won't allow it though...
```
"output": "Error: rendered manifests contain a resource that already exists. Unable to continue with install: ClusterRole \"traefik-ingress-controller\" in namespace \"\" exists and cannot be imported into the current release: invalid ownership metadata; annotation validation error: key \"meta.helm.sh/release-namespace\" must equal \"System_alpha-tangelie-dev-default\": current value is \"default\"\n",
```

Can set a different `release-name` to install the same chart multiple times (even in the same namespace). 
ReleaseName can only be `53` characters...
We need to add the fqSystemName to the release to ensure it's unique. How many characters can that be?
Subscriber Name: 10 char
System name: 10 char

```bash
System_<env>-<sub>-<sys>
# 9 static chars
# 5 = env
# 10 = sub
# 10 = sys
# Total: 34 char

# 53-34=19
# Adding a hyphen between system name and service name gives us 18 char

```
Can't do fqSystemName.. So if we reserve `26` char for the service name, that leaves us with `27` char for the identifier:
```bash
# 5 = env
# 10 = sub
# 10 = sys
# 27 = service-name
# 1 = hypen
<env><sub><system>-<service-name>
```


## Using Extensible Admission Controller

- Need to write a Mutating Admissions Webhook
	- Adds the `nodeSelector` if not present based on the namespace.
- Need to write a Validating Admission Webhook
	- Validates the `nodeSelector` is added to any pod scheduled in a PSS namespace.

General notes:
* Use a `namespaceSelector` to limit these controllers to only Platform and System namespaces.
	* https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/#matching-requests-namespaceselector
* Could use this pattern to enforce resource limits too.
	* https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/#example-of-idempotent-mutating-admission-webhooks
* Can likely only look only at `pod` `CREATE` and `UPDATE` operations
	* Need to inject `nodeSelector` on pods. Other resources like `Deployments` create the pods and it seems safer to inject it at the pod level to ensure every.
	* Maybe targeting a `Deployment` is better?
		* Less chatty
		* What about `statefulset`, `daemonset` arbitrary pods, etc?
* How to handle the TLS requirements?
	* Can use a `Service` to route the requests
	* Can Istio's sidecar handle this?
	* The helm chart for this admission controller can include the certificate?
	* Should run the `Service` and the pods on port `443`.
	* [Generate the cert with Helm](https://medium.com/nuvo-group-tech/move-your-certs-to-helm-4f5f61338aca)
* Need somewhere for this software to run...
	* Can put it in the `Platform` namespace but likely this software will need to manage that namespace too. Is there a chicken before the egg problem here? Can the controller manage the pods of itself (new pods)? 
		* If only runs on the `Platform` namespace, those nodes **cannot** go down as all pod creation would be rejected. 
	* Just put it in its own namespace and it can be scheduled on any node? Might be good enough for MVP.
		* Argubably this makes the most sense.. This software is part of how we manage PSS OKE clusters (think `kube-system`).
* Do we need a kill switch?
	* Need to be able to **very quickly** turn off these controllers in case of a bug that impacts production.
		* Validate with leadership + security it is better to have pods running in wrong nodes than to have a production outage.
		* "Turn off" is likely always return an `OK` response.
  

```bash
pkcs12 -export -in /secret/webhook-cert.pem -inkey /secret/webhook-key.pem -name webhook -out /shared-config/webhook.p12 -passout pass:
```

# SQL Scripts

## Drop PSS

```sql
set serveroutput on;
declare  
  cursor c_session_kill(p_schema_name varchar2) is  
  SELECT        'ALTER SYSTEM KILL SESSION '''||sid||','||serial#||''' IMMEDIATE' as kill_session_cmd  
  FROM v$session  
  where username = upper(p_schema_name);  
  
  l_count NUMBER := 0;  
  
  cursor c_schemas is  
     select 'pss_migration_data' as schema_name from dual  
     union all  
     select 'pss_migration_config' as schema_name from dual  
     union all  
     select 'pss_migration_system' as schema_name from dual  
     union all  
     select 'pss_migration_util' as schema_name from dual  
     union all  
     select 'pss_data' as schema_name from dual  
     union all  
     select 'pss_config' as schema_name from dual  
     union all  
     select 'pss_system' as schema_name from dual  
     union all  
     select 'pss_apex' as schema_name from dual  
     union all  
     select 'pss_application' as schema_name from dual;  
begin  
  
  for l_schema in c_schemas loop  
    dbms_output.put_line('Attempting to drop schema ' || l_schema.schema_name || '.');  
    begin  
      for session_rec in c_session_kill(l_schema.schema_name) loop  
        dbms_output.put_line('Killing session with cmd: ' || session_rec.kill_session_cmd);  
        execute immediate session_rec.kill_session_cmd;  
      end loop;  
      execute immediate 'DROP USER ' || l_schema.schema_name || ' CASCADE';  
      dbms_output.put_line('Dropped schema ' || l_schema.schema_name || '.');  
    exception  
      when others then        dbms_output.put_line('Failed to drop schema ' || l_schema.schema_name || '.');  
    end;  
  end loop;  
  
  -- clear admin changelog  
  delete from DATABASECHANGELOG;  
  commit;  
  
  -- Make sure all the schemas got deleted  
  select count(*)  
  into l_count  
  from dba_users  
  where username like 'PSS_%';  
  
  if l_count != 0 then  
    dbms_output.put_line('Failure! Schemas still exist.');
  else 
    dbms_output.put_line('Completed Successfully!');  
  end if;  
  
end;  
/
```

