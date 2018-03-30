## edge-lb


### pool config without tls

```
{
  "apiVersion": "V2",
  "name": "myproxy",
  "count": 1,
  "haproxy": {
    "frontends": [
      {
        "bindPort": 8080,
        "protocol": "HTTP",
        "linkBackend": {
          "defaultBackend": "myservice"
        }
      }
    ],
    "backends": [{
      "name": "myservice",
      "protocol": "HTTP",
      "services": [{
        "endpoint": {
          "type": "ADDRESS",
          "address": "service.marathon.l4lb.thisdcos.directory",
          "port": 8080
        }
      }]
    }]
  }
}
```

### pool config with tls

```
{
  "apiVersion": "V2",
  "name": "myproxy",
  "count": 1,
  "autoCertificate": true,
  "haproxy": {
    "frontends": [
      {
        "bindPort": 8080,
        "protocol": "HTTPS",
        "certificates": [
          "$AUTOCERT"
        ],
        "linkBackend": {
          "defaultBackend": "myservice"
        }
      }
    ],
    "backends": [{
      "name": "myservice",
      "protocol": "HTTPS",
      "services": [{
        "endpoint": {
          "type": "ADDRESS",
          "address": "service.marathon.l4lb.thisdcos.directory",
          "port": 8080
        }
      }]
    }]
  }
}
```
