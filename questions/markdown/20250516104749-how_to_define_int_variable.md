How to define int variable?

To define an integer variable in CFEngine, you can use the following syntax:

```cfengine3
vars:
    "variable_name" int => "42";
```

In the following example, `my_number` is the name of the variable, `int` specifies that it is an integer type and `42` is the value assigned to the variable.

```cfengine3
bundle agent example_int_variable
# @brief "Brief Description"
{
  vars:
      "my_number" int => "42";

  reports:
      "The number is: $(my_number)";
}
bundle agent __main__
{
  methods:
      "example_int_variable";
}
```

    R: The number is: 42

Note: You can also use suffixes to represent larger integer values, such as:

-   `k` = value times 1000
-   `m` = value times 1000^2
-   `g` = value times 1000^3
-   `K` = value times 1024
-   `M` = value times 1024^2
-   `G` = value times 1024^3

For example:

```cfengine3
bundle agent example_int_suffixes
# @brief "Brief Description"
{
  vars:

      "num[42k]" int => "42k";
      "num[42m]" int => "42m";
      "num[42g]" int => "42g";
      "num[42K]" int => "42K";
      "num[42M]" int => "42M";
      "num[42G]" int => "42G";

      "numbers" slist => getindices( num );

  reports:
      'num[$(numbers)]  == $(num[$(numbers)])';
}
bundle agent __main__
{
  methods:
      "example_int_suffixes";
}
```

    R: num[42M]  == 44040192
    R: num[42K]  == 43008
    R: num[42k]  == 42000
    R: num[42m]  == 42000000
    R: num[42G]  == 45097156608
    R: num[42g]  == 42000000000
