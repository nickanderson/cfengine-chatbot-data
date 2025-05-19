[CFEngine Examples](id:38277465-771a-4db4-983a-8dfd434b1aff)

It\'s the bundle that will get used if no bundlsequence is provided AND
the file it\'s contained within is the policy *entry*.

:::: captioned-content
::: caption
one.cf
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both"}
bundle agent main
{
  methods:
      "example1";
}

bundle agent example1
{
  reports: "$(this.bundle) in $(this.promise_filename)";
}
```
::::

:::: captioned-content
::: caption
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both"}
bundle agent main
{
  methods:
      "example2";
}

bundle agent example2
{
  reports: "$(this.bundle) in $(this.promise_filename)";
}
```
::::

:::: captioned-content
::: caption
zero.cf
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both"}
body file control
{
      inputs => { "/tmp/one.cf", "/tmp/two.cf" };
}
bundle agent main
{
  methods:
      "example1";
      "example2";
}
```
::::

:::: captioned-content
::: caption
Shell command with stderr
:::

``` {.bash org-language="sh" results="output" exports="both"}
exec 2>&1
chmod 600 /tmp/*.cf
cf-agent -Kf /tmp/one.cf
cf-agent -Kf /tmp/two.cf
cf-agent -Kf /tmp/zero.cf
:
```
::::

``` example
R: example1 in /tmp/one.cf
R: example2 in /tmp/two.cf
/tmp/zero.cf:6:0: error: Duplicate definition of bundle main with type agent
/tmp/zero.cf:6:0: error: Duplicate definition of bundle main with type agent
/tmp/one.cf:7:0: error: Duplicate definition of bundle main with type agent
/tmp/one.cf:7:0: error: Duplicate definition of bundle main with type agent
/tmp/two.cf:7:0: error: Duplicate definition of bundle main with type agent
/tmp/two.cf:7:0: error: Duplicate definition of bundle main with type agent
   error: Policy failed validation with command '"/home/nickanderson/.cfagent/bin/cf-promises" -c "/tmp/zero.cf"'
   error: CFEngine was not able to get confirmation of promises from cf-promises, so going to failsafe
   error: CFEngine failsafe.cf: /home/nickanderson/.cfagent/inputs /home/nickanderson/.cfagent/inputs/failsafe.cf
   error: TRUST FAILED, server presented untrusted key: SHA=0e4765d53cb0f9808c1de230f7c6b966c5ec3465c2d7a4b2446ab1fc4b684c24
   error: No suitable server found for '/home/nickanderson/.cfagent/inputs'
   error: Promise belongs to bundle 'failsafe_cfe_internal_update' in file '/home/nickanderson/.cfagent/inputs/failsafe.cf' near line 121
   error: Errors encountered when actuating files promise '/home/nickanderson/.cfagent/inputs'
   error: TRUST FAILED, server presented untrusted key: SHA=0e4765d53cb0f9808c1de230f7c6b966c5ec3465c2d7a4b2446ab1fc4b684c24
   error: No suitable server found for '/home/nickanderson/.cfagent/modules'
   error: Promise belongs to bundle 'failsafe_cfe_internal_update' in file '/home/nickanderson/.cfagent/inputs/failsafe.cf' near line 130
   error: Errors encountered when actuating files promise '/home/nickanderson/.cfagent/modules'
   error: TRUST FAILED, server presented untrusted key: SHA=0e4765d53cb0f9808c1de230f7c6b966c5ec3465c2d7a4b2446ab1fc4b684c24
   error: No suitable server found for '/home/nickanderson/.cfagent/inputs'
   error: Promise belongs to bundle 'failsafe_cfe_internal_update' in file '/home/nickanderson/.cfagent/inputs/failsafe.cf' near line 144
   error: Comment is 'If we failed to fetch policy we try again using
                    the legacy default in case we are fetching policy
                    from a hub that is not serving mastefiles via a
                    shortcut.'
   error: Errors encountered when actuating files promise '/home/nickanderson/.cfagent/inputs'
   error: Method 'failsafe_cfe_internal_update' failed in some repairs
R: Built-in failsafe policy triggered
```

------------------------------------------------------------------------

:::: captioned-content
::: caption
one.cf
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" tangle="/tmp/one.cf"}
bundle agent __main__
{
  methods:
      "example1";
}

bundle agent example1
{
  reports: "$(this.bundle) in $(this.promise_filename)";
}
```
::::

:::: captioned-content
::: caption
two.cf
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" tangle="/tmp/two.cf"}
bundle agent __main__
{
  methods:
      "example2";
}

bundle agent example2
{
  reports: "$(this.bundle) in $(this.promise_filename)";
}
```
::::

:::: captioned-content
::: caption
zero.cf
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both"}
body file control
{
      inputs => { "/tmp/one.cf", "/tmp/two.cf" };
}
bundle agent __main__
bundle agent main
{
  methods:
      "example1";
      "example2";
}
```
::::

:::: captioned-content
::: caption
Shell command with stderr
:::

``` {.bash org-language="sh" results="output" exports="both"}
exec 2>&1
chmod 600 /tmp/*.cf
cf-agent -Kf /tmp/one.cf
cf-agent -Kf /tmp/two.cf
cf-agent -Kf /tmp/zero.cf
:
```
::::

``` example
R: example1 in /tmp/one.cf
R: example2 in /tmp/two.cf
R: example1 in /tmp/one.cf
R: example2 in /tmp/two.cf
```

- *bundle agent [[main]{.underline}]{.underline}* is only parsed if the
  policy file is the entry. the entry is the first policy file loaded
  (typically, `promises.cf`{.verbatim}, `update.cf`{.verbatim})
- You can have issues if you have both
  `bundle agent __main__`{.verbatim} and `bundle agent main`{.verbatim}
  in the same set of inputs.
  - So, you should never include a file with
    `bundle agent main`{.verbatim} from a file that has
    `bundle agent __main__`{.verbatim}
