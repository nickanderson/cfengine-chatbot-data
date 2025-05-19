``` {.cfengine3 tangle="parsing_suggestions_from_lynis_report_dat.cf" extra-opts="--show-evaluated-vars=lynis_suggestions\\\\."}
bundle agent __main__
{
vars:
      "lynis_report_dat" string => "/tmp/lynis-report.dat";

methods:
      "Mock linus report.dat"
        usebundle => mock_lynis_report_dat( $(lynis_report_dat) );

      "Parse suggestions from lynis report"
        usebundle => lynis_suggestions( $(lynis_report_dat) );

}
bundle agent mock_lynis_report_dat(file)
{
      # The lynis-report.dat file format is line based. Getting it into
      # digest able parts requires several promises.

      # Example lynis-report.dat snippet
      #  # Lynis Report
      #  lynis_version=2.7.4
      #  suggestion[]=BOOT-5122|Set a password on GRUB bootloader to prevent altering boot configuration (e.g. boot in single user mode without password)|-|-|
      #  suggestion[]=FILE-6354|Check 5 files in /tmp which are older than 90 days|-|-|

      # = Is the Primary key, value separator, followed by |. Since there can be
      # multiple lines with the same key, we need to index based on the line
      # number, which is why we are using data_readstringarrayidx().

  files:
      "$(file)"
        content => "# Lynis Report
lynis_version=2.7.4
suggestion[]=BOOT-5122|Set a password on GRUB bootloader to prevent altering b
suggestion[]=FILE-6354|Check 5 files in /tmp which are older than 90 days|-|-|";
}
bundle agent lynis_suggestions(file)
{
  vars:

      "d"
        if => fileexists( $(file) ),
        data => data_readstringarrayidx( $(file),
                                         "",
                                         "=",
                                         inf,
                                         inf);

      "i" slist => getindices( d );

      # Generate a datastructure to contain suggestions. We extract the
      # Suggestion ID to use as the key, and extract the description to use as
      # the value. In order to not lose information when there are multiple
      # suggestions with the same ID we use the index position from the original
      # structure to make unqiue entries for each comment

      "suggestion[$(with)]"
        string => nth( string_split( "$(d[$(i)][1])", "\|", 3 ), 1 ),
        if => strcmp( "suggestion[]", "$(d[$(i)][0])"),
        meta => { "inventory",
                  "attribute_name=CISOfy Lynis Suggestions",
                  "lynis-control-id=$(with)"
        },
        with => nth( string_split( "$(d[$(i)][1])", "\|", 3 ), 0 );

      "si" slist => getindices( suggestion );
}
```
