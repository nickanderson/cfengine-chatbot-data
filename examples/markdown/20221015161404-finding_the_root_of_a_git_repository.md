``` {.cfengine3 tangle="finding_the_root_of_a_git_repository.cf"}
bundle agent __main__
{
  reports:
      "$(with)" with => storejson( findfiles_up( "/home/nickanderson/src/yasnippet-snippets/snippets/c-lang-common/", ".git", inf  ));

      "findfiles_up also has an alias, search_up():$(with)"
        with => storejson( search_up( "/home/nickanderson/src/yasnippet-snippets/snippets/c-lang-common/", ".git", inf  ));
}
```
