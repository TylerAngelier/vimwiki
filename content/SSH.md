# SSH

# Multiple similiar entries in ssh config

>  HostName
>     Specifies the real host name to log into.  This can be used to
>     specify nicknames or abbreviations for hosts.  If the hostname
>     contains the character sequence ‘%h’, then this will be replaced
>     with the host name specified on the commandline (this is useful
>     for manipulating unqualified names).

```
Host XXX1 XXX2 XXX3
  HostName %h.YYY.com
  
# OR this is the host and hostname match
Host XXX1 XXX2 XXX3
  HostName %h
```

Reference: https://unix.stackexchange.com/a/61666
