[Genders](https://github.com/chaos/genders/tree/master) is a static
cluster configuration database used for cluster configuration
management. It is used by a variety of tools and scripts for management
of large clusters.

A genders file looks like this:

:::: captioned-content
::: caption
/etc/genders
:::

``` conf
# nodename attr[=value],attr[=value]
nodename linux,oel7,prod,enc=10-01-r3-e1
precision-5570 linux,ubntu24,prod,enc=10-01-r3-e1
```
::::

# Policy

This policy defines `g_`{.verbatim} prefixed `namespace`{.verbatim}
scoped classes for the attributes without values and defines variables
for attributes with values both tagged for reporting to an Enterprise
Hub but only the variables tagged for inventory.

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" extra-args="--show-evaluated-vars=genders" tangle="/tmp/genders.cf"}
bundle agent __main__
{
  methods:
      "init";
      "genders";
}
bundle agent init
{
  files:
      "$(genders.filename)"
        content => "# nodename attr[=value],attr[=value]
# Some attributes are exposed to CFEngine as classes
# E.g. g_prod (genders production)
nodename linux,remote,oel7,prod,dfas, rad,oci,enc=g10-01-r3-e1
$(sys.uqhost) linux,remote,ubntu24,prod,dfas,rad,oci,enc=g10-01-r3-e1
";

}

bundle agent genders
{
  vars:
      "filename" string => "/tmp/genders";

      # I started trying to parse it in one go, but the first , separated attribte was getting combind with the nodename so abandoned
      # "node_attrs"
      #   if => fileexists( "$(genders.filename)" ),
      #   data => data_readstringarray(
      #                                 "$(genders.filename)", # filename,
      #                                 "#[^\n]*",      # comment,
      #                                 " ",      # split,
      #                                 "inf",          # maxentries
      #                                 "inf");         # maxbytes


      # I can't seem to get a negative lookahead to work.. exclude all lines that aren't my gender line, so i read the whole file in and then filter out lines not matching the executing node
      "my_entry"
        slist => filter( "^$(sys.uqhost)\s+.*",
                         readstringlist( "$($(this.bundle).filename)",
                                         "#[^\n]*",
                                         "\n", inf, inf ),
                         true, false, inf );

      # Then we get the list of attribute[=values]
      "my_attrs_vals"
        slist => string_split( nth( string_split( nth( my_entry, 0 ),
                                                  "\s+", inf ), 1), ",", inf );

      # Then we get the list of attributes that don't have  value
      "my_attrs_noval" slist => filter( ".*=.*", my_attrs_vals, "true", "true", inf );

      # Then we get the list of attributes that do have  value
      "my_attrs_wval" slist => filter( ".*=.*", my_attrs_vals, "true", "false", inf );

      # Now I make a map of attributes that have values to their values and I tag for reporting and inventory

      "map_attrs_w_val[$(with)]"
        string => nth( string_split( "$(my_attrs_wval)", "=", 2 ), 1 ),
        meta => { "derived-from=/etc/genders",
                  "report",
                  "inventory",
                  "attribute_name=/etc/genders[$(with)]" },
        with => nth( string_split( "$(my_attrs_wval)", "=", 2 ), 0 );

      "_map_attrs_w_val_list" slist => getindices( "map_attrs_w_val" );

  classes:
      "g_$(my_attrs_noval)"
        meta => { "derived-from=/etc/genders", "report" },
        scope => "namespace";

  reports:
      "Classes defined derived from /etc/genders: $(with)"
        with => join( ", ", classesmatching( ".*", "derived-from=/etc/genders" ));

      "$(_map_attrs_w_val_list) = $(map_attrs_w_val[$(_map_attrs_w_val_list)])";
}
```

``` example
R: Classes defined derived from /etc/genders: g_prod, g_dfas, g_linux, g_rad, g_ubntu24, g_remote, g_oci
R: enc = g10-01-r3-e1
```

# [DONE]{.done .DONE} File feature issue for function to parse /etc/genders into a data structure {#file-feature-issue-for-function-to-parse-etcgenders-into-a-data-structure}

<https://northerntech.atlassian.net/browse/CFE-4500>
