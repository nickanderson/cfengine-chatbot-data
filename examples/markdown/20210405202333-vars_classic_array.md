```{=org}
#+roam_tags: CFEngine
```
```{=org}
#+filetags: CFEngine
```
``` {.cfengine3 tangle="vars_classic_array.cf"}
bundle agent __main__
{
  vars:
    "array[key0]" string => "value0";
    "array[key1]" string => "value1";
    "array[key2]" string => "value2";

  reports:
    "$(with)" with => join(", ", getindices( "array" ));
    "$(with)" with => join(",", getvalues( "array" ));
}
```

**References:**

- [join()](id:b91239e5-37fb-4d53-8335-9a38a16800ca)
- [getindices()](id:cb299172-277a-42de-a1c9-c82e54379e4e)
- [getvalues()](id:d202c34d-21c3-46e9-a668-79fbdb61b9e7)
- [with =\>](id:5f995365-46bb-4701-80d8-12c163fc6ca8)
