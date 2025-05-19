:::: captioned-content
::: caption
Example Policy
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both"}
bundle agent main
{
  methods:
    "example_1" usebundle => text_between(
      "example_1",
      "Hello Rosetta Code world",
      "Hello ",
      " world",
      "Rosetta Code"
    );
    "example_2" usebundle => text_between(
      "example_2",
      "Hello Rosetta Code world",
      "start",
      " world",
      "Hello Rosetta Code"
    );
    "example_3" usebundle => text_between(
      "example_3",
      "Hello Rosetta Code world",
      "Hello ",
      "end",
      "Rosetta Code world"
    );
  reports:
    "CFEngine $(sys.cf_version)"; 
}

bundle agent text_between(example_name, text, start_delim, end_delim, expected)
{
  vars:

      "$(example_name)_tmp" string => ifelse( strcmp("start", "$(start_delim)" ), "$(text)",
                                               nth(string_split("$(text)", $(start_delim), 2), 1));

      "$(example_name)_actual" string => ifelse( strcmp("end", "$(end_delim)"), "$($(example_name)_tmp)",
                                                 nth(string_split("$($(example_name)_tmp))", $(end_delim), 2), 0));

  classes:
    "$(example_name)_ok" expression => strcmp("$(expected)", "$($(example_name)_actual)");

  reports:
    "$(example_name).text:        '$(text)'";
    "$(example_name).start_delim: '$(start_delim)'";
    "$(example_name).end_delim:   '$(end_delim)'";
    "$(example_name).expected:    '$(expected)'";
    "$(example_name)_tmp:         '$($(example_name)_tmp)'";
    "$(example_name)_actual:      '$($(example_name)_actual)'";
#    "strcmp(expected,actual):     '$(with)'" with => strcmp("$(expected)", "$($(example_name)_actual)");
    "$(example_name) PASS" if => "$(example_name)_ok";
    "$(example_name) FAIL" if => "$(example_name)_NOT_ok";
}
```
::::

``` example
R: example_1.text:        'Hello Rosetta Code world'
R: example_1.start_delim: 'Hello '
R: example_1.end_delim:   ' world'
R: example_1.expected:    'Rosetta Code'
R: example_1_tmp:         'Rosetta Code world'
R: example_1_actual:      'Rosetta Code'
R: example_1 PASS
R: example_2.text:        'Hello Rosetta Code world'
R: example_2.start_delim: 'start'
R: example_2.end_delim:   ' world'
R: example_2.expected:    'Hello Rosetta Code'
R: example_2_tmp:         'Hello Rosetta Code world'
R: example_2_actual:      'Hello Rosetta Code'
R: example_2 PASS
R: example_3.text:        'Hello Rosetta Code world'
R: example_3.start_delim: 'Hello '
R: example_3.end_delim:   'end'
R: example_3.expected:    'Rosetta Code world'
R: example_3_tmp:         'Rosetta Code world'
R: example_3_actual:      'Rosetta Code world'
R: example_3 PASS
R: CFEngine 3.19.0a.da01eaa81
```

# References

- [CFEngine Examples](id:38277465-771a-4db4-983a-8dfd434b1aff)
- [ifelse()](id:3f8422a1-c478-4512-a166-e3dfb91784ec)
- [strcmp()](id:a136eeee-4d49-4d15-afb7-0e8c1104d488)
- [nth()](id:841e294b-9bbc-4145-a1b0-4fab0802edbe)
- [string~split~()](id:61ecf84b-2333-4b5b-87dc-385f16bffd4b)
