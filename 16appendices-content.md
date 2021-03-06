
# Exercises

Among the most challenging and valuable skills you can acquire is the skill of figuring out what you are supposed to do in a work situation. Work is usually underspecified because specification is itself half the work. If someone has to do all the work of specifying the work in detail, why pay you an exorbitant sum of money to do the easy half?

For each of the following exercises, begin by forming a plan of action rather than waiting for a plan to be given to you. Try to understand the problem, break it down into parts, and use what you know to plan the activities that lead to a solution. You will learn more if you ask the instructor to verify the plan you came up with than if you ask the instructor to present a plan.

## Exercise 1, Chaos Report

Identify projects that succeed, are challenged, or impaired, and relate the outcomes to the lists in the Chaos report article.

### Example

A software project I worked on was to visualize the relationships between regulatory documents from different authorities so the client could streamline their workflow to meet the requirements. The project spec assumed that the input was well-formed html. There was no budget to develop any filters or checks on the html. The visualization software did not work well and malformed html was found to be the cause. The client insisted that I identify the sources of malformed documents so he could take disciplinary action. He turned out to be the source of most of the malformed documents but his staff had been reluctant to identify his errors. Staff members were tasked with correcting some of his mistakes and the project scope was increased to include some filtering. The factors from the list involved included

1. lack of user input from staff members reluctant to include filtering in spec.
2. incomplete requirements due to 1 above.
3. changing requirements due to having to deal with filtering and different input
4. lack of executive support in that some time was wasted as the client leader tried to identify other culprits and staff tried to avoid being the one to bring the leader's problems into the open
5. technology incompetence on the part of the leader resulted in malformed html, e.g., renaming MS Word documents with an html extension and including them in the input data.
6. lack of resources in that, during specification, the client insisted that the desired budget for input filtering be\linebreak stricken.

## Exercise 2, Improvised ETL

Do NOT look at any published solution to this problem! Use the data.json file in Content > Resources as the input. Show the exact commands you would use to solve the problem (any language) in a plain text file in the exercise2 dropbox.

**Problem specification:**
You got your hands on some data that was leaked from a social network and you want to help the poor people.
Luckily you know a government service to automatically block a list of credit cards.
The service is a little old school though and you have to upload a CSV file in the exact format. The upload fails if the CSV file contains invalid data.
The CSV file should have two columns, Name and Credit Card. Also, it must be named using the following pattern:

`YYYYMMDD.csv`

The leaked data doesn't have credit card details for every user and you need to pick only the affected users.
You don't have much time to act.

### Example solution

The following is an example solution for the CSV Challenge problem discussed on Hacker News and presented on GitHub.

```bash
mkdir hw/csvChallenge
mv -i data.json hw/csvChallenge/
cd hw/csvChallenge/
```
Work with a copy if at all possible. Throughout I use files of the form `tmpNN` so I have an audit trail. If I spot problems, I can go back through the files to find the source. At the end of the process, I can delete them all and recreate them if I need to run the process again.

```bash
cp -p data.json tmp01
```
Get rid of rows without a credit card number. Throughout this solution, I'm using mostly Perl one-line scripts entered at the terminal. There are no files associated with them; the entire script is here. Since Perl is design to be used on semi-structured text files, it has some facilities for automatically looping over every line of input and applying the same one-line script. The redirection operators cause it to accept the file `tmp01` as input and send the output to file `tmp02`.

```bash
perl -ne 'print unless /"creditcard":null/' < tmp01 >tmp02
perl -ne 'print if     /"creditcard":null/' < tmp01 >tmp03
```
The number of rows in `tmp02` and `tmp03` should add up to the number of rows in `tmp01`.

```bash
wc tmp01 tmp02 tmp03
```

The output of the above is

