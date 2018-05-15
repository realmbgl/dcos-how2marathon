## networking


### host networking

*marathon.json*, showing *host* networking configuration, the port gets auto assigned and is available to the executable via environment variable.
```js
{
  "id": "service",
  "container": {
    "type": "MESOS",
    ...
  },
  "networks": [
    {
      "mode": "host"
    }
  ],
  "portDefinitions": [ {
      "name": "web",
      "port": 8080,
      "protocol": "tcp",
      "labels": {
        "VIP_0": "service:8080"
      }
  } ],
  "cmd": "<executable> $PORT0"
}

```

### virtual networking

*marathon.json*, showing *virtual networking* configuration. Virtual networking provides an ip per container, the port is the container port.
```js
{
  "id": "service",
  "container": {
    "type": "MESOS",
    ...
    "portMappings": [
        {
            "containerPort": 8080,
            "servicePort": 8080,
            "protocol": "tcp",
            "labels": {
              "VIP_0": "service:8080"
            }
        }
    ]
  },
  "networks": [
    {
      "name": "dcos",
      "mode": "container"
    }
  ],
  "cmd": "<executable> $PORT0"
}

```

### moustache templating, virtual networking on/off

*marathon.json*, showing how to make host or virtual networking a configuration option.
```js
{
  "id": "service",
  "container": {
    "type": "MESOS",
    ...
    {{#service.virtual_network_enabled}}
    "portMappings": [
        {
            "containerPort": 8080,
            "servicePort": 8080,
            "protocol": "tcp",
            "labels": {
              "VIP_0": "service:8080"
            }
        }
    ]
    {{/service.virtual_network_enabled}}
  },
  {{#service.virtual_network_enabled}}
  "networks": [
    {
      "name": "dcos",
      "mode": "container"
    }
  ],
  {{/service.virtual_network_enabled}}
  {{^service.virtual_network_enabled}}
  "networks": [
    {
      "mode": "host"
    }
  ],
  "portDefinitions": [ {
      "name": "web",
      "port": 8080,
      "protocol": "tcp",
      "labels": {
        "VIP_0": "service:8080"
      }
  } ],
  {{/service.virtual_network_enabled}}
  "cmd": "<executable> $PORT0"
}

```

