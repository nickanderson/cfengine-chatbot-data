``` {.cfengine3 tangle="output_data_from_yaml_in_specific_format.cf"}
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

  reports:

      "CFEngine '$(sys.cf_version)'";
      "$(with)::isbn: '$(yaml[books][$(schemas)][isbn])' with book_name: '$(yaml[books][$(schemas)][book_name])'"
        with => regex_replace( "$(schemas)", "_schema", "", "g" );
}
```
