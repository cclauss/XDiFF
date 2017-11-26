# Why do I want to use a database?
A database allows you to compare the results of how the software was executed when using different inputs, versions, implementations or operating systems. All the test cases to be evaluated are contained in one place and any issues found across multiple scenarios will be detected. Not only there is value on exploiting the vulnerabilities with the higher risk, but also the ones that affect multiple pieces of software at the same time. The performance and capabilities of SQLite for the fuzzer were proven to be better than MySQL and Redis.

## How's the database structure?
The initial analysis of the database was constructed around how to fuzz programming languages. They allow you to create any piece of software, so they will have access to all the functionalities. With this in mind, this is the basic look of a plain SQLite database used by XDiFF:

<pre>
# sqlite3 dbs/plain.sqlite 
SQLite version 3.11.0 2016-02-15 17:29:24
Enter ".help" for usage hints.
sqlite> .tables
<b>function</b>              <b>fuzz_software</b>         <b>fuzz_testcase_result</b>
<b>fuzz_constants</b>        <b>fuzz_testcase</b>         <b>value</b>      
</pre>

There are two tables where you may want to manually insert or edit some values:
* **value**: contains the items that will replace the ```[[test]]``` values in *function*. If you don't have a 'function', you can use the values in here with input fuzzers.

* **function**: contains what you want to fuzz. There is a special keyword ```[[test]]``` that gets replaced by the values contained in **value**. For example, if you would like to fuzz the print() function, you would normally want to have in here ```print([[test]])```.

The tables that start with 'fuzz_' are generated by XDiFF:
* **fuzz_testcase**: contains the combination of *function* and *value*

* **fuzz_software**: contains the software defined in *software.ini*

* **fuzz_testcase_result**: contains the result of executing the software defined in *fuzz_software* with the input defined in *fuzz_testcase*

* **fuzz_constants**: contains internal constant values used by the fuzzer

## Grab a sample database
Let's grab a copy of the plain.sqlite database:
```
cp dbs/plain.sqlite shells.sqlite
```

## Insert testcases

Data can be inserted in the database using a ***sqlite3*** parser or using the ***dbaction.py*** script. In case your test case/s are in a file, you may want to insert it directly into the database like this for example:
<pre>
echo "insert into value values (readfile('<b>sample_file</b>'))"|sqlite3 <b>shells.sqlite</b> 
</pre>

## Insert combinations of functions/values

If you have a certain function (or portion of code) that you want to fuzz with certain values, you can insert first the functions into the database:
<pre>
./dbaction.py -d <b>shells.sqlite</b> -t function -i "<b>foo([[test]])</b>"
</pre>

Insert the values that you want to use to fuzz the piece of code within the function table:
<pre>
./dbaction.py -d <b>shells.sqlite</b> -t value -i "<b>bar</b>"
</pre>

Then you can generate the permutations:
<pre>
./dbaction.py -d <b>shells.sqlite</b> -g 1
2017-11-20 22:06:24,901 INFO dbaction: Values: 1 - Functions: 1
2017-11-20 22:06:24,901 INFO dbaction: <b>Testcases generated: 1</b>
2017-11-20 22:06:24,902 INFO dbaction: Time required: 0.0 seconds
</pre>

You can later confirm how the information everything looks like:
<pre>
./dbaction.py -d <b>shells.sqlite</b> -t <b>fuzz_testcase</b> -p
----------------------------------------------------------------------------------------------------------
| fuzz_testcase (1 rows)                                                                                 |
----------------------------------------------------------------------------------------------------------
| id               | testcase                                                                            |
----------------------------------------------------------------------------------------------------------
| 1                | <b>foo(bar)</b>                                                                            |
----------------------------------------------------------------------------------------------------------
</pre>

## Extending the detection

Part of the install process required to create a command named ```canaryfile``` (and ```canaryfile.bat```). When this file gets executed, it produces a specific output that can be later analyzed. Basically, you want the string ```canaryfile``` as part of your values.

Moreover, if the software may open network connections, you also want to define the ```canaryhost``` as part of the potential values to be used. The connections will be detected locally and be included as part of the output to be analyzed.

# What's next?

You want to define [the software](https://github.com/IOActive/XDiFF/wiki/The-software)