```
  10002   45897 1612898 tmp01
   5006   22983  850410 tmp02
   4996   22914  762488 tmp03
  20004   91794 3225796 total
```
Remove all the surrounding material in each remaining row. One difference between these scripts and the scripts above is the use of `-pe` instead of `-ne`. That difference is simple. The `e` in both cases says that what follows is a one-line script. The `p` and `n` both cause the script to loop over every line of input. The `p` differs by automatically printing the output while the `n` does not automatically print. I included a `print` statement in the previous scripts so using a `p` loop would have printed two copies of every line.

```bash
perl -pe 's/"email.*"creditcard"://' < tmp02 > tmp04
perl -pe 's/{"name"://'              < tmp04 > tmp05
perl -pe 's/},//'                    < tmp05 > tmp06
```

Because this is a JSON file, opening and closing brackets remain. Remove them.

```bash
perl -ne 'print unless /^[\[\]]$/' < tmp06 >tmp07
```
Verify that the result makes sense. First, check that the above only removed the opening and closing brackets. The `diff` utility underlies many programs that you use in daily practice. You should learn to use it because it is used in so many different contexts. The following version is only the default. It can be made to operate in many different ways, including the use of patches to convert one file into another file. Here, it prints the line numbers where a difference occurs, line 1 in the first file and line 0 in the second file, then it shows what is different. The first file contains an open bracket. If the open bracket had been in the second file in the argument list, diff would have used `>` instead of `<` to mark the difference. The same thing is true for the next difference, which occurs on line 5006 of the first file and line 5004 of the second file. The difference here is JSON's closing bracket.

```bash
diff tmp06 tmp07
1d0
< [
5006d5004
< ]
```
Now remove some pieces of the file, sort them, remove repeated lines, and resort in the order of number of repeats. This provides an abbreviated picture that may be easier to visually inspect.

```bash
cut -f 1 -d " " tmp07 | sort | uniq -c | sort -n >tmp08
cut -f 2 -d ,   tmp07 | sort | uniq -c | sort -n >tmp09
```
This check reveals that one row still has an extraneous curly brace so remove it.

```bash
perl -pe 's/}//' < tmp07 > tmp10
```
The above step can be checked by examining differences between the two files.

```bash
diff tmp07 tmp10
5004c5004
< "Austin Langworth","1212-1221-1121-1234"}
---
> "Austin Langworth","1212-1221-1121-1234"
```
Remove more pieces of the file to make it easier to spot errors.

```bash
cut -f 1 -d , tmp10 | sort | uniq -c | sort -n >tmp11
```
The above step did not reveal much---no name appeared more than twice.

Another way might be to remove the names, then break the names apart.

```bash
cut -f 1 -d , tmp10 >tmp12
cut -f 2 -d " " tmp12 | sort | uniq -c | sort -n >tmp13
```

The problem with the above is that it gives me the false impression of errors. Some of the names appear to have no ending quote marks. By searching for some of the suspect names, such as Abbey, I realize that many names have three components instead of two. A better check might be to count the number of lines containing the string `","`. In the following output, I've put the output after the last two steps.

```bash
grep --only-matching "\",\"" tmp10 >tmp14
wc tmp10 tmp14
  5004  11706 199297 tmp10
  5004   5004  20016 tmp14
 10008  16710 219313 total
sort tmp14 | uniq -c
   5004 ","
```
The fact that each of these files have 5,004 rows indicates to me that the correct separator appears in each row. The last step requested by the challenge is to name the result in the following way.

```bash
cp -p tmp10 20150425.csv
```

That instruction seemed weird to me. All rows had the same creation date listed. One alternative might be to create a file corresponding to each creation date. Here is an old Perl script I wrote to create a file for each row number in a file with a format like

```
...
405	bla de bla
406	more of this
406	some of that
407	this and that
407	it just keeps on
...
```
The above fragment would be processed by the following script into three files, named 405, 406, and 407. The first one will contain one row, while the others will each contain two rows.

