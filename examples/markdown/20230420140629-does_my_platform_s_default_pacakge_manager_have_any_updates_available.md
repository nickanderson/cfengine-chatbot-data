Note: This is expected to be integrated into the MPF. It will not work
as a standalone example (3.21.0):

``` {.cfengine3 tangle="does_my_platform_s_default_pacakge_manager_have_any_updates_available.cf"}
bundle agent __main__
{
  vars:

      # What updates does the platform default package manager have ...
      "package_updates"
        data => packageupdatesmatching(".*", # package_regex
                                       ".*", # version_regex
                                       ".*", # arch_regex
                                       # method_regex : If package_inventory in body common control is not set,
                                       "$(default:package_module_knowledge.platform_default)");


  classes:
      "up_to_date"
        expression => strcmp( 0, length( package_updates ) );

  reports:
      "We are $(with)up to date" with => ifelse( "up_to_date", "", "NOT " );

}
```
