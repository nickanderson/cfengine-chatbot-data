`defaults`{.verbatim} are evaluated after `vars`{.verbatim} and before
`classes`{.verbatim}. They promises provide a way to set a default a
variable value and the possibility to override variable values that
match a regular expression. `defaults`{.verbatim} are not evaluated
during pre-evaluation.

In CFEngine undefined variables do not map to the empty string, they
remain as variables for possible future expansion. Some variables might
be defined but still contain unresolved variables. To handle this you
will need to match the `$(abc)`{.verbatim} form of the variables.

Negative look-ahead ( `(?!expression)`{.verbatim} ) regular expressions
are very useful for `if_match_regex`{.verbatim} to check against known
valid patterns.

:::: captioned-content
::: caption
Example overriding variables that match a regular expression
:::

``` cfengine3
bundle agent main
{
  defaults:

      "number"
        string => "0",
        comment => "If this holds anything but digits we override it to a
                    valid default";
        if_match_regex =>

          # Just to demonstrate how you can format regular expressions for readability wih inline documentation.
          # Note that because (?msx) ignores whitespace you'll have to use `\s` and
          # perhaps modifiers to express any whitespace, including new lines.
          #  

         "(?xms)
            # Digits only
            (?!
               [[:digit:]] # Digits
               +           # One or more times
            ).*";          # Anyting


      "PermitRootLogin"
        string => "no",
        if_match_regex => "(prohibit-password|yes)",
        comment => "We explicitly disallow root logins, but allow any other
                    value (even invalid config options)";

  vars:

      "numbers"
        comment => "We expect numbers",
        string => "We use defaults to validate this"; 

      "PermitRootLogin"
        string => "forced-commands-only";

  reports:
      "number holds '$(number)'";
      "PermitRootLogin holds '$(PermitRootLogin)'";
}
```
::::

``` cfengine3
bundle agent main
{
  classes:
      "classes_are_first" expression => not( isvariable( "main.classes_are_first" ) );
      "vars_are_first" expression => isvariable( "main.vars_are_first" );

  vars:
      "classes_are_first" string => "Classes are first";
      "vars_are_first" string => "Vars are first", if => not( "vars_are_first" );

  reports:

    "$(classes_are_first)"
      if => "classes_are_first";

    "$(vars_are_first)"
      if => "vars_are_first";
}

bundle agent terrible
{


  defaults:

      # We can have default values even if variables are not defined at all.
      # This is equivalent to a variable definition, so not particularly useful.

      "X" string => "I am a default value";
      "Y" slist => { "I am a default list item 1", "I am a default list item 2" };
      "a" string => "AAAAAAAAA",   if_match_regex => "";

  methods:

      # More useful, defaults if parameters are passed to a param bundle

      "example" usebundle => mymethod("","bbb");

  reports:

      "The default value of X is $(X)";
      "The default value of Y is $(Y)";
}

###########################################################

bundle agent mymethod(a,b)

{
  vars:

      "no_return" string => "ok"; # readfile("/dont/exist","123");

  defaults:

      "a" string => "AAAAAAAAA",   if_match_regex => "";

      "b" string => "BBBBBBBBB",   if_match_regex => "";

      "no_return" string => "no such file";

  reports:

      "The value of a is $(a)";
      "The value of b is $(b)";

      "The value of no_return is $(no_return)";

}
```

\> How do I know if vars or classes come first during cfengine
evaluation?

1.  Read the documentation on Normal Order.

2.  Run the agent with --verbose or -v and grep for `pass 1`{.verbatim}.

    Write a test policy that has both vars and classes promises.

    :::: captioned-content
    ::: caption
    `/tmp/test.cf`{.verbatim}
    :::

    ``` cfengine3
    bundle agent main
    {
     vars:
        "question" string => "What comes first, vars or classes?";

     classes:
        "grep_verbose_output_for_pass_1" expression => "any";
    }
    ```
    ::::

    Grep the verbose output.

    :::: captioned-content
    ::: caption
    Grepping for promises that run on the frist pass
    :::

    ``` {.shell exports="both" results="output" wrap="EXAMPLE"}
    chmod 600 /tmp/test.cf 
    cf-agent -Kvf /tmp/test.cf | grep "pass 1"
    ```
    ::::

    ``` example
    verbose: V: BEGIN variables (pass 1)
    verbose: C: BEGIN classes / conditions (pass 1)
    ```

3.  Write policy that tells you.

``` {.cfengine3 verbose="t"}
bundle agent main
#@ brief Demonstrate if vars or classes come first
{

  classes:
      "classes_are_first" expression => not( isvariable( "main.classes_are_first" ) );
      "vars_are_first" expression => isvariable( "main.vars_are_first" );

  vars:
      "classes_are_first" string => "Classes come first";
      "vars_are_first" string => "Vars come first", if => not( "vars_are_first" );
      "answer" string => ifelse( classes_are_first, "ifelse: Classes come first",
                                 vars_are_first, "ifelse: Vars come first",
                                 "Logic Error" );

  reports:

    "$(classes_are_first)"
      if => "classes_are_first";

    "$(vars_are_first)"
      if => "vars_are_first";

    "$(answer)";
}
```