```perl
while (<>) {
  chomp($_);
  ($recno,$recbody) = split(/\t/,$_);
  if (open OUTFILE, ">>$recno") {
    printf OUTFILE "%s\n",$recbody
  } else {
    unless (open OUTFILE, ">$recno") {
      die "Cannot open $recno: $!";
    }
    printf OUTFILE "%s\n",$recbody
  }
}
```
The above script could be easily adapted to use the creation dates in the `data.json` file. All rows of this particular file have the same creation date, so you would have to add rows or change some of the dates to verify that the above script produces separate output files for each date.

In conclusion, I should emphasize that I would probably not use this method for such a small file. I would probably use the same substitutions but interactively in a Vim edit window. The syntax in the above Perl scripts will work in Vim except that the `print unless` commands would be replaced by `g/bla/d` where `bla` would be replaced by the regular expressions shown in the Perl scripts. For such a simple example, the same regular expressions could be used in any language, including Python, PHP, Powershell, Some non-P languages like awk, sed, Go, and Javascript use the same syntax for regular expressions. 

Only for more complicated examples do the different languages usually exhibit different regular expression syntax. The differences are well documented in a book called *Mastering Regular Expressions* by Friedl. The `regularexpression.info` website may be another useful resource.

### Another example solution
For the special case of transforming JSON, which is only one of many similar tasks, there is a "little language" called `jq` to act as a filter on JSON input. A solution using this language is extremely compact:

```bash
jq -r '.[]|select (.creditcard!=null) | [.name,.creditcard]
       |@csv' < data.json > $(date %Y%m%d).csv
```

The above `jq` "program" begins with the `-r` option to `jq` which says to provide "raw" output instead of strings. If you run this without `-r` you will get extra pairs of escaped quotes intended to make the results into strings.

There are then four `jq` statements which are pipelined so that the output of each one becomes the input to the next one. These pipelined statements are in one set of single quotes and together constitute a `jq` program.

The first `jq` statement is `.[]` which simply returns its input as an array. We need that because the remaining statements take arrays as inputs.

The second `jq` statement is a `select` statement which filters rows meeting a condition named in parentheses. In this case, the condition is that the credit card number not be null. The output of this statement will be all the JSON objects with a credit card number.

The third `jq` statement chooses only the keys whose values should be output. In this case we need only the name and credit card number.

The fourth `jq` statement is a special statement that converts its input into csv (comma separated values) format. There are special statements available in `jq` for a variety of formats. You should read the very long page at `man jq` for a complete list of these as well as the very many other features of `jq` any time you have to manage JSON.

Finally, there is the I/O redirection. `jq` reads from standard input so we can say `< data.json` to provide the input redirection. Next is the output redirection `>$(date +%Y%m%d).csv`. This is not `jq` but instead a feature of `bash` called command interpolation. Anything construct inside of `$()` is interpreted as a command and that command is run by `bash` and its output is inserted at exactly that point in the command before the outer command is run. In this case, the inner command is `date +%Y%m%d` which is a utility that can be run by `bash`. If you look at the page `man date`, you will find that saying `date` by itself outputs the current date and time in a default format but that you can alter that format by saying `+` followed by something called a format string. The format string consists of letters preceded by percent signs, each of which supplies some part of the date in some format. The format string `%Y%m%d` outputs the current date in YYYYmmdd format. That numeral is inserted at that point in the outer command causing the output to be directed to a file with the name 20170910.csv for instance if the command is run on that date.

### Small solutions to common problems
I recommend that you look at the writings of Peteris Krumins, a prolific blogger on *one-liners*, small solutions to common problems. You should also consider looking at sites like commandlinefu for similar advice on small solutions to common problems.

## Exercise 3, Version Control
For this exercise, you must create and submit a proper patch file to the exercise dropbox. To help you understand how to do so, here is an example of creating a patch file. After this example are more instructions for the exercise.

\begin{figure}[htbp]
\begin{center}
\tikzstyle{background rectangle}=[rounded corners,fill=yellow!05]
\definecolor{cmdcolor}{rgb}{0.95,0.00,0.05} 
\definecolor{excolor}{rgb}{0.3,0.4,0.5} 
\definecolor{arrowcolor}{rgb}{0.7,0.8,0.9} 
\begin{tikzpicture}[show background rectangle] 

