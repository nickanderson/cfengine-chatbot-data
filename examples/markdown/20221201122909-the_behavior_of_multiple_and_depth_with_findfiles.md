:::: captioned-content
::: caption
Generate a deep directory structure for finding things in
:::

``` {.bash org-language="sh" results="output" exports="both"}
exec 2>&1
rm -rf /tmp/test
mkdir -p /tmp/test/1/2/3/4/5/6/7/8/9/10/11/12/13/14/15/16/17/18/19/20
printf "0" > "/tmp/test/0.txt"
printf "1" > "/tmp/test/1/1.txt"
printf "2" > "/tmp/test/1/2/2.txt"
printf "3" > "/tmp/test/1/2/3/3.txt"
printf "4" > "/tmp/test/1/2/3/4/4.txt"
printf "5" > "/tmp/test/1/2/3/4/5/5.txt"
printf "6" > "/tmp/test/1/2/3/4/5/6/6.txt"
printf "7" > "/tmp/test/1/2/3/4/5/6/7/7.txt"
printf "8" > "/tmp/test/1/2/3/4/5/6/7/8/8.txt"
printf "9" > "/tmp/test/1/2/3/4/5/6/7/8/9/9.txt"
printf "10" > "/tmp/test/1/2/3/4/5/6/7/8/9/10/10.txt"
printf "11" > "/tmp/test/1/2/3/4/5/6/7/8/9/10/11/11.txt"
printf "12" > "/tmp/test/1/2/3/4/5/6/7/8/9/10/11/12/12.txt"
printf "13" > "/tmp/test/1/2/3/4/5/6/7/8/9/10/11/12/13.txt"
printf "14" > "/tmp/test/1/2/3/4/5/6/7/8/9/10/11/12/13/14.txt"
printf "15" > "/tmp/test/1/2/3/4/5/6/7/8/9/10/11/12/13/14/15.txt"
printf "16" > "/tmp/test/1/2/3/4/5/6/7/8/9/10/11/12/13/14/15/16.txt"
printf "17" > "/tmp/test/1/2/3/4/5/6/7/8/9/10/11/12/13/14/15/16/17.txt"
printf "18" > "/tmp/test/1/2/3/4/5/6/7/8/9/10/11/12/13/14/15/16/17/18.txt"
printf "19" > "/tmp/test/1/2/3/4/5/6/7/8/9/10/11/12/13/14/15/16/17/18/19.txt"
printf "20" > "/tmp/test/1/2/3/4/5/6/7/8/9/10/11/12/13/14/15/16/17/18/19/20.txt"
:
```
::::

