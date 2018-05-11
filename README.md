# Developing Enterprise Grade Marathon Services

This guide shows by sample how to develop enterprise grade marathon services supporting the following.

- [universal containerizer (ucr)](ucr.md)
    - why ucr
    - cmd and fetch
    - cmd and image
    - cmd and endless loop
    - using container console, interactive container exploration


- [service account](service-account.md)
    - when do you need a service account
    - create service accout and service account secret
    - configure the service with service account and service account secret
    - moustache templating


- [multi tenancy](multi-tenancy.md)
    - service names scoped by folders/groups
    - mapping fully qualified service name to vip name
    - moustache templating, configurable service name
 
 
- [networking](networking.md)
    - host network
    - virtual network
    - moustache templating, virtual networking on/off


- [tls](tls.md)
    - creating a certificate request
    - a tls service enpoint implementation
    - a tls service endpoint implementation requiring client authentication
    - moustache templating, tls on/off, client authenticaton on/off


- [healthchecks](healthchecks.md)
    - without tls
    - with tls
    - with tls and client authentication
    - moustache templating, tls on/off, client authenticaton on/off


- placement constraints


- volumes
    - local persistent volumes
    - external persistent volumes
    

- [edge-lb](edge-lb.md)
