``` {.cfengine3 tangle="measure_value_with_pipe_stream_type.cf"}
bundle agent example_measurement_probe
# Note: This is an /agent/ bundle, it needs to be run somehow, included in
# inputs, tagged for autorun, called by a methods promise in another policy or
# added to the bundlesequence.
{
  meta:
    linux::
      "tags" slist => { "autorun" };

  files:
      "/tmp/my-probe"
        content => "#!/bin/bash
echo $RANDOM",
        perms => m( "700" ),
        comment => "measurements promises having stream_type => 'pipe' can not
                    take parameters, so we render a script to output the data we
                    want to sample";

}

bundle monitor thing
# Note: measurement bundles should /not/ be added to the bundlesequence
# cf-monitord /must/ be restarted in order to pick up the new promise
{
  measurements:

      "/tmp/my-probe"
        handle => "my_probe",
        stream_type => "pipe",
        data_type => "int",
        history_type => "weekly",
        units => "whole numbers",
        match_value => single_value( "\d+" );

}

bundle agent __main__
{
  methods:
      "example_measurement_probe";
}
```

# References

- [CFEngine Examples](id:38277465-771a-4db4-983a-8dfd434b1aff)
- [cf-monitord](id:99de5de8-26a7-4778-9251-05151523a5f7)