:::: captioned-content
::: caption
Example Policy
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" tangle="the_behavior_of_multiple_stars_and_depth_with_findfiles.cf"}
bundle agent __main__
{
  vars:
      "i1" slist => findfiles( "/tmp/test/*.txt" );
      "i2" slist => findfiles( "/tmp/test/**.txt" );
      "i3" slist => findfiles( "/tmp/test/***.txt" );
      "i4" slist => findfiles( "/tmp/test/****.txt" );
      "i5" slist => findfiles( "/tmp/test/*****.txt" );
      "i6" slist => findfiles( "/tmp/test/******.txt" );
      "i7" slist => findfiles( "/tmp/test/*******.txt" );
      "i8" slist => findfiles( "/tmp/test/********.txt" );
      "i9" slist => findfiles( "/tmp/test/*********.txt" );
      "i10" slist => findfiles( "/tmp/test/*********.txt" );
      "i11" slist => findfiles( "/tmp/test/**********.txt" );

  reports:
      '1 findfiles( "/tmp/test/*.cf" )';
      '* $(i1)';

      '2 findfiles( "/tmp/test/**.cf" )';
      "** $(i2)";

      '3 findfiles( "/tmp/test/***.cf" )';
      "*** $(i3)";

      '4 findfiles( "/tmp/test/****.cf" )';
      "**** $(i4)";

      '5 findfiles( "/tmp/test/*****.cf" )';
      "***** $(i5)";

      '6 findfiles( "/tmp/test/******.cf" )';
      "****** $(i6)";

      '7 findfiles( "/tmp/test/*******.cf" )';
      "******* $(i7)";

      '8 findfiles( "/tmp/test/********.cf" )';
      "******** $(i8)";

      '9 findfiles( "/tmp/test/*********.cf" )';
      "********* $(i9)";

      '10 findfiles( "/tmp/test/**********.cf" )';
      "********* $(i10)";

      '11 findfiles( "/tmp/test/***********.cf" )';
      "********** $(i11)";

}
```
::::

``` example
R: 1 findfiles( "/tmp/test/*.cf" )
R: * /tmp/test/0.txt
R: 2 findfiles( "/tmp/test/**.cf" )
R: ** /tmp/test/0.txt
R: ** /tmp/test/1/1.txt
R: ** /tmp/test/1/2/2.txt
R: ** /tmp/test/1/2/3/3.txt
R: ** /tmp/test/1/2/3/4/4.txt
R: ** /tmp/test/1/2/3/4/5/5.txt
R: 3 findfiles( "/tmp/test/***.cf" )
R: *** /tmp/test/0.txt
R: *** /tmp/test/1/1.txt
R: *** /tmp/test/1/2/2.txt
R: *** /tmp/test/1/2/3/3.txt
R: *** /tmp/test/1/2/3/4/4.txt
R: *** /tmp/test/1/2/3/4/5/5.txt
R: 4 findfiles( "/tmp/test/****.cf" )
R: **** /tmp/test/0.txt
R: **** /tmp/test/1/2/2.txt
R: **** /tmp/test/1/2/3/4/4.txt
R: **** /tmp/test/1/2/3/4/5/6/6.txt
R: **** /tmp/test/1/2/3/4/5/6/7/8/8.txt
R: **** /tmp/test/1/2/3/4/5/6/7/8/9/10/10.txt
R: 5 findfiles( "/tmp/test/*****.cf" )
R: ***** /tmp/test/0.txt
R: ***** /tmp/test/1/2/2.txt
R: ***** /tmp/test/1/2/3/4/4.txt
R: ***** /tmp/test/1/2/3/4/5/6/6.txt
R: ***** /tmp/test/1/2/3/4/5/6/7/8/8.txt
R: ***** /tmp/test/1/2/3/4/5/6/7/8/9/10/10.txt
R: 6 findfiles( "/tmp/test/******.cf" )
R: ****** /tmp/test/0.txt
R: ****** /tmp/test/1/2/3/3.txt
R: ****** /tmp/test/1/2/3/4/5/6/6.txt
R: ****** /tmp/test/1/2/3/4/5/6/7/8/9/9.txt
R: ****** /tmp/test/1/2/3/4/5/6/7/8/9/10/11/12/12.txt
R: ****** /tmp/test/1/2/3/4/5/6/7/8/9/10/11/12/13.txt
R: ****** /tmp/test/1/2/3/4/5/6/7/8/9/10/11/12/13/14/15/16.txt
R: 7 findfiles( "/tmp/test/*******.cf" )
R: ******* /tmp/test/0.txt
R: ******* /tmp/test/1/2/3/3.txt
R: ******* /tmp/test/1/2/3/4/5/6/6.txt
R: ******* /tmp/test/1/2/3/4/5/6/7/8/9/9.txt
R: ******* /tmp/test/1/2/3/4/5/6/7/8/9/10/11/12/12.txt
R: ******* /tmp/test/1/2/3/4/5/6/7/8/9/10/11/12/13.txt
R: ******* /tmp/test/1/2/3/4/5/6/7/8/9/10/11/12/13/14/15/16.txt
R: 8 findfiles( "/tmp/test/********.cf" )
R: ******** /tmp/test/0.txt
R: ******** /tmp/test/1/2/3/4/4.txt
R: ******** /tmp/test/1/2/3/4/5/6/7/8/8.txt
R: ******** /tmp/test/1/2/3/4/5/6/7/8/9/10/11/12/12.txt
R: ******** /tmp/test/1/2/3/4/5/6/7/8/9/10/11/12/13.txt
R: ******** /tmp/test/1/2/3/4/5/6/7/8/9/10/11/12/13/14/15/16/17.txt
R: 9 findfiles( "/tmp/test/*********.cf" )
R: ********* /tmp/test/0.txt
R: ********* /tmp/test/1/2/3/4/4.txt
R: ********* /tmp/test/1/2/3/4/5/6/7/8/8.txt
R: ********* /tmp/test/1/2/3/4/5/6/7/8/9/10/11/12/12.txt
R: ********* /tmp/test/1/2/3/4/5/6/7/8/9/10/11/12/13.txt
R: ********* /tmp/test/1/2/3/4/5/6/7/8/9/10/11/12/13/14/15/16/17.txt
R: 10 findfiles( "/tmp/test/**********.cf" )
R: 11 findfiles( "/tmp/test/***********.cf" )
R: ********** /tmp/test/0.txt
R: ********** /tmp/test/1/2/3/4/5/5.txt
R: ********** /tmp/test/1/2/3/4/5/6/7/8/9/10/10.txt
R: ********** /tmp/test/1/2/3/4/5/6/7/8/9/10/11/12/13/14/15/16.txt
```
