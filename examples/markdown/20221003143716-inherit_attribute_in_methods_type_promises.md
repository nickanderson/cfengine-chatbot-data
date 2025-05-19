``` {#example .cfengine3 include-stdlib="t" log-level="info" exports="both" command-in-result="t" tangle="inherit_attribute_in_methods_type_promises.cf"}
bundle agent main
{
    methods: "guac";
}
bundle agent guac
{
    classes:
      "tomcat" expression => "guacamole";

    methods:
      "Taco"
        usebundle => taco,
        inherit => "true";
}
bundle agent taco
{
    reports:
      tomcat::
        "With tomcat on the side please";
      !tomcat::
        "No tomcat, thanks";
}
```

```{=org}
#+call: example()
```
```{=org}
#+RESULTS:
```
``` example
# cf-agent --no-lock --log-level info --file /home/nickanderson/org/roam/daily/work/cfengine3-tHYz8J
R: No tomcat, thanks
```

```{=org}
#+call: example() :define guacamole
```
```{=org}
#+RESULTS:
```
``` example
# cf-agent --define guacamole --no-lock --log-level info --file /home/nickanderson/org/roam/daily/work/cfengine3-is6sCY
R: With tomcat on the side please
```
