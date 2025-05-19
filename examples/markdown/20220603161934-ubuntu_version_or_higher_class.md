For ubuntu 20 or higher ....

:::: captioned-content
::: caption
Example Policy
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" tangle="ubuntu_version_or_higher_class.cf"}
bundle agent __main__
{
  reports:
      "I am running $(sys.os_release[NAME]) $(sys.os_version_major)";


  classes:
      "ubuntu_20_or_higher"
        scope => "namespace",
        expression => and(
                           strcmp("$(sys.os_release[NAME])", "Ubuntu"),
                           isgreaterthan( $(sys.os_version_major), "19" ));

  reports:
      ubuntu_20_or_higher::
        "It's newer than Ubuntu 19, so I have the class ubuntu_20_or_higher defined.";

      !ubuntu_20_or_higher::
        "It's older than Ubuntu 20";
}
```
::::

``` example
R: I am running Ubuntu 22
R: It's newer than Ubuntu 19, so I have the class ubuntu_20_or_higher defined.
```

# References

- [CFEngine Examples](id:38277465-771a-4db4-983a-8dfd434b1aff)
