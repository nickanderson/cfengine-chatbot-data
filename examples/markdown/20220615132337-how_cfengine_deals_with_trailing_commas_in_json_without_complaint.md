The last entry in `graylog`{.verbatim} has a trailing comma which isn\'t
valid JSON

:::: captioned-content
::: caption
/tmp/parametrage~dessources~.json
:::

``` {.json tangle="/tmp/parametrage_des_sources.json"}
{
    "mogodb" : {
        "nom_de_la_liste"     : "mogodb.list",
        "nom_clef_brute"     : "GPG-KEY-mongodb.gpg.key",
        "url_repo"           : "http://repo.mongodb.org/apt/debian buster/mongodb-org/4.2 main"
    },
    "elasticsearch" : {
        "nom_de_la_liste"     : "elasticsearch.list",
        "nom_de_la_clef"     : "GPG-KEY-elasticsearch.gpg.key",
        "url_repo"           : "https://artifacts.elastic.co/packages/oss-7.x/apt stable main"
    },

    "graylog" : {
        "nom_de_la_liste"     : "graylog.list",
        "nom_de_la_clef"     : "graylog-keyring.gpg.key",
        "url_repo"           : "https://packages.graylog2.org/repo/debian/ stable 4.2",
    }
}
```
::::

:::: captioned-content
::: caption
Shell command with stderr
:::

``` {.bash org-language="sh" results="output" exports="both"}
exec 2>&1
jq < /tmp/parametrage_des_sources.json  
python3 -m json.tool < /tmp/parametrage_des_sources.json  
:
```
::::

``` example
parse error: Expected another key-value pair at line 17, column 5
Expecting property name enclosed in double quotes: line 17 column 5 (char 688)
```

Though CFEngine handles it without complaint.
