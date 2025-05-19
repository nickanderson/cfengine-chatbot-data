``` json
[
  {
    "name": "/tmp/create-multiple-files-with.content-1.txt",
    "content": "Hello CFEngine!"
  },
  {
    "name": "/tmp/create-multiple-files-with.content-2.txt",
    "content": "Bye CFEngine!"
  }
]
```

# Lars first solution {#lars-first-solution cfengine_functions="[[id:02720c30-efe9-4bb8-b360-fbf79886a13d][Function: isvariable()]] [[id:5da3dcf8-5ea7-41e9-92a2-23e3755ad6fd][Function: format()]] [[id:b9a498ff-0f13-4195-9850-9d1b4ec7a403][Function: concat()]] [[id:24647e3a-2af2-4460-897d-5b539bff2171][Function: eval()]] [[id:22b0b944-3335-40c8-957c-0e6a474d1c85][Function: length()]] [[id:0dfced06-32c5-4cbd-80d6-9e0b99f9b953][Function: expandrange()]]" cfengine_promisetypes="[[id:afe3a708-4fc0-482a-bddf-5c97a0a557e4][Promise type: meta]] [[id:431e6692-7600-4467-a0c0-609ea7c09a17][Promise type: classes]] [[id:b31e06a4-d3b1-44f2-9292-cd20ca17cb66][Promise type: vars]] [[id:23504787-b597-41ff-819d-b9625f773210][Promise type: files]]"}

``` {.cfengine3 tangle="managing_files_described_by_a_data_container-lars-v0.cf"}
bundle agent create_multiple_files_with_content
# @brief Create single files with content.
{
  meta:
      "tags"
        slist => { "autorun" };

  classes:
      "input_present"
        expression => isvariable("my_namespace:my_bundle.files");

  vars:
    input_present::
      "num_files"
        string => format("%d", concat(eval("$(with)-1", math, infix))),
        with => concat(length("my_namespace:my_bundle.files"));
      "range"
        slist => { expandrange("[0-$(num_files)]", 1) };

  files:
    input_present::
      "$(my_namespace:my_bundle.files[$(range)][name])"
        content => "$(my_namespace:my_bundle.files[$(range)][content])";
}
```

# Nicks first solution {#nicks-first-solution cfengine_functions="[[id:cb299172-277a-42de-a1c9-c82e54379e4e][Function: getindices()]]" cfengine_promisetypes="[[id:b31e06a4-d3b1-44f2-9292-cd20ca17cb66][Promise type: vars]] [[id:23504787-b597-41ff-819d-b9625f773210][Promise type: files]] [[id:c458bf16-1ba9-499f-a801-e94e0f80a5c9][Promise type: reports]]"}

``` {.cfengine3 tangle="managing_files_described_by_a_data_container-nick-v0.cf"}
bundle agent __main__
{
  vars:
      "d" data => '[
  {
    "name": "/tmp/create-multiple-files-with.content-1.txt",
    "content": "Hello CFEngine!"
  },
  {
    "name": "/tmp/create-multiple-files-with.content-2.txt",
    "content": "Bye CFEngine!"
  }
]';

      "i" slist => getindices( d );

  files:
    write_file::
      "$(d[$(i)][name])"
        content => "$(d[$(i)][content])";

  reports:
      "$(d[$(i)][name]) <- $(d[$(i)][content])";

}

```
