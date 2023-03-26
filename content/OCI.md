# OCI

[Oracle Cloud Infrastructure](https://www.oracle.com/cloud/)

# OCIR - Container Registry

## Login

Login to a specific region:

```bash
# Format is <region-key>.ocir.io
# Phoenix
docker login phx.ocir.io
# Ashburn
docker login iad.ocir.io
# Langley
docker login ocir.us-langley-1.oci
# Luke
docker login ocir.us-luke-1.oci
```

* Username will be in a format like `<tenancy-namespace>/<username>`. 
  * **Note**: Username can be a bit "odd" as if it is a Oracle Identity Cloud Service account, such as through IDCS, then we will supply a value like `oracleidentitycloudservice/<username>`.
* Password will be an OCI [auth token](https://docs.oracle.com/en-us/iaas/Content/Registry/Tasks/registrygettingauthtoken.htm).

[Official Documentation](https://docs.oracle.com/en-us/iaas/Content/Functions/Tasks/functionslogintoocir.htm)

## Push an Image

Build your image
```bash
docker build -t foobar .
```

Tag the image
```bash
docker tag foobar phx.ocir.io/axkxa1g0qepm/foobar:latest
```

Push the image
```bash
docker push phx.ocir.io/axkxa1g0qepm/foobar:latest
```

Or in a bit more of a script-able way...

```bash
export oci_registry=phx.ocir.io
export oci_namespace=axkxa1g0qepm
export image_name=foobar
export image_tag=latest
export oci_tag=${image_tag:=latest}
export oci_image_path=$oci_registry/$oci_namespace/$image_name:$oci_tag

docker login $oci_registry
docker build -t $image_name:$image_tag .
docker tag $image_name:$image_tag $oci_image_path
docker push $oci_image_path
```
