## multi tenancy


### service names scoped by folders/groups

*marathon.json*, showing id set to a foldered service name.
```js
{
  "id": "/org1/space1/service",
  "container": {
    "type": "MESOS",
    ...
  },
  ...
  "cmd": "<executable> $PORT0"
}

```

### mapping fully qualified service name to vip name

*marathon.json*, showing foldere service name used in id and vip.
```js
{
  "id": "/org1/space1/service",
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
      ...
      "labels": {
        "VIP_0": "/org1/spaces1/service:8080"
      }
  } ],
  "cmd": "<executable> $PORT0"
}

```

In the actual vip name the slahes will be removed.

```
orrg1space1service.marathon.l4lb.thisdcos.directory:8080
```

### moustache templating, configurable service name

As we learned the service name can contain slashes. You can use a template variable {{service.name}} aside from setting the id only in places that can deal with the slashes. The vip setting for example can handle it.

*marathon.json.moustache*, ...
```js
{
  "id": "{{service.name}}",
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
      ...
      "labels": {
        "VIP_0": "{{service.name}}:8080"
      }
  } ],
  "cmd": "<executable> $PORT0"
}

```


