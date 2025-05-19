Note, the user with which you authenticate this API call must have the
host delete capability, e.f. the `admin`{.verbatim} user has the
`admin`{.verbatim} role, which by default has all Host API capabilities.

``` {.bash org-language="sh" results="output" exports="both" dir="/ssh:hub|sudo:hub:"}
curl --insecure --user admin:admin https://hub.example.com/api/host/SHA=9b6e2813b00426761c72f462db78613710a9e23decc93ba4997e12576a802f2c
```

``` example
{
  "data": [
    {
      "firstseen": "1667419961",
      "hostname": "desktop-aj2m57e",
      "id": "SHA=9b6e2813b00426761c72f462db78613710a9e23decc93ba4997e12576a802f2c",
      "ip": "192.168.56.1",
      "lastreport": "1667835406"
    }
  ],
  "meta": {
    "count": 1,
    "page": 1,
    "timestamp": 1667835462,
    "total": 1
  }
}
```
