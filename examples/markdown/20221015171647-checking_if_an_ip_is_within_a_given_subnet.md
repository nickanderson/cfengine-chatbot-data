``` {.cfengine3 tangle="checking_if_host_has_an_ip_is_within_a_given_subnet.cf"}
bundle agent __main__
{
  vars:
      "subnet" string => "192.168.42.0/24";

  reports:
      "One of my ip addresses is not the subnet '$(subnet)'"
        unless => isipinsubnet( $(subnet) );

      "One of my ip addresses is inside the subnet '$(subnet)'"
        if => isipinsubnet( $(subnet) );

      "IPs: $(with)" with => join( ", ", @(sys.ip_addresses) );
}
```
