```{=org}
#+filetags: CFEngine_Example
```
``` {.cfengine3 tangle="function_eval_pi_and_ceil.cf"}
bundle agent __main__
{
  vars:
    "pi" string => eval( "pi", "math", "infix");
    "pi_rounded_up" string => eval( "ceil(pi)", "math", "infix" ); 
  reports:
    "pi == '$(pi)'";
    "pi_rounded_up == '$(pi_rounded_up)'";
}
```
