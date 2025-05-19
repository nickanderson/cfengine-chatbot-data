<https://docs.cfengine.com/docs/3.21/reference-functions-maparray.html>

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" extra-opts="--show-evaluated-vars=default:main\\\\." tangle="/tmp/example.cf" command-in-result="t"}
bundle agent __main__
{
  vars:
    "data_container"
      data => '[{"name": "a", "value": "b"},{"name": "c", "value": "d"}]';

    # "data_container_index"
    #   slist => getindices("data_container");
    # "data_container_array[$(data_container_index)]"
    #   string => "$(data_container[$(data_container_index)][name])";

    # We reference back to 'd' because this.k is the positional location in the array returned by packagesmatching()
    "data_container_names" slist => sort( maparray( '$(data_container[$(this.k)][name])', data_container), lex );
}
```

``` example
# cf-agent --no-lock --log-level info --show-evaluated-vars=default:main\\. --file /tmp/example.cf
Variable name                            Variable value                                               Meta tags                                Comment
default:main.data_container              [{"name":"a","value":"b"},{"name":"c","value":"d"}]          source=promise
default:main.data_container_names         {"a","c"}                                                   source=promise
```

This can be useful if you want to iterate over package names returned
from `pacakgesmatching()~or ~pacakgeupdatesmatching()`.[^1][^2]

# Footnotes

[^1]: [packagesmatching()](id:b77acc35-20cf-4103-b031-ff420224ac72)

[^2]: [packageupdatesmatching()](id:6b1cebe6-f5e7-4057-afad-ccabd6878565)
