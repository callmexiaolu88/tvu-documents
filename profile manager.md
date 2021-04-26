# Register file construction

| filed name | description              | sample                                                       |
| ---------- | ------------------------ | ------------------------------------------------------------ |
| module     | module name              | module="AutoRecord"                                          |
| type       | the format of data filed | "type": "json"  /<br />type: "yaml" /<br />type="toml"       |
| env        | environment variable     | [{"name":"PACK_PID", "type":"string"}, {"name":"PACK_NAME", "type":"string"}] |
| data       | profile template         | "{\"TargetKey\":\"\",\"RemoteDirectory\":\"\",\"DeleteAfterUpload\":false}" |

```json
{
    "module":"AutoRecord",
    "profileid":"BC2B7511-7DEE-45D8-BD30-C11D1720B1DF",
    "env":[
        {
           "name":"PACK_PID",
           "type":"string"
        },
        {
           "name":"PACK_NAME",
           "type":"string"
        }
      ],
    "data":"{\"TargetKey\":\"\",\"RemoteDirectory\":\"\",\"DeleteAfterUpload\":false}"
}
```



# Preserve file construction

| filed name | description                   | sample                                                       |
| ---------- | ----------------------------- | ------------------------------------------------------------ |
| module     | module name                   | module="AutoRecord"                                          |
| name       | profile name                  | name="default"                                               |
| version    | profile version               | version="v1"                                                 |
| type       | the format of data filed      | "type": "json"  /<br />type: "yaml" /<br />type="toml"       |
| env        | environment variable          | [{"name":"PACK_PID", "type":"string"}, {"name":"PACK_NAME", "type":"string"}] |
| data       | profile template              | "{\"TargetKey\":\"\",\"RemoteDirectory\":\"\",\"DeleteAfterUpload\":false}" |
| enable     | enable current profile or not | enable=true                                                  |

```json
{
    "module":"AutoRecord",
    "name":"default",
    "version":"1",
    "enable":true,
    "profileid":"BC2B7511-7DEE-45D8-BD30-C11D1720B1DF",
    "templateid":"62DA396E-BAD8-466E-9885-F62C660FA144",
    "data":"{\"TargetKey\":\"\",\"RemoteDirectory\":\"\",\"DeleteAfterUpload\":false}"
}
```

```json
{
    "module":"AutoRecord",
    "version":"1",
    "id":"[hashcode]",
    "env":[
        {
           "name":"PACK_PID",
           "type":"string"
        },
        {
           "name":"PACK_NAME",
           "type":"string"
        }
      ],
    "data":"{\"TargetKey\":\"\",\"RemoteDirectory\":\"\",\"DeleteAfterUpload\":false}"
}
```

# Transfer file construction

```json
{
    "profileid":"BC2B7511-7DEE-45D8-BD30-C11D1720B1DF",
    "data":"{\"TargetKey\":\"\",\"RemoteDirectory\":\"\",\"DeleteAfterUpload\":false}"
}
```

