``` {.cfengine3 tangle="munging_yaml_data_structure_into_a_list_of_strings.cf"}
bundle agent main
{
  vars:
      "yaml" data => parseyaml ('
  books:
    scifi_schema:
      isbn: "isbn_scifi"
      book_name: "book_name_scifi"
    travel_schema:
      isbn: "isbn_travel"
      book_name: "book_name_travel"
');

      "schemas" slist => getindices( "@(yaml[books])" );

      # Munge the data structure into a unified list of strings
      "arr_munged[$(schemas)]"
        string => "$(with)::isbn: '$(yaml[books][$(schemas)][isbn])' with book_name: '$(yaml[books][$(schemas)][book_name])'",
        with => regex_replace( "$(schemas)", "_schema", "", "g" );
      "l_munged" slist => getvalues( arr_munged );

  reports:

      "CFEngine '$(sys.cf_version)'";
      "$(l_munged)";
}
```
