[CFEngine Examples](id:38277465-771a-4db4-983a-8dfd434b1aff)

:::: captioned-content
::: caption
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" extra-opts="--show-evaluated-vars=default:main\\\\."}
bundle agent __main__
{
    vars:
      "ß" string => "Value";
      "ś" string => "value2";
      "š" string => "value3";

    reports:
      "CFEngine $(sys.cf_version)";
}
```
::::

``` example
R: CFEngine 3.18.0a.b248be4d0
Variable name                            Variable value                                               Meta tags                               
default:main.ß                          Value                                                        source=promise                          
default:main.ś                          value2                                                       source=promise                          
default:main.š                          value3                                                       source=promise                          
```