\node(a) at (0,4)
  [shape=rectangle,draw=none,fill=excolor,text=white,scale=0.65]
  {\sf diffexample1.txt};
\node(b) at (1,3)
  [shape=ellipse,draw=none,fill=cmdcolor,text=white,scale=0.65]
  {\sf edit};
\node(c) at (1,2)
  [shape=rectangle,draw=none,fill=excolor,text=white,scale=0.65]
  {\sf new};
\node(d) at (0.5,1)
  [shape=ellipse,draw=none,fill=cmdcolor,text=white,scale=0.65]
  {\sf perl};
\node(e) at (0,0.25)
  [shape=rectangle,draw=none,fill=excolor,text=white,scale=0.65]
  {\sf output};

\node(f) at (2,4)
  [shape=ellipse,draw=none,fill=cmdcolor,text=white,scale=0.65]
  {\sf copy};

\node(g) at (3,4)
  [shape=rectangle,draw=none,fill=excolor,text=white,scale=0.65]
  {\sf old};
\node(h) at (3,2.75)
  [shape=ellipse,draw=none,fill=cmdcolor,text=white,scale=0.65]
  {\sf diff};
\node(i) at (3,1.75)
  [shape=rectangle,draw=none,fill=excolor,text=white,scale=0.65]
  {\sf patchfile};

\node(j) at (4,4)
  [shape=ellipse,draw=none,fill=cmdcolor,text=white,scale=0.65]
  {\sf copy};

\node(k) at (5.5,4)
  [shape=rectangle,draw=none,fill=excolor,text=white,scale=0.65]
  {\sf testforpatch};
\node(l) at (4.5,2.75)
  [shape=ellipse,draw=none,fill=cmdcolor,text=white,scale=0.65]
  {\sf patch};

\draw [->,very thick,arrowcolor] (a) to (b);
\draw [->,very thick,arrowcolor] (a) to (f);
\draw [->,very thick,arrowcolor] (b) to (c);
\draw [->,very thick,arrowcolor] (c) to (d);
\draw [->,very thick,arrowcolor] (d) to (e);
\draw [->,very thick,arrowcolor] (f) to (g);
\draw [->,very thick,arrowcolor] (g) to (j);
\draw [->,very thick,arrowcolor] (j) to (k);
\draw [->,very thick,arrowcolor] (c) to (h);
\draw [->,very thick,arrowcolor] (g) to (h);
\draw [->,very thick,arrowcolor] (h) to (i);
\draw [->,very thick,arrowcolor] (i) [bend right=30] to (l);
\draw [->,very thick,arrowcolor] (k) [bend right=30] to (l);
\draw [->,very thick,cmdcolor] (l) [bend right=30] to (k);
\draw [boundaryline,cmdcolor,text={equivalent}] (k) [bend right=340] to (5.75,1) to [bend right=270] (c);

\node (m) at (3.9,0.3) [text=red]{\sf \scriptsize equivalent after patch};
%  \draw [->,very thick,arrowcolor] (e) to [bend right=45] (p);
%  \node (itoc) at (2,3.5) {\sf \small esc};
%  \node (ctoi) at (2.1,2.2) {\sf \small i,I,a,A,o,O};
%  \node (etoc) at (0.6,1.0) {\sf \small esc,Enter};
%  \node (ctoe) at (-1.3,1.2) {\sf \small :};
\end{tikzpicture} 
\end{center}
\caption{using diff and patch}\label{fiDiffExample}
\end{figure}

### Example
Start with the `diffexample1.txt` file from myCourses. Make two copies of it.

```bash
cp -p diffexample1.txt old
cp -p diffexample1.txt new
```

Make the new copies executable.

```bash
chmod 700 old 
chmod 700 new 
```
Look up the commands we'll be using.

```bash
man diff
man patch
```
Modify `new` to read from `stdin` and write to `stdout`.

