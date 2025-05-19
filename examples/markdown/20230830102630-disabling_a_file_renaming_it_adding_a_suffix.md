``` {.cfengine3 tangle="disabling_a_file_renaming_it_adding_a_suffix.cf"}
bundle agent __main__
{
  files:
    "/tmp/example.txt"
      content => "Disable me!";

    "/tmp/example.txt"
      rename => default:disable;
}
# For reference:
# body rename disable
# # @brief Disable the file
# {
#         disable => "true";
# }
```
