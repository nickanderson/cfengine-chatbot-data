Created From: [Third response](~/org/refile.org::*Third%20response)

As of 3.15.0 the better way to accomplish this is via `basename()`.

``` {.cfengine3 include-stdlib="no"}
bundle agent main
{
  vars:
      "files" slist => { "file1.cf", "file2.cf", "file3.cf" };

  methods:

      "Strip suffix using nth() and string_split()"
        usebundle => strip_suffix_nth_string_split( $(files) );

     "Strip suffix by nesting string_split() in nth()"
       usebundle => strip_suffix_nth_nested_string_split( $(files) );

      "Strip suffix using regex_replace()"
        usebundle => strip_suffix_regex_replace( $(files) );

      "Strip suffix using maplist() and regex_replace()"
        usebundle => strip_suffix_maplist( @(files) );

  reports:
      "CFEngine $(sys.cf_version)";
}

bundle agent strip_suffix_nth_string_split(name)
{
  vars:
    # This is compatible with 3.7.x
      "_n" slist => string_split( $(name), "\.cf", '2');
      "n" string => nth( _n, 0);

  reports:
      "$(this.bundle) passed $(name) -> $(n)";
}

bundle agent strip_suffix_nth_nested_string_split(name)
{
  vars:
    # This is compatible with 3.10.x
      "n" string => nth( string_split( $(name), "\.cf", '2'), 0);

  reports:
      "$(this.bundle) passed $(name) -> $(n)";
}

bundle agent strip_suffix_regex_replace(name)
# @depends: 3.10
{
  vars:
    # This is compatible with 3.10.x
      "n"  string => regex_replace( $(name), "\.cf$", "", "g" );

  reports:
      "$(this.bundle) passed $(name) -> $(n)";
}

bundle agent strip_suffix_maplist(names)
# @breif NOTE this want's to operate on the entire listof names at once
{
  vars:
    # This is compatible with 3.10.x
      "n" slist => maplist( regex_replace( $(this), "\.cf$", "", "g" ), @(names) );

  reports:
      "$(this.bundle) $(n)";
}

bundle agent strip_suffix_regex_replace_with(name)
# @depends 3.11 (with attibute)
{
  reports:
    # This is compatible with 3.11.x
      "$(this.bundle) passed $(name) -> $(with)"
        with => regex_replace( $(name), "\.cf$", "", "g" );
}


```
