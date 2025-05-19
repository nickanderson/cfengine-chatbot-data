`/tmp/test.json`{.verbatim}:

``` {.json tangle="/tmp/test.json"}
{
    "test": "this is just a test"
}
```

`/tmp/test.notjson`{.verbatim}:

``` {.json tangle="/tmp/test.notjson"}
I've got a lovely bunch of coco-nuts, there they are standing in a row!
```

`/tmp/test.notjsonbutvalid`{.verbatim}:

``` {.json tangle="/tmp/test.notjsonbutvalid"}
"test": "this is not valid json, but CFEngine seems to tolerate it. BUG!"
```

And some policy to exercise it:

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" extra-opts="--show-evaluated-vars=default:main\\\\." tangle="checking_to_see_if_json_files_are_valid.cf"}
bundle agent __main__
{

  vars:
      "files" slist => { "/tmp/test.json", "/tmp/test.notjson", "/tmp/test.notjsonbutvalid" };

      # "d1" data => readjson( "/tmp/test.notjsonbutvalid" ); # Prodcues an error: readjson: non-container from parsing JSON file '/tmp/test.notjsonbutvalid'

  reports:

      "validjson() thinks that the content in $(files) is $(with) JSON string!"
        if => fileexists( "$(files)" ),
        with => ifelse( validjson( readfile( "$(files)" ) ), "valid", "not valid"),
        printfile => cat( $(files) );

}
```

``` example
R: validjson() thinks that the content in /tmp/test.json is valid JSON string!
R: {
R:     "test": "this is just a test"
R: }
R: validjson() thinks that the content in /tmp/test.notjson is not valid JSON string!
R: I've got a lovely bunch of coco-nuts, there they are standing in a row!
R: validjson() thinks that the content in /tmp/test.notjsonbutvalid is valid JSON string!
R: "test": "this is not valid json, but CFEngine seems to tolerate it. BUG, IMHO!"
Variable name                            Variable value                                               Meta tags                                Comment
default:main.files                        {"/tmp/test.json","/tmp/test.notjson","/tmp/test.notjsonbutvalid"} source=promise
```
