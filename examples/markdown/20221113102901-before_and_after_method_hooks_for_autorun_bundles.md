# Bundles tagged with hook~methods(before\|after)~=namespace:bundle {#99c28b20-8d26-4498-a01d-7e7e861ffcb4}

``` {.cfengine3 tangle="before_and_after_method_hooks_for_autorun_bundles.cf"}
bundle agent BeforeMyAutorunBundle()
{
  meta:
      "tags" slist => { "hook_methods_before=default:MyAutorunBundle" };

  reports:
      "Hello from $(this.namespace):$(this.bundle)";

}
bundle agent AfterMyAutorunBundle()
{
  meta:
      "tags" slist => { "hook_methods_after=default:MyAutorunBundle" };

  reports:
      "Hello from $(this.namespace):$(this.bundle)";
}
bundle agent SomeBundleAfterMyAutorunBundle()
{
  meta:
      "tags" slist => { "hook_methods_after=default:MyAutorunBundle" };

  reports:
      "Hello from $(this.namespace):$(this.bundle)";
}
bundle agent MyAutorunBundle()
# @brief MyAutorunBundle ...
{
  meta:
      "tags"
        slist => { "autorun" };
  reports:
      "Hello from $(this.namespace):$(this.bundle)";
}
bundle agent AnotherAutorunBundle()
# @brief MyAutorunBundle ...
{
  meta:
      "tags"
        slist => { "autorun" };
  reports:
      "Hello from $(this.namespace):$(this.bundle)";
}

bundle agent autorun_discovery
{
  vars:

      "autorun" slist => bundlesmatching( ".*", "autorun" );
      "hook_before[$(autorun)]" slist => bundlesmatching( ".*", "hook_methods_before=$(autorun)" );
      "hook_after[$(autorun)]" slist => bundlesmatching( ".*", "hook_methods_after=$(autorun)" );
}
bundle agent autorun(autorun_bundle)
{
  vars:
  methods:

      # Some bug affects associative arrays which necessitates running a bundle to
      # run the before hook bundles instead of running them directly.

      "Before hook: $(autorun_bundle)" usebundle => autorun_hook_before( $(autorun_bundle) );

      "$(autorun_budnle)" usebundle => "$(autorun_bundle)";

      "Before hook: $(autorun_bundle)" usebundle => autorun_hook_after( $(autorun_bundle) );

}
bundle agent autorun_hook_before(autorun_bundle)
{
  methods:
      "Before hook $(autorun_bundle)"
        usebundle => "$(autorun_discovery.hook_before[$(autorun_bundle)])";
}
bundle agent autorun_hook_after(autorun_bundle)
{
  methods:
      "After hook $(autorun_bundle)"
        usebundle => "$(autorun_discovery.hook_after[$(autorun_bundle)])";

}
bundle agent __main__
# @brief Drive default bundlesequence if called as policy
{
  methods:
      "Autorun Discovery..."
        usebundle => autorun_discovery;

      "Autorun Discovery..."
        usebundle => autorun("$(autorun_discovery.autorun)");

   reports:
      "CFEngine $(sys.cf_version)";
}
```

# Bundles tagged with autorun AND hook~methods(before\|after)~=namespace:bundle

This however, suffers from running hooked bundles both before/after but
also because they are tagged autorun themselves

Need to develop this further so that bundles tagged with autorun AND
hook~methods(before\|after)~=.\* are not run with just the autorun
bundles.

``` {.cfengine3 tangle="before_and_after_method_hooks_for_autorun_bundles.cf"}
bundle agent BeforeMyAutorunBundle()
{
  meta:
      "tags" slist => { "hook_methods_before=default:MyAutorunBundle" };

  reports:
      "Hello from $(this.namespace):$(this.bundle)";

}
bundle agent AfterMyAutorunBundle()
{
  meta:
      "tags" slist => { "hook_methods_after=default:MyAutorunBundle" };

  reports:
      "Hello from $(this.namespace):$(this.bundle)";
}
bundle agent SomeBundleAfterMyAutorunBundle()
{
  meta:
      "tags" slist => { "autorun", "hook_methods_after=default:MyAutorunBundle" };

  reports:
      "Hello from $(this.namespace):$(this.bundle)";
}
bundle agent MyAutorunBundle()
# @brief MyAutorunBundle ...
{
  meta:
      "tags"
        slist => { "autorun" };
  reports:
      "Hello from $(this.namespace):$(this.bundle)";
}
bundle agent AnotherAutorunBundle()
# @brief MyAutorunBundle ...
{
  meta:
      "tags"
        slist => { "autorun" };
  reports:
      "Hello from $(this.namespace):$(this.bundle)";
}

bundle agent autorun_discovery
{
  vars:

      "autorun" slist => bundlesmatching( ".*", "autorun" );
      "hook_before[$(autorun)]" slist => intersection( bundlesmatching( ".*", "hook_methods_before=$(autorun)" ),
                                                       @(autorun) );
      "hook_after[$(autorun)]" slist => intersection( bundlesmatching( ".*", "hook_methods_after=$(autorun)" ),
                                                      @(autorun) );
}
bundle agent autorun(autorun_bundle)
{
  vars:
  methods:

      # Some bug affects associative arrays which necessitates running a bundle to
      # run the before hook bundles instead of running them directly.

      "Before hook: $(autorun_bundle)" usebundle => autorun_hook_before( $(autorun_bundle) );

      "$(autorun_budnle)" usebundle => "$(autorun_bundle)";

      "Before hook: $(autorun_bundle)" usebundle => autorun_hook_after( $(autorun_bundle) );

}
bundle agent autorun_hook_before(autorun_bundle)
{
  methods:
      "Before hook $(autorun_bundle)"
        usebundle => "$(autorun_discovery.hook_before[$(autorun_bundle)])";
}
bundle agent autorun_hook_after(autorun_bundle)
{
  methods:
      "After hook $(autorun_bundle)"
        usebundle => "$(autorun_discovery.hook_after[$(autorun_bundle)])";

}
bundle agent __main__
# @brief Drive default bundlesequence if called as policy
{
  methods:
      "Autorun Discovery..."
        usebundle => autorun_discovery;

      "Autorun Discovery..."
        usebundle => autorun("$(autorun_discovery.autorun)");

   reports:
      "CFEngine $(sys.cf_version)";
}
```
