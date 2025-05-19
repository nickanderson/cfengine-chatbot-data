``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" tangle="set_a_class_if_some_elements_of_a_list_match_a_regular_expression.cf"}
bundle agent __main__
{

  vars:
      "users" slist => { "" };
  classes:

      "_have_control_executor_runagent_socket_allow_users"
        expression => some( ".+.*", "users" );

  reports:
      _have_control_executor_runagent_socket_allow_users::
        "I have users specified";
      !_have_control_executor_runagent_socket_allow_users::
        "I don't have users specified";


}
```

``` example
R: I don't have users specified
```
