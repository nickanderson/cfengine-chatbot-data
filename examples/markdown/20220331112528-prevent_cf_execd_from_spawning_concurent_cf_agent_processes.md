If want to prevent `cf-execd` from spawning concurrent `cf-agent`
processes you can adjust `exec_command`{.verbatim} in
`body executor control`{.verbatim} not execute `update.cf`{.verbatim}
and `promises.cf`{.verbatim} policy entries directly.

# Using `flock` on Linux

On Linux you can use `flock` (or some similar strategy) to prevent
`cf-execd` from spawning `cf-agent` when there is already another
`cf-agent` spawned by `cf-execd` that is still running.

To do so, adjust `exec_command`{.verbatim} in
`body executor control`{.verbatim}
([controls/cf~execd~.cf](https://github.com/cfengine/masterfiles/blob/c2954fd7bcb4e238d7368d69f869eec0fbbe35f6/controls/cf_execd.cf#L50-L52)).

``` cfengine3
linux::
  exec_command => '/bin/flock --nonblock --exclusive /var/cfengine/state/cf-execd.exec_command --command "$(sys.cf_agent) -Dfrom_cfexecd,cf_execd_initiated -f \"$(sys.update_policy_path)\" ; $(sys.cf_agent) -Dfrom_cfexecd,cf_execd_initiated"';
```

This will result in Mission Portal showing a Health condition for
\"`Last agent run unsuccessful`{.verbatim}\" when there is a lock held.
If you do not want to treat skipping an agent run because an existing
lock is held as a promise not kept resulting in Mission Portal showing
\"`Last agent run unsuccessful`{.verbatim}\" Health alert add
`--conflict-exit-code 0`{.verbatim} to the `flock` command. In this
case, skipped runs might be surfaced under the Health condition for
\"`Agent not recently run`{.verbatim}\".

``` cfengine3
linux::
  exec_command => '/bin/flock --conflict-exit-code 0 --nonblock --exclusive /var/cfengine/state/cf-execd.exec_command --command "$(sys.cf_agent) -Dfrom_cfexecd,cf_execd_initiated -f \"$(sys.update_policy_path)\" ; $(sys.cf_agent) -Dfrom_cfexecd,cf_execd_initiated"';
```

If you want to *wait* indefinitely for the lock to be freed instead of
exiting, remove the `--nonblock`{.verbatim} option from the flock
command:

``` cfengine3
linux::
  exec_command => '/bin/flock --exclusive /var/cfengine/state/cf-execd.exec_command --command "$(sys.cf_agent) -Dfrom_cfexecd,cf_execd_initiated -f \"$(sys.update_policy_path)\" ; $(sys.cf_agent) -Dfrom_cfexecd,cf_execd_initiated"';
```

If you want to *wait* for a limited period of time before timing out use
the `--timeout`{.verbatim} for `flock`:

``` cfengine3
linux::
  exec_command => '/bin/flock --timeout 30 --exclusive /var/cfengine/state/cf-execd.exec_command --command "$(sys.cf_agent) -Dfrom_cfexecd,cf_execd_initiated -f \"$(sys.update_policy_path)\" ; $(sys.cf_agent) -Dfrom_cfexecd,cf_execd_initiated"';
```

**Note:**

- **Be sure that `/bin/flock` is present on the hosts being targeted or
  no policy will be executed and the host will have cut itself off from
  getting further policy updates.**
- Using this sort of global lock will prevent `cf-agent` from culling
  long running agents that have breached `expireafter`{.verbatim}. You
  may want to be sure that some external watchdog is watching out for
  excessively long held locks or excessively long running `cf-agent`
  processes.
- These examples only target preventing `cf-execd` from spawning
  `cf-agent` when there is an existing agent spawned by `cf-execd`
  already running. It will not affect the ability to launch an agent via
  `cf-runagent` nor will it skip if there is an existing process that
  was spawned by `cf-runagent` or an existing `cf-agent` spawned in some
  other way. Similar treatment could be done for
  `cfruncommand`{.verbatim} in `body server control`{.verbatim}
  (`controls/cf-serverd.cf`{.verbatim}) to make the same lock apply to
  `cf-runagent` spawned executions of `cf-agent`.
