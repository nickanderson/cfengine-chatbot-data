``` {.cfengine3 tangle="read_module_protocol.cf"}
bundle agent __main__
{
  classes:
      "module_data_read"
        expression => read_module_protocol( "/tmp/module-protocol-output.dat" ),
        if => isplain(  "/tmp/module-protocol-output.dat" );

  reports:
    module_data_read::
      "Owner: $(fabios_inventory.owner)";
      "fabios_inventory.owner Tags: $(with)"
        with => join( ", ", getvariablemetatags( "fabios_inventory.owner" ));

}
bundle agent init
{
  files:
      "/tmp/module-protocol-output.dat"
        content => `^context=fabios_inventory
^meta=inventory,attribute_name=Owner
=owner=Fabio
^meta=inventory,attribute_name=Administrative Contacts
_contact= { 'Fabio', 'Nick', '911' }
^meta=inventory,attribute_name=USB Devices
=myarray[key]= array key val
=usb_device[Bus 001 Device 002: ID 8087:8001]=Intel Corp.
=usb_device[Bus 001 Device 001: ID 1d6b:0002]=Linux Foundation 2.0 root hub
=usb_device[Bus 003 Device 003: ID 17ef:1010]=Lenovo Lenovo ThinkPad Dock
=usb_device[Bus 003 Device 001: ID 1d6b:0003]=Linux Foundation 3.0 root hub
=usb_device[Bus 002 Device 009: ID 04ca:703c]=Lite-On Technology Corp. Integrated Camera
=usb_device[Bus 002 Device 007: ID 8087:0a2a]=Intel Corp.
=usb_device[Bus 002 Device 005: ID 04f3:036d]=Elan Microelectronics Corp. Touchscreen
=usb_device[Bus 002 Device 054: ID 046d:081a]=Logitech, Inc. Webcam C260
=usb_device[Bus 002 Device 053: ID 17ef:100f]=Lenovo
=usb_device[Bus 002 Device 055: ID 074d:0005]=Micronas GmbH
=usb_device[Bus 002 Device 052: ID 17ef:1010]=Lenovo Lenovo ThinkPad Dock
=usb_device[Bus 002 Device 004: ID 413c:2006]=Dell Computer Corp.
=usb_device[Bus 002 Device 002: ID 413c:1004]=Dell Computer Corp. Dell USB Keyboard Hub
=usb_device[Bus 002 Device 001: ID 1d6b:0002]=Linux Foundation 2.0 root hub
^meta=noreport
%mydata={ "key": "value", "array": [1,2,3] }
^meta=inventory,attribute_name=Roles,role=server
+module_role_server
^meta=noreport
^persistence=10
+module_persistent_class_defined_for_10_minutes
`;

}
```
