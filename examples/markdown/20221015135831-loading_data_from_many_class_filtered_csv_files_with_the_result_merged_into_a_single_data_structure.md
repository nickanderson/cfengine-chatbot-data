``` {.cfengine3 tangle="loading_data_from_many_class_filtered_csv_files_with_the_result_merged_into_a_single_data_structure.cf"}
bundle agent __main__
{
  %?
}
```

:::: captioned-content
::: caption
csv/filemode.csv
:::

``` {.csv tangle="~/vagrant/CFEngine/3.12.1/CFE-2768/csv/filemode.csv"}
classexpression,file,permission
any,/tmp/file1,600
```
::::

:::: captioned-content
::: caption
csv/filemode.d/000.csv
:::

``` {.csv tangle="~/vagrant/CFEngine/3.12.1/CFE-2768/csv/filemode.d/000.csv"}
classexpression,file,permission
any,/tmp/file2,700
```
::::

:::: captioned-content
::: caption
csv/filemode.d/001.csv
:::

``` {.csv tangle="~/vagrant/CFEngine/3.12.1/CFE-2768/csv/filemode.d/001.csv"}
classexpression,file,permission
any,$(main.MYFILE),777
```
::::

The CSV files **need** to have CRLF line endings.

``` {.bash org-language="sh" dir="~/vagrant/CFEngine/3.12.1/CFE-2768"}
find . -name "*.csv" | xargs -n 1 unix2dos
```

:::: captioned-content
::: caption
test.cf
:::

``` {.cfengine3 tangle="~/vagrant/CFEngine/3.12.1/CFE-2768/test.cf" include-stdlib="f" verbose-mode="nil" inform-mode="nil"}
bundle agent main
#@brief Example loading data from many class filtered csv files merged into a single data structure
{
  vars:
      "MYFILE" string => "/tmp/NicksFile";

      "item"          string => "filemode";
      "csvdir"        string => "$(this.promise_dirname)/csv";

      "csvfiles"
        slist  => { "$(csvdir)/$(item).csv",
                    lsdir("$(csvdir)/$(item).d", ".*.csv", "true") };

      "csvdata" data   => '[]';
      "csvdata"
        data => mergedata("csvdata",
                          classfiltercsv( $(csvfiles),
                                          "true", # Data file contains column headings
                                          0));    # Column containing class expression to filter with

      "i" slist => getindices(csvdata);

      # Enforce that each file in the merged, filtered data set has permissions enforced
      # files:
      #   "$(csvdata[$(i)][file])"
      #     perms => m( "$(csvdata[$(i)][permission])" );

  reports:
      "$(csvfiles)" printfile => cat( $(csvfiles) );

      "Expanded$(const.n)$(with)"
        with => string_mustache( "{{%-top-}}", data_expand(csvdata));
}

body perms m(mode)
# @brief Set the file mode
# @param mode The new mode
{
        mode   => "$(mode)";
}
body printfile cat(file)
# @brief Report the contents of a file
# @param file The full path of the file to report
{
        file_to_print => "$(file)";
        number_of_lines => "inf";
}
```
::::

``` {.bash org-language="sh" dir="~/vagrant/CFEngine/3.12.1/CFE-2768"}
chmod 600 test.cf
cf-agent -KIf ./test.cf --show-evaluated-vars=default:main.csvdata
```

``` example
[root@host001 CFE-2768]#   cf-agent -KIf ./test.cf --show-evaluated-vars=default:main.csvdata
R: /vagrant/CFE-2768/./csv/filemode.csv
R: classexpression,file,permission
R: any,/tmp/file1,600
R: /vagrant/CFE-2768/./csv/filemode.d/000.csv
R: classexpression,file,permission
R: any,/tmp/file2,700
R: /vagrant/CFE-2768/./csv/filemode.d/001.csv
R: classexpression,file,permission
R: any,$(main.MYFILE),777
R: Expanded
[
  {
    "file": "/tmp/file1",
    "permission": "600"
  },
  {
    "file": "/tmp/file2",
    "permission": "700"
  },
  {
    "file": "/tmp/NicksFile",
    "permission": "777"
  }
]
Variable name                            Variable value                                               Meta tags
default:main.csvdata                     [{"file":"/tmp/file1","permission":"600"},{"file":"/tmp/file2","permission":"700"},{"file":"$(main.MYFILE)","permission":"777"}] source=promise
```