```bash
vi new
```
Verify that it works.

```bash
./new < data.json > $(date +"%Y%m%d").csv
```

The command above constructs the filename using the `date` command. That command can be run with any format of your choice. Try some examples.

```bash
date +"%Y"
date +"%Y%m"
date +"%Y%m%d"
date +"%y%m%d"
```
One way to check the output of the above command is to run `wc` which stands for word count. It counts lines, words, and characters in a file. I know off the top of my head that the result should have 5004 lines.

```bash
wc 20160208.csv 
```

Now that `old` and `new` differ, run `diff` on them in three different ways, ranging from less to more readable.

```bash
diff old new
diff -y old new
vimdiff old new
```
The first method, `diff old new`, may be used by a program called `patch` to turn one into the other. This is a basic utility in software repositories. Now create a patch and use it.

```bash
diff old new >patchfile
```
Now the contents of `patchfile` can be read by `patch` to turn `old` into `new`. Create a copy of `old` that we can use to test and to compare.

```bash
cp -p old testforpatch
```
Before making any changes, compare `testforpatch` to `old` and `new`.

```bash
diff testforpatch new
diff testforpatch old
```

Apply the patch using the form `patch` *infile* *patchfile*.

```bash
patch testforpatch patchfile
```
After applying the patch, compare `testforpatch` to `old` and `new`.

```bash
diff testforpatch new
diff testforpatch old
```

Figure \ref{fiDiffExample} shows the relationships between the files and processes involved in this process. The files are denoted by rectangles and the processes are denoted by ellipses. The file called *test for patch* is altered by the `patch` program so that it starts out as equivalent to `old` but is altered to be equivalent to `new`. The `diff` program produces the patchfile used by the `patch` program to make this change.

### Exercise Instructions

The input file is a Node.js script called `diffexample2.js`. That file needs to be changed in the following ways.

1. The only records to be saved are those without null credit cards and also without null email addreses.

2. The email address must be saved as well as the name and credit card number.

3. The input file name (such as `data.json`) must be accepted from the command line rather than being hardcoded. It is also acceptable to read from STDIN instead. Either approach is fine.

4. The date for the output file should be the date on which the script is run, not the date of the first record.

Submit only the patch that converts the `diffexample2.js` file into a version with the above improvements. Do not submit any other files. Do not zip the file. Name the file `patchfile.txt`. Any other files or filenames will be considered an error and will result in point deductions.

## Exercise 4, Make
The Make exercises will be completed in class.

## Exercise 5, Build
The build exercises will be completed in class.

## Exercise 6, Logging
`logback` is the new hotness. Your boss is disappointed that you are using `simpleLogger` since everybody who matters has switched to `logback`. Therefore, the boss wants to see a quick demo of `logback` pronto.

### Modify the wombat
Create a new file, `Wombat3.java` with all the contents of `Wombat2.java`. Make changes in this file so that you have two additional methods and both of those methods are exercised in main. Each of those methods should must set a different property, not temperature. The property may be fictitious. Each method should log at a different level and it must not be a level that we already logged in the `setTemperature` method. For reference, we already logged at the debug, info, and warn levels.

### Configure logging
This file must be logged using `logback` so the `simplelogger.properties` file is not useful to us. Figure out how to enter the properties into a `logback.xml` and create that file. Be sure to set it up so that a timestamp is added in the exact same format as in the `simplelogger` example. Be sure to set the logging level so that all levels are logged.

### Deliver
The Java program file, `Wombat3.java` and the logback properties file, `logback.xml`. In the comments of the dropbox, list the appropriate commands to compile and run `Wombat3`. Be sure to include exactly the correct commands or you will lose easy points!

## Exercise 7, Test Fixture

Create three program files, MainTester.java and two test program files, and one data file. The two test files can be any two of the test files you created for milestone 1, slightly modified to accept objects to test.

### MainTester.java
The MainTester class in this file will run all of the tests. You must read the following command line options.

