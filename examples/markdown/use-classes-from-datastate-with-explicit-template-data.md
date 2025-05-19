``` {.cfengine3 exports="both"}
bundle agent main
{
   vars:

     "m" data => '{ "MyData": "explicit" }';

     # Get current datastate, pick classes, reparent classes, merge explicit
     # data on to classes

     "s" data => datastate(), if => not( isvariable ( s ) );
      # Break datastate out into all classes and variables separately, TODO
      # datastate("vars"), datastate("classes"), datastate("all")
     "c" data => mergedata( '{ }', "s[classes]" );
     "c1" data => mergedata( '{"classes": c}' );
     "r" data => mergedata( c1, m );

   reports:
     "$(with)"
       with => string_mustache("My data is {{{MyData}}} and I am running CFEngine {{#classes.enterprise_edition}}Enterprise{{/classes.enterprise_edition}}{{^classes.enterprise_edition}}Community{{/classes.enterprise_edition}}", r );

      "The full data container:$(const.n)$(with)"
       with => string_mustache("{{%-top-}}", r );
}
```

``` example
R: My data is explicit and I am running CFEngine Enterprise
R: The full data container:
{
  "MyData": "explicit",
  "classes": {
    "127_0_0_1": true,
    "172_17_0_1": true,
    "192_168_0_1": true,
    "192_168_100_1": true,
    "192_168_10_1": true,
    "192_168_121_1": true,
    "192_168_122_1": true,
    "192_168_42_189": true,
    "4_cpus": true,
    "64_bit": true,
    "Day8": true,
    "GMT_Afternoon": true,
    "GMT_Day8": true,
    "GMT_Hr13": true,
    "GMT_Hr13_Q4": true,
    "GMT_Lcycle_2": true,
    "GMT_Min45_50": true,
    "GMT_Min49": true,
    "GMT_Monday": true,
    "GMT_October": true,
    "GMT_Q4": true,
    "GMT_Yr2018": true,
    "Hr08": true,
    "Hr08_Q4": true,
    "Hr8": true,
    "Lcycle_2": true,
    "Min45_50": true,
    "Min49": true,
    "Monday": true,
    "Morning": true,
    "October": true,
    "PK_SHA_43c979e264924d0b4a2d3b568d71ab8c768ef63487670f2c51cd85e8cec63834": true,
    "Q4": true,
    "Yr2018": true,
    "agent": true,
    "any": true,
    "cfengine": true,
    "cfengine_3": true,
    "cfengine_3_12": true,
    "cfengine_3_12_0": true,
    "compiled_on_linux_gnu": true,
    "debian": true,
    "debian_buster": true,
    "enterprise": true,
    "enterprise_3": true,
    "enterprise_3_12": true,
    "enterprise_3_12_0": true,
    "enterprise_edition": true,
    "fe80__5ee0_c5ff_fe9f_f38f": true,
    "fe80__fc54_ff_fe61_1523": true,
    "fe80__fc54_ff_fe7a_cef0": true,
    "fe80__fc54_ff_fef0_9262": true,
    "fe80__fc54_ff_fef8_3bda": true,
    "feature": true,
    "feature_curl": true,
    "feature_def": true,
    "feature_def_json": true,
    "feature_def_json_preparse": true,
    "feature_xml": true,
    "feature_yaml": true,
    "ipv4_127": true,
    "ipv4_127_0": true,
    "ipv4_127_0_0": true,
    "ipv4_127_0_0_1": true,
    "ipv4_172": true,
    "ipv4_172_17": true,
    "ipv4_172_17_0": true,
    "ipv4_172_17_0_1": true,
    "ipv4_192": true,
    "ipv4_192_168": true,
    "ipv4_192_168_0": true,
    "ipv4_192_168_0_1": true,
    "ipv4_192_168_10": true,
    "ipv4_192_168_100": true,
    "ipv4_192_168_100_1": true,
    "ipv4_192_168_10_1": true,
    "ipv4_192_168_121": true,
    "ipv4_192_168_121_1": true,
    "ipv4_192_168_122": true,
    "ipv4_192_168_122_1": true,
    "ipv4_192_168_42": true,
    "ipv4_192_168_42_189": true,
    "ipv4_gw_192_168_42_1": true,
    "ipv6_fe80__5ee0_c5ff_fe9f_f38f": true,
    "ipv6_fe80__fc54_ff_fe61_1523": true,
    "ipv6_fe80__fc54_ff_fe7a_cef0": true,
    "ipv6_fe80__fc54_ff_fef0_9262": true,
    "ipv6_fe80__fc54_ff_fef8_3bda": true,
    "linux": true,
    "linux_4_15_0_33_lowlatency": true,
    "linux_x86_64": true,
    "linux_x86_64_4_15_0_33_lowlatency": true,
    "linux_x86_64_4_15_0_33_lowlatency__36_Ubuntu_SMP_PREEMPT_Wed_Aug_15_17_20_28_UTC_2018": true,
    "mac_02_42_d2_da_e3_92": true,
    "mac_52_54_00_3c_1e_b7": true,
    "mac_52_54_00_6b_62_06": true,
    "mac_52_54_00_7d_55_73": true,
    "mac_52_54_00_9b_e8_bd": true,
    "mac_52_54_00_ff_1d_f1": true,
    "mac_5c_e0_c5_9f_f3_8f": true,
    "net_iface_docker0": true,
    "net_iface_lo": true,
    "net_iface_virbr0": true,
    "net_iface_virbr1": true,
    "net_iface_virbr3": true,
    "net_iface_virbr4": true,
    "net_iface_virbr5": true,
    "net_iface_wlan0": true,
    "nickanderson_thinkpad_w550s": true,
    "nova": true,
    "nova_3": true,
    "nova_3_12": true,
    "nova_3_12_0": true,
    "nova_edition": true,
    "systemd": true,
    "ubuntu": true,
    "ubuntu_18": true,
    "ubuntu_18_04": true,
    "ubuntu_18_4": true,
    "x86_64": true
  }
}
```

:::: captioned-content
::: caption
Example Policy
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both"}
bundle agent __main__
{
  vars:
    "l" slist => classesmatching( ".*" );
    "m" string => storejson( mapdata( "none", '{ "$(this.v)": true }', l ) );
    # Remove the array characters []
    "n" string => regex_replace( "$(m)", "(\[)(.*)(\])", "{\2}", "gms" );
    "o" string => string_replace( $(n), '"{ \\', '' );
    "p" string => string_replace( $(o), '}"', '' );
    "q" string => string_replace( $(p), '\\', '' );
    "z" data => parsejson( '{ "classes": $(q) }' );
  reports:
    "Hello $(with)"
      with => string_mustache( "{{#classes.feature_yaml}}YAML!{{/classes.feature_yaml}}",
                               z );
}
```
::::

``` example
R: Hello YAML!
```
