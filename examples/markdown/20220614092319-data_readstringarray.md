Prototype
:   `data_readstringarray(filename, comment, split, maxentries, maxbytes)`{.verbatim}

Parse line based delimited file into a `data`{.verbatim} variable.

``` {.cfengine3 tangle="data_readstringarray.cf"}
bundle agent __main__
{
  methods:
      "init";
      "demo";
}
bundle agent demo
{
  vars:
      "pkgs_desired_data"
        if => fileexists( "/tmp/line-based-delimited.dat" ),
        data => data_readstringarray( "/tmp/line-based-delimited.dat",
                                      "\s+#[^\n]*",
                                      "\s+|\s+",
                                      inf,
                                      inf);
      "i" slist => getindices( pkgs_desired_data );

  reports:
      "$(with)"
        with => storejson( pkgs_desired_data );
      "'$(i)' should be '$(pkgs_desired_data[$(i)][1])'";



}
bundle agent init
{
  files:
      "/tmp/line-based-delimited.dat"
        content => "  # This is a line I would like to ignore
# Package Name | Desired Package State (present|absent)
vim | present
emacs | present
nano | absent
";

}
```