`-h` prints a short help message on the console, telling what options are available and what they mean.

`-n` means that what follows is a test object.

`-f` means that what follows is the name of a test object file, containing one or more test objects.

The MainTester class should have default test objects in case no command line options are given. These default test objects should be supplied to the test classes instead of test objects being hard-coded into the test classes.

### Test classes
These are defined in
files you submitted in milestone 1,
files like `EdgeConnectorTest.java`.
Instead of having strings hardcoded like `"1|2|3|testStyle1"`, you will define test objects in MainTester with attributes like 1, 2, 3, testStyle1, and so on. These objects will then be passed to the two test classes so each test class file must be slightly rewritten to accept these objects.

### Data file
The data file will contain test objects so you have a choice as to whether to supply test objects on the command line or from a file.

### Additional info
Maybe it would help to say that you would not use `-n` and `-f` at the same time. You might provide a test object on the command line or you might provide test objects in a file.

Let's suppose that the test object has the attributes 1, 2, 3, teststyle1, and teststyle2. These are five attributes that you previously entered as five strings. When I run the program on the command line, I could provide these attributes. For example, I could say

```bash
java -cp bla.jar MainTester -n "1,2,3,teststyle1,teststyle2"
```

(Here I have substituted `bla.jar` for whatever is required on the command line. You have to specify the actual command in your readme file.)

Inside MainTester I can associate those five attributes with the appropriate object and pass them off to the test that uses them.

As an alternative, I might say

```bash
java -cp bla.jar MainTester -f testobjectfile.txt
```

where testobjectfile.txt is a file containing lines like

```
1,2,3,teststyle1,teststyle2
...
```

It is practical to put a bunch of test objects in a file. It is not so practical to name a bunch of test objects on the command line. Still, either way gives you the flexibility to rerun tests with different data without recompiling the programs.
 
### Deliverables
Deliver exactly five files, `MainTester.java`, two test files, the data file, and a `readme.md` file containing the commands to run. These files *must* be zipped. Name the file `ex7.zip`. I will place the files in a folder containing the project source code before I run MainTester. Note: If you have improved the project source code and would prefer that I test against your improved version, include it in a separate file called `revisedProjectSourceCode.zip`. Otherwise assume I will use the files in `finalProjectSourceCode.zip`.


## Exercise 8, Profiling
Find one complicated application you have written over the course of your school career.  If you can’t find one of your own to use, get one from Github. It can be written in Java, C#, PHP or Objective-C or any other language that has tools available for static analysis and profiling.

Next you need to run the code through a static code analyzer / optimizer and again through a profiler.  You can pick any tools you would like to use, either standalone or part of an IDE.

Write a 2--4 page summary on the results, 1--2 pages per part.  Include in your summary which tools you used, how you found them to use (easy, difficult, useful, etc). If you wrote the code, highlight where you could have improved your coding techniques.  You can also include if you disagree with the findings of the tools regarding readability, maintainability, or any other characteristics. Find one bug / issue that you don’t understand at first glance. Investigate and report on what it is / means.

In addition to the above, here are some points you can include: What did the analyzers find? What category of stuff? How much in each category? Do you agree with what they found?  If one is from early on and one later, did you make the same mistakes or did your coding techniques improve?  For profiling: what part of your app took longest to run? Used the most memory? Is there anything you could've done to improve either?

Please put your summary in a markdown file in the dropbox on myCourses by the due date on the dropbox. Any graphics should be in .png, .jpg, or .pdf format and referenced in the markdown file. Given time, we'll share the results in class.

# Exams

## Exam 1, Development methodologies through build utilities
 
Give short, succinct answers to about twenty-five questions on the material from development methodologies, diagramming development, version control, and build utilities.
 
## Exam 2, Testing through efficient code
Provide short answers about testing, error handling, logging, bug tracking, profiling, generic code, data driven code, reverse engineering, and efficient code.

## Exam 3, Application deployment through documentation
Answer questions about application deployment, help systems, packages, frameworks, namespaces, JARs, DLLs, and documentation.

