``` {.cfengine3 tangle="validdata_with_json.cf"}
bundle agent __main__
{
  vars:
      "json_string_0" string => '{ "valid": "json" }';
      "json_string_1" string => '{ invalid json }';

  reports:
      "json_string_0 is valid"
        if => validdata( "$(json_string_0)", JSON);

      "json_string_1 is invalid"
        unless => validdata( "$(json_string_1)", JSON);
}
```
