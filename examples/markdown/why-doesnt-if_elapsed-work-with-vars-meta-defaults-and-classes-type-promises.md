CLOSED: \[2020-01-21 Tue 08:40\]

> Why doesn\'t `body action if_elapsed`{.verbatim} work with
> `vars`{.verbatim} type promises?

Let\'s start with the details of `body action if_elapsed`{.verbatim}.

``` cfengine3
body action if_elapsed(x)
# @brief Evaluate the promise every `x` minutes
# @param x The time in minutes between promise evaluations
{
      ifelapsed => "$(x)";
      expireafter => "$(x)";
}
```

The [`ifelapsed`{.verbatim} action body
attribute](https://docs.cfengine.com/docs/master/reference-promise-types.html#ifelapsed)
is the number of minutes before next allowed assessment of a promise. It
overrides
[`body agent control ifelapsed`{.verbatim}](https://docs.cfengine.com/docs/master/reference-components-cf-agent.html#ifelapsed)
which defaults to 1 minute. It\'s intended to prevent overload due to
unintentional resource consumption, but it can be used to control
frequency. The [`expireafter`{.verbatim} action body
attribute](https://docs.cfengine.com/docs/master/reference-promise-types.html#expireafter)
is the number of minutes a promise is allowed to run before the agent is
subject to termination.

So the `if_elapsed`{.verbatim} action body is saying that once the
promise has been verified as having an outcome of kept or repaired it
should remain locked for x minutes. The agent should not try to
re-verify it until that lock is no longer valid. Additionally if another
agent sees that an agent is still executing the promise after x minutes
then the apparently stuck agent should be terminated.

In cfengine a promise is locked once it is seen to be kept or repaired.
By default promises are locked for 1 minute. Until a lock expires the
promise will be skipped. In this example, there are two promises. One
promise to report the date, and one promise to report
`Hello`{.verbatim}. The date will be different each time the agent is
run so the promise lock will not be the same between two agent runs and
thus it will not be affected by promise locking.

:::: captioned-content
::: caption
Example illustrating promise locking
:::

``` {#Example illustrating promise locking .cfengine3 use-locks="yes"}
bundle agent main
{
   reports:
     "$(sys.date)";
     "Hello";
     "Infrequent hello"
      action => if_elapsed( "5" );
}
```
::::

Here are the results of several executions of
`cf-agent -f /tmp/example.cf`.

```{=org}
#+Caption: Example illustrating promise locking first run
```
```{=org}
#+RESULTS: Example illustrating promise locking first run
```
``` example
R: Sat Dec 23 09:53:57 2017
R: Hello
R: Infrequent hello
```

```{=org}
#+Caption: Example illustrating promise locking second run
```
```{=org}
#+RESULTS: Example illustrating promise locking second run
```
``` example
R: Sat Dec 23 09:54:19 2017
```

```{=org}
#+Caption: Example illustrating promise locking third run
```
```{=org}
#+RESULTS: Example illustrating promise locking third run
```
``` example
R: Sat Dec 23 09:54:59 2017
R: Hello
```

```{=org}
#+Caption: Example illustrating promise locking fourth run
```
```{=org}
#+RESULTS: Example illustrating promise locking fourth run
```
``` example
R: Sat Dec 23 09:55:14 2017
```

```{=org}
#+Caption: Example illustrating promise locking fifth run
```
```{=org}
#+RESULTS: Example illustrating promise locking fifth run
```
``` example
R: Sat Dec 23 09:55:59 2017
R: Hello
```

```{=org}
#+Caption: Example illustrating promise locking sixth run
```
```{=org}
#+RESULTS: Example illustrating promise locking sixth run
```
``` example
R: Sat Dec 23 10:38:03 2017
R: Hello
R: Infrequent hello
```

`vars`{.verbatim} type promises do not participate in promise locking
and as such can not be affected by the `ifelapsed`{.verbatim} action
body attribute or the `ifelapsed`{.verbatim} agent control attribute.

:::: captioned-content
::: caption
Example illustrating vars don\'t participate in locking
:::

``` {#Example illustrating vars don't participate in locking .cfengine3 use-locks="yes"}
bundle agent main
{
  vars:
      "msg" string => "Hello World";

  reports:

      "$(sys.date)";

      "$(msg)"
        action => immediate;
}
```
::::

Here are the results of a couple runs (
`cf-agent -f /tmp/vars-are-not-locked.cf` ).

```{=org}
#+RESULTS: Example illustrating vars don't participate in locking
```
``` example
R: Tue Dec 26 13:03:15 2017
R: Hello World
```

Another run a few seconds later:

```{=org}
#+RESULTS: Example illustrating vars don't participate in locking
```
``` example
R: Tue Dec 26 13:03:28 2017
R: Hello World
```

We can see that `Hello World`{.verbatim} was reported in both runs. It
was only reported because we attached `body action immediate`{.verbatim}
to the reports promise. Note the `vars`{.verbatim} promise did not
require `body action immediate`{.verbatim} in order for the variable
`msg`{.verbatim} to hold the value `Hello World`{.verbatim}.

<https://docs.cfengine.com/docs/3.10/reference-promise-types.html#expireafter>

Promises which take a long time to verify should usually be protected
with a long value for this parameter.

<https://docs.cfengine.com/docs/3.10/reference-components-cf-agent.html#ifelapsed>

`body action if_elapsed`{.verbatim} works on promise locks.

<https://docs.cfengine.com/docs/3.10/reference-promise-types.html#ifelapsed>

- Does not mention that it uses promise locks
- Does not mention what factors are considered by promise locks

<https://docs.cfengine.com/docs/3.10/reference-promise-types.html#ifelapsed>

- Does not mention that it uses promise locks
- Does not mention what factors are considered by promise locks

<https://docs.cfengine.com/docs/master/reference-components-cf-agent.html#expireafter>

- Mentions it represents the locking time after wich cfengine will
  attemtpt to kill and restart its attempt to keep a promise
- 

<https://docs.cfengine.com/docs/3.10/guide-faq-what-is-promise-locking.html>

- TODO note in faq entry that vars promises are not subject to promise
  locking.

- Promise locking is based on promise outcomes, variables and classes
  unlike other promise types do not have outcomes.

- Vars aren\'t subject to promise locking because vars can and are often
  expected to contain dynamic content. Note that variable values are
  refreshed on each pass of convergence. Variables that you do not want
  to be refreshed can be guarded with
  `if => not( isvariable( $(this.promiser) ) )`{.verbatim} for example:

  ``` cfengine3
  bundle agent main
  {
    vars:
        "foo" string => "cat";
        "foo" string => "dog";
        "foo" string => "bar", if => not( isvariable( $(this.promiser) ));

    reports:
        "foo == $(foo)";

  }
  ```

  ```{=org}
  #+RESULTS:
  ```
  ``` example
  R: foo == dog
  ```

  See also cached functions, and

In cfengine a promise is locked once it is seen to be kept or repaired.
By default promises are locked for 1 minute. Until a lock expires the
promise will be skipped. In this example, there are two promises. One
promise to report the date, and one promise to report
`Hello`{.verbatim}. The date will be different each time the agent is
run so it will not be affected by promise locking.

:::: captioned-content
::: caption
Example illustrating promise locking
:::

``` {#Example illustrating promise locking .cfengine3 use-locks="yes"}
bundle agent main
{
   reports:
     "$(sys.date)";
     "Hello";
     "Infrequent hello"
      action => if_elapsed( "5" );
}
```
::::

Here are the results of several executions of
`cf-agent -f /tmp/example.cf`.

```{=org}
#+Caption: Example illustrating promise locking first run
```
```{=org}
#+RESULTS: Example illustrating promise locking first run
```
``` example
R: Sat Dec 23 09:53:57 2017
R: Hello
R: Infrequent hello
```

```{=org}
#+Caption: Example illustrating promise locking second run
```
```{=org}
#+RESULTS: Example illustrating promise locking second run
```
``` example
R: Sat Dec 23 09:54:19 2017
```

```{=org}
#+Caption: Example illustrating promise locking third run
```
```{=org}
#+RESULTS: Example illustrating promise locking third run
```
``` example
R: Sat Dec 23 09:54:59 2017
R: Hello
```

```{=org}
#+Caption: Example illustrating promise locking fourth run
```
```{=org}
#+RESULTS: Example illustrating promise locking fourth run
```
``` example
R: Sat Dec 23 09:55:14 2017
```

```{=org}
#+Caption: Example illustrating promise locking fifth run
```
```{=org}
#+RESULTS: Example illustrating promise locking fifth run
```
``` example
R: Sat Dec 23 09:55:59 2017
R: Hello
```

```{=org}
#+Caption: Example illustrating promise locking sixth run
```
```{=org}
#+RESULTS: Example illustrating promise locking sixth run
```
``` example
R: Sat Dec 23 10:38:03 2017
R: Hello
R: Infrequent hello
```

We can run the agent without locks (`-K`{.verbatim}, or
`--no-lock`{.verbatim}) and we can adjust locking for individual
promises using an `action`{.verbatim} body. Here we execute the agent
with out any additional options but attach an action body
(`immediate`{.verbatim}) which has `ifelapsed`{.verbatim} set to 0 to
disregard any locks.

``` {.cfengine3 use-locks="yes"}
bundle agent main
{
  reports:

    "CFEngine $(sys.cf_version)"
      action => immediate;

      "It's $(with)"
        with => "$(sys.date)";
}
```

```{=org}
#+RESULTS: Second run
```
``` example
R: CFEngine 3.11.0
R: It's Sat Dec 23 08:49:49 2017
```

```{=org}
#+RESULTS: First run
```
``` example
R: CFEngine 3.11.0
R: It's Sat Dec 23 08:49:43 2017
```

Ok, now that you understand promise locking.

What about vars type promises?

> So how would I report only once every x period when the report
> contains a unique string?

The canonical way to make decisions in CFEngine is using classes. So it
should come as no surprise that you can define classes that persist for
a period of time and use the presence or absence of those classes to
create contexts that are active for a period of time. Persistent classes
are different from promise locks. They will still apply even when the
agent is run without locks.

In this example we extend `body classes results`{.verbatim} to define
persistent classes that are used to guard the report of the date. We
define the persistent classes for 5 minutes.

``` {.cfengine3 use-locks="yes" tangle="/tmp/test.cf"}
bundle agent main
{
  reports:

    "CFEngine $(sys.cf_version)"
      action => immediate;

    !date_report_reached::
      "It's $(sys.date)"
        classes => my_persist_results( "date_report", "5" );

  vars:

    "c" slist => classesmatching( "date_report.*" );

  reports:

    "Found class '$(c)'"
      action => immediate;
}
body classes my_persist_results(class_prefix, minutes)
# @brief Extends `body classes results` and makes classes persistent.
#
# @ **Note:** This body defines `namespace` scoped classes
{
        inherit_from => results( "namespace", $(class_prefix) );
        persist_time => "$(minutes)";
}
```

Here is the policy in action:

``` example
/tmp λ cf-agent -Kf./test.cf 
R: CFEngine 3.11.0
R: It's Sat Dec 23 09:22:08 2017
R: Found class 'date_report_reached'
R: Found class 'date_report_kept'
/tmp λ cf-agent -Kf./test.cf 
R: CFEngine 3.11.0
R: Found class 'date_report_reached'
R: Found class 'date_report_kept'
/tmp λ cf-agent -Kf./test.cf 
R: CFEngine 3.11.0
R: It's Sat Dec 23 09:29:44 2017
R: Found class 'date_report_reached'
R: Found class 'date_report_kept'
cf-agent -Kf./test.cf 
R: CFEngine 3.11.0
R: It's Sat Dec 23 09:22:08 2017
R: Found class 'date_report_reached'
R: Found class 'date_report_kept'
```

Promise locking and classes type promises

``` {.cfengine3 use-locks="yes"}
bundle agent main
{
  classes:

    "myclass" expression => "any";

  reports:

    myclass::
      "$(sys.date) The class is defined";
}
```

We can see that the report is emitted during each run, thus
`myclass`{.verbatim} is defined regardless of any locks.

```{=org}
#+RESULTS: one
```
``` example
R: Tue Dec 26 15:19:22 2017 The class is defined
```

```{=org}
#+RESULTS:
```
``` example
R: Tue Dec 26 15:19:29 2017 The class is defined
```