# Milestones
 
Your team has been asked to take over a project for a programmer who has left the company.  The application currently reads in a file that is created by Edge Diagrammer (a data modeling tool) and generates the SQL statements required to create the tables for a MySQL database.  Your general overall task is to evaluate the application, refactor the code and make it as reusable and extensible as possible.

There are two goals for the finished application:

 1.  Make the application able to generate appropriate statements for several database management systems (e.g. MySQL, Oracle, etc.) for the current Edge Diagrammer file with minimal changes to the code. You need to think of how you can structure the code and provide it with the information it needs to create the tables on a different DBMS without major rewriting of the code. This will allow the company to switch vendors with minimal effort.

 2.  Have the application be able to replace Edge Diagrammer (it is expensive) with some other tool or schema description file (e.g. XML) and perform the same functions, again with minimal changes to the application itself the same as for the first objective. 

 These goals should be achieved by using any number of the methods we discussed or will discuss in class. You need to use a version control system for all of your work, including any documents that are required. Host this on Github and add me with admin rights as in the version control exercise. My username at Github is mickmcq.

## Milestone 1, Test plan
What you must test initially is that, given a specific input file
 (Edge Diagrammer, XML, etc), the application will generate
 an expected output file (the DDL for what the database is).
 You should use the `Courses.edg` file and the `CoursesEdgeDiagram.pdf` file to understand the input and its presentation. (It should not be necessary but you are welcome to obtain a free trial of Edge Diagrammer from Pacestar if it is still available.)
 In addition, you should run specific tests on four of five files mentioned later in these instructions.

This assignment mimics a typical situation in which you will find yourselves in the workplace. You will receive an unfinished codebase that does not work and will be asked to make it work.

In this project, I'm trying to break this process down into steps to make it easier for you. The very first thing you should do is to plan how you will test it and actually write several tests. That is what this milestone is for.

You know that the code is supposed to return database create and insert statements, so you can make up some create and insert statements and test them. For example, if you enter the following code into mysql, it will run without errors:

```sql
drop database if exists blah;
create database blah;
use blah;
create table foo (
  bar int,
  bas varchar(5),
  qux varchar(255)
);
```

\noindent
I can enter this into mysql by saying

```bash
mysql -u myusername -p < stmts.sql
```

\noindent
assuming I have entered it into a file called stmts.sql. It will run silently.

Next, I can run `mysqlcheck -u myusername -p -c blah` and I should get the following output.

```
blah.foo                      OK
```

\noindent
You should be able to test for that output using jUnit or a similar test framework. How do you know that the table should be named `foo`? You can look in the Courses.edg file to understand that. When you see a section of the file marked

```
## Figures & Connectors Section:
```

\noindent
you know that what follows are the descriptions of entities that will be turned into tables. That section begins with the following information:

```
Figure 1
{
  Style "Entity"
  Text "STUDENT"
```

\noindent
so you know that there will be a table called `STUDENT`. If you search for strings that look like this, you will soon find

```
Figure 2
{
  Style "Entity"
  Text "FACULTY"
```

\noindent
so now you know that there will be a table called `FACULTY`. You should be able to extract all the table names by writing code that checks for these strings. Then you can compare them to the names given in the output creation statements.

If your create statements fail to create a database at all, the above `mysqlcheck` command should return something like the following.

```
mysqlcheck: Got error: 1049: Unknown database 'blah'
when selecting the database
```

Suppose on the other hand that your program produces no file. Can you test that? If you took ISTE120 or ISTE121 you should be able to test for the existence of a file in Java. 
Suppose on the other hand that your program produces a file containing

```
create
```

\noindent
and nothing else. When I try to run that through mysql, it returns

```
ERROR 1064 (42000) at line 1: You have an error in your
SQL syntax; check the manual that corresponds to your
MySQL server version for the right syntax to use
near '' at line 1
```

