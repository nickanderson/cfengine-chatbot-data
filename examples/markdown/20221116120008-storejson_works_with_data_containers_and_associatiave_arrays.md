``` {.cfengine3 tangle="storejson_works_with_data_containers_and_associatiave_arrays.cf"}
bundle agent __main__
{
  vars:
      "arr[one]" int => "1";
      "arr[2]" string => "2";

  reports:
      "Directly: $(with)"
        with => storejson( arr );

      "With proper type: $(with)"
        with => storejson( mergedata( arr ) );
}
```
