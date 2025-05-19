``` {.cfengine3 tangle="splitting_the_numeric_portion_from_network_interfaces.cf"}
bundle agent main
{
  vars:
     "my_list"    slist => { "xvnc10", "xvnc30", "xvnc40" };

     # When you split a string, you will have at least 2 parts from splitting
     # it. Since you are splitting off the front of the string you end up with
     # an empty string (the first half of your split string) and then the digits
     # in the second half. Since you are only interested in the second half of
     # that split, you can pick it out using nth(). You might be able to do
     # similar using data_regextract() or similar.

     "my_split[$(my_list)]"  string => nth( string_split( $(my_list), "^xvnc", 2), 1);

  reports:
    "CFEngine: $(sys.cf_version)";
    "my_list   $(my_list)";
    "my_split $(my_split[$(my_list)])";
}
```