\noindent
so I can check for whether the string "ERROR" occurs and assert that my test fails if the word "ERROR" is found in the output.

So there are multiple ways to check for multiple kinds of errors and you should employ multiple tests.

In addition to testing for output, it is possible to test several of the files in the project source code zip file (`ProjSrcCode.zip`). These include the following classes.

- CreateDDLMySQL
- EdgeConnector
- EdgeConvertCreateDDL
- EdgeField
- EdgeTable
- EdgeConvertFileParser

The other classes would be difficult to test directly using jUnit or a similar test framework. To simplify testing those classes, a working example of testing EdgeConnector is included in myCourses under Content > resources > workingExample.tar. To extract it after downloading, say

```bash
tar xvf workingExample.tar
```

\noindent
Switch to the workingExample folder and say

```bash
javac -cp .:junit-4.12.jar:hamcrest-core-1.3.jar \
  EdgeConnectorTest.java
java -cp .:junit-4.12.jar:hamcrest-core-1.3.jar \
  org.junit.runner.JUnitCore EdgeConnectorTest
```

\noindent
for which you should see output like the following:

```
JUnit version 4.12
.............
Time: 0.007

OK (13 tests)
```

You can play around with `EdgeConnectorTest.java` to make it fail some tests.
Your task (in addition to the above) is to test four more classes. You will have to create four files, such as

```
CreateDDLMySQLTest.java
EdgeConvertCreateDDLTest.java
EdgeConvertFileParserTest.java
EdgeFieldTest.java
EdgeTableTest.java
```

depending on which one you choose to omit (only do four of
the five).

## Milestone 2, SDLC
A description of what type of SDLC you plan on
 using and why.  Include in this any milestones, backlogs,
 etc. that are required for the initial phases of your
 chosen SDLC. Also include a flowChart of what the code
 does. This flowchart should reflect the code running correctly. In other words, if you have discovered bugs, don't flowchart the bugs.

## Milestone 3, Deployment strategy
This should include deployment packaging. It should be possible for a novice to run the program from your package. This could be as simple as a jar file and a readme file, if it works correctly, or could be a more complicated installer program as mentioned in the Application Deployment section of this document.

## Milestone 4, Help system
This should be a system, not a document. It should be accessible to a user of the program. It should run from the help menu item in the program. There should be at least two screens of help, demonstrating that you can add help to the program. It is not acceptable to provide separate help outside the program, even though that might be worthwhile. This exercise demonstrates that you can add help within the system.

## Milestone 5, Refactored abstracted code
Include with the code a separate
 description of what you changed and why.   Also include how
 your code solves the goals above---in other words, what you
 would have to do to use a different DBMS or modeling
 program’s file or some other description file.


# Software

## Improvised etl (extract / transfer / load)

### Vim

- split
- vsplit
- folding
- regex
- global
- syntax highlighting
- hex edit
- dbext
- vimdiff

### tmux / wemux

- split-window
- select-window
- list-buffers
- copy-mode
- paste-buffer
- swap-pane
- select-pane - list-buffers
- copy-mode
- paste-buffer- list-buffers
- copy-mode
- paste-buffer
- resize-pane
- detach

### shell utilities

- bash
- autoconf
- automake
- awk
- bg
- cat
- chgrp
- chmod
- chown
- column
- curl
- cut
- diff
- df
- du
- echo
- fg
- find
- file
- fmt
- grep
- head
- httrack
- jobs
- join
- less
- ln
- make
- man
- paste
- printf
- ps
- pwd
- readline
- recode
- rsync
- sed
- sort
- stat
- strings
- tail
- tar
- tcpdump
- time
- top
- uname
- uniq
- uptime
- wget
- which
- whoami
- xargs

### ldap utils (undecided)

- ldapadd
- ldapcompare
- ldapcomplete
- ldapexop
- ldapmodify
- ldapmodrdn
- ldappasswd
- ldapsearch
- ldapurl
- ldapwhoami

## Junit

## Ant

## Log4J

## Cucumber

## SAX

# References


