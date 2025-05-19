``` {.cfengine3 tangle="canonifyuniquely.cf"}
bundle agent __main__
{
  vars:
      "s" string => "hello-world";
      "a" string => canonifyuniquely($(s));
      "b" string => concat(canonify($(s)), "_", hash($(s),"sha1"));

  reports:
      "a = $(a)";
      "b = $(b)";
      "a and b are identical"
        if => strcmp( $(a), $(b) );

}
```

**References:**

- [CFEngine Examples](id:38277465-771a-4db4-983a-8dfd434b1aff)
- [canonifyuniquely()](id:e0afaf78-745e-41bd-9ca6-2a02ff322108)
- [concat()](id:b9a498ff-0f13-4195-9850-9d1b4ec7a403)
- [canonify()](id:3575108f-cb76-4107-82eb-3db20b302e13)
- [hash()](id:798b10f2-4f1c-4cab-a5ff-7613d713af04)
- [strcmp()](id:a136eeee-4d49-4d15-afb7-0e8c1104d488)
