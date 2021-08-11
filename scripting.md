# Shell script tutorial

## Creating a script

use `chmod 755` or `chmod +x` with filename.sh to make the script executable.

+ Use `which bash` command to find out the location of shell scripts.

```bash
#! /bin/bash

echo "Hello World"
```

+ Use shebang `#! /path/to/bash` to differentiate what process is the script for.

+ comments begin with `#`

---

## Variables

There are some rules about variable names:

+ Variable names may consist of alphanumeric characters (letters and numbers) and underscore characters.

+ The first character of a variable name must be either a letter or an underscore.

+ Spaces and punctuation symbols are not allowed.

```bash
#! /bin/bash

NAME="Sean"

# Below are two different ways of calling a variable
echo "My name is $NAME"
echo "My name is ${NAME}"
```

The word `variable` implies a value that changes, and in many applications, variables are used this way.
However, some variables are used as a constant. 
A constant is just like a variable in that it has a name and contains a value.
The difference is that the value of a constant does not change.
The shell makes no distinction between variables and constants; they are mostly for the programmer's convenience.

> A common convention is to use uppercase letters to designate constants and lower case letters for true variables.

```bash
#!/bin/bash

# Program to output a system information page

TITLE="System Information Report For $HOSTNAME"

echo "<html>
        <head>
          <title>$TITLE</title>
        </head>
        <body>
          <h1>$TITLE</h1>
        </body>
      </html>"
```

> The shell variable `HOSTNAME` is the network name of the machine.

The shell actually does probide a way to enforce the immutability of constants,
through the use of the `declare` built-in command with the `-r` (read-only) option:

`declare -r TITLE="Page Title"`

the shell would prevent any subsequent assignment to TITLE.
This feature is rarely used, but it exists for very formal scripts.

As we have seen, variables are assigned values this way:

`variable=value`

where `variable` is the name of the variable and `value` is a string.
Unlike some other programming languages, the shell does not care about the type of data assigned to a variable;
it treats them all as strings.
You can force the shell to restrict the assignment to integers by using the `declare` command with the `-i` option,
but, like setting variables as read-only, this is rarely done.

> Note that in an assignment, there must be no spaces between the variable name, the equal sign, and the value.

So, what can the value consist of? It can have anything that we can expand into a string.

```bash
a=z                   # Assign the string "z" to variable a.
b="a string"          # Embedded spaces must be within quotes.
c="a string and $b"   # Other expansions such as variables can be expanded into the assignment
d="$(ls -l foo.txt)"  # Results of a command.
e=$((5 * 7))          # Arithmetic expansion.
f="\t\ta string\n"    # Escape sequences such as tabs and newlines
```

Multiple variable assignments may be done on a single line.

`a=5 b="a string"`

During expansion, variable names may be surrounded by optional braces, `{}`. 
This is useful in cases where a variable name becomes ambiguous because of its surrounding context.

Here, we try to change the name of a file from `myfile` to `myfile1`, using a variable:

```bash
$ filename="myfile"
$ touch "$filename"
$ mv "$filename" "$filename1"
mv: missing destination file operand after `myfile'
```

This attempt fails because the shell interprets the second argument of the `mv` command as a new (and empty) variable.
The problem can be overcome this way:

`$ mv "$filename" "${filename}1"`

By adding the surrounding braces, the shell no longer interprets the trailing `1` as part of the variable name.

> It's good practice to enclose variables and command substitutions in double quotes to limit the effects of word-splitting by the shell.
> Quoting is especially important when a variable might contain a filename.

We'll add some data to our report:
namely, the date and time the report was created and the username of the creator.

```bash
#!/bin/bash

# Program to output a system information page

TITLE="System Information Report For $HOSTNAME"
CURRENT_TIME="$(date +"%x %r %Z")"
TIMESTAMP="Generated $CURRENT_TIME, by $USER"

echo "<html>
        <head>
          <title>$TITLE</title>
        </head>
        <body>
          <h1>$TITLE</h1>
          <p>$TIMESTAMP</p>
        </body>
      </html>
```

---

## Here Documents

There is another way of outputting text called a `here document` or `here script`.
A here document is an additional form of I/O redirection in which we embed a body of text
into our script and feed it into the standard input of a command.
It works like this:

```bash
command << token
text
token
```

where `command` is the name of command that accepts standard input and `token` is a string
used to indicate the end of the embedded text.

Here we'll modify our script to use a here document:

```bash
#!/bin/bash

# Program to output a system information page

TITLE="System Information Report For $HOSTNAME"
CURRENT_TIME="$(date +"%x %r %Z")"
TIMESTAMP="Generated $CURRENT_TIME, by $USER"

cat << _EOF_
<html>
  <head>
    <title>$TITLE</title>
  </head>
  <body>
    <h1>$TITLE</h1>
    <p>$TIMESTAMP</p>
  </body>
</html>
_EOF_
```

Instead of using `echo`, our script now uses `cat` and a here document.
The string `_EOF_` (meaning *end of file*, a common convention) was selected as the token
and marks the end of the embedded text.

> Note that the token must appear alone and that there must not be trailing spaces on the line.

So, what's the advantage of using a here document?
It's mostly the same as `echo`, except that, by default, single and double quotes within here documents
lose their special meaning to the shell.

Here's a command line example:

```bash
$ foo="some text"
$ cat << _EOF_
> $foo
> "$foo"
> '$foo'
> \$foo
> _EOF_
some text
"some text"
'some text'
$foo
```

As we can see, the shell pays no attention to the quotation marks.
It treats them as ordinary characters.
This allows us to embed quotes freely within a here document.

Here documents can be used with any command that accepts standard input.
In this example, we use a here document to pass a series of commands to the `ftp` program to retrieve a file from
a remote FTP server:

```bash
#!/bin/bash

# Script to retrieve a file via FTP

FTP_SERVER=ftp.nl.debian.org
FTP_PATH=/debian/dists/stretch/main/installer-amd64/current/images/cdrom
REMOTE_FILE=debian-cd_info.tar.gz

ftp -n << _EOF_
open $FTP_SERVER
user anonymous me@linuxbox
cd $FTP_PATH
hash
get $REMOTE_FILE
bye
_EOF_
ls -l "$REMOTE_FILE"
```

> If we change the redirection operator from `<<` to `<<-`, the shell will ignore
leading tab characters (but not spaces) in the here document.
This allows a here document to be indented, which can improve readability.

```bash
#!/bin/bash

# Script to retrieve a file via FTP

FTP_SERVER=ftp.nl.debian.org
FTP_PATH=/debian/dists/stretch/main/installer-amd64/current/images/cdrom
REMOTE_FILE=debian-cd_info.tar.gz

ftp -n <<- _EOF_
  open $FTP_SERVER
  user anonymous me@linuxbox
  cd $FTP_PATH
  hash
  get $REMOTE_FILE
  bye
_EOF_
ls -l "$REMOTE_FILE"
```

> This feature can be somewhat problematic because many text editors (and programmers themselves)
will prefer to use spaces instead of tabs to achieve indentation in their scripts.

---

## Shell functions

```bash
#! /bin/bash

function sayHello() {
  echo "Hello World"
}

sayHello
```

### Top down design

Our script currently performs the following steps to generate the HTML document:

1. Open page.
2. Open page header.
3. Set page title.
4. Close page header.
5. Oepn page body.
6. Output page heading.
7. Output timestamp
8. Close page body
9. Close page

For our next stage of development, we will add some tasks between steps 7 and 8.
This will include the following:

+ System uptime and load.

  + This is the amount of time since the last shutdown or reboot and the average number of tasks
  currently running on the processor over several time intervals.

+ Disk space.

  + This is the overall use of space on the system's storage devices.

+ Home space.

  + This is the amount of storage space being used by each user.

If we had a command for each of these tasks, we could add them to our script simply through command substitution.

```bash
#!/bin/bash

# Program to output a system information page

TITLE="System Information Report For $HOSTNAME"
CURRENT_TIME="$(date +"%x %r %Z")"
TIMESTAMP="Generated $CURRENT_TIME, by $USER"

cat << _EOF_
<html>
  <head>
    <title>$TITLE</title>
  </head>
  <body>
    <h1>$TITLE</h1>
    <p>$TIMESTAMP</p>
    $(report_uptime)
    $(report_disk_space)
    $(report_home_space)
  </body>
</html>
_EOF_
```

We could create these additional commands in two ways.
We could write three separate scripts and place them in a directory listed in our `PATH`,
or we could embed the scripts within our program as *shell functions*.

Shell functions are "mini-scripts" that are located inside other scripts and can act as autonomous programs.

Shell functions have two syntatic forms.

First here is the more formal form:

```bash
function name {
  commands
  return
}
```

Here is the simpler (and generally preferred) form:

```bash
name () {
  commands
  return
}
```

where `name` is the name of the function and `commands` is a series of commands contained within the function.

Both forms are equivalent and may be used interchangeably.

The following is a script that demonstrates the use of a shell function:

```bash
#!/bin/bash

# Shell function demo

function step2 {
  echo "Step 2"
  return
}

# Main program starts here

echo "Step 1"
step2
echo "Step 3"
```

The function's `return` command terminates the function and returns control to the program
at the line following the function call, and the final `echo` command is executed.

> Note that for function calls to be recognized as shell functions and not interpreted as the 
names of external programs, shell function definitions must appear in the script before they are called.

We'll add minimal shell function definitions to our script:

```bash
#!/bin/bash

# Program to output a system information page

TITLE="System Information Report For $HOSTNAME"
CURRENT_TIME="$(date +"%x %r %Z")"
TIMESTAMP="Generated $CURRENT_TIME, by $USER"

report_uptime () {
  return
}

report_disk_space () {
  return
}

report_home_space () {
  return
}

cat << _EOF_
<html>
  <head>
    <title>$TITLE</title>
  </head>
  <body>
    <h1>$TITLE</h1>
    <p>$TIMESTAMP</p>
    $(report_uptime)
    $(report_disk_space)
    $(report_home_space)
  </body>
</html>
_EOF_
```

Shell function names follow the same rules as variables.

> A function must contain at least one command. The `return` command (which is optional) satisfies the requirement.

### Local Variables

In the scripts we have written so far, all the variables (including constants) have been *global variables*.
Global variables maintain their existence throughout the program.
This is fine for many things, but it can sometimes complicate the use of shell functions.

Inside shell functions, it is often desirable to have *local variables*.
Local variables are accessible only within the shell function in which they are defined and cease to exist
once the shell function terminates.

Having local variables allows the programmer to use variables with names that may already exist,
either in the script globally or in other shell functions, without having to worry about potential name conflicts.

Here is an example script that demonstrates how local variables are defined and used:

```bash
#!/bin/bash

# local-vars: script to demonstrate local variables

foo=0   # global variable foo

funct_1 () {
  local foo   # variable foo local to funct_1
  foo=1
  echo "funct_1: foo = $foo"
}

funct_2 () {
  local foo  # variable foo local to funct_2
  foo=2
  echo "funct_2: foo = $foo" 
}

echo "global: foo = $foo"
funct_1
echo "global: foo = $foo"
funct_2
echo "global: foo = $foo"
```

We see that the assignment of values to the local variable `foo` within both shell functions
has ino effect on the value of `foo` defined outside the functions.

This feature allows shell functions to be written so that they remain independent of each other
and of the script in which they appear.
This is valuable because it helps prevent one part of a program from interfering with another.
It also allows shell functions to be written so that they can be portable.
That is, they may be cut and pasted from script to script, as needed.

### Keep Scripts Running

While developing our program, it is useful to keep the program in a runnable state.
By doing this, and testing frequently, we can detect errors early in the development process.
This will make debugging problems much easier.
By adding empty functions, called *stubs* is programmer-speak,
we can verify the logical flow of our program at an early stage.

When constructing a stub, it's a good idea to include something that provides feedback to the programmer,
which shows the logical flow is being carried out.

We could change the functions to include some feedback:

```bash
#!/bin/bash

# Program to output a system information page

TITLE="System Information Report For $HOSTNAME"
CURRENT_TIME="$(date +"%x %r %Z")"
TIMESTAMP="Generated $CURRENT_TIME, by $USER"

report_uptime () {
  echo "Function report_uptime executed."
  return
}

report_disk_space () {
  echo "Function report_disk_space executed."
  return
}

report_home_space () {
  echo "Function report_home_space executed."
  return
}

cat << _EOF_
<html>
  <head>
    <title>$TITLE</title>
  </head>
  <body>
    <h1>$TITLE</h1>
    <p>$TIMESTAMP</p>
    $(report_uptime)
    $(report_disk_space)
    $(report_home_space)
  </body>
</html>
_EOF_
```

If we run the script again, we can now see that, in fact, our three functions are being executed.

With our function framework in place and working, it's time to flesh out some of the function code.

First, here's the `report_uptime` function:

```bash
report_uptime () {
  cat <<- _EOF_
          <h2>System Uptime</h2>
          <pre>$(uptime)</pre>
_EOF_
  return
}
```

We use a here document to output a section header and the output of the `uptime` command,
surrounded by `<pre>` tags to preserve the formatting of the command.

The `report_disk_space` function is similar:

```bash
report_disk_space () {
  cat <<- _EOF_
          <h2>Disk Space Utilization</h2>
          <pre>$(df -h)</pre>
_EOF_
  return
}
```

This function uses the `df -h` command to determine the amount of disk space.

Lastly, we'll build the `report_home_space` function:

```bash
report_home_space () {
  cat <<- _EOF_
          <h2>Home Space Utilization</h2>
          <pre>$(du -sh /home/*)</pre>
_EOF_
  return
}
```

We use the `du` command with the `-sh` options to perform this task.
This, however, is not a complete solution to the problem.
While it will work on some systems (Ubuntu, for example), it will not work on others.
The reason is that many systems set the permissions of home directories to prevent them from being world-readable,
which is a reasonable security measure.
On these systems, the `report_home_space` function will only work if our script is run with superuse privileges.
A better solution would be to have the script adjust its behavior according to the privileges of the user.

---

## Flow Control: Branching with IF

How can we make our report-generator script adapt to the privileges of the user running the script?
The solution to this problem will require us to find a way to "change directions" within our script,
based on the results of a test.
In programming terms, we need the program to *branch*.

```bash
#! /bin/bash

read -p "Enter your age: " AGE

if [ "$AGE" -ge "19"]
then
  echo "You can drink"
elif [ "$AGE" -lt "19" ]
then
  echo "You aren't allowed to drink"
else
  echo "Please input your age"
fi
```

```bash
#! /bin/bash

FILE="myscript.sh"
if [ -f "$FILE" ]
then
  echo "$FILE is a file"
fi
```

The `if` statement has the following syntax:

```bash
if commands; then
  commands
[elif commands; then
  commands...]
[else
  commands]
fi
```

where `commands` is a list of commands.

### Exit Status

Commands (including the scripts and shell functions we write) issue a value to the system when they terminate,
called an *exit status*.
This value, which is an integer in the range of 0 to 255, indicates the success or failure of the command's execution.

By convention, a value of zero indicates success and any other value indicates failure.

The shell provides a parameter that we can use to examine the exit status:

```bash
$ ls -d /usr/bin
/usr/bin
$ echo $?
0
$ ls -d /bin/usr
ls: cannot access /bin/usr: No such file or directory
$ echo $?
2
```

In this example, we execute the `ls` command twice.
The first time, the command executes successfully.
If we display the value of the parameter `$?`, we see that it is zero.
We execute the `ls` command a second time (specifying a nonexistent directory), producing an error,
and examine the parameter `$?` again.
This time it contains a 2, indicating that the command encountered an error.

Some commands use different exit status values to provide diagnostics for errors,
while many commands simply exit with a value of 1 when they fail.

> Man pages often include a section entitled "Exit Status", describing what codes are used.
However, a zero always indicates success.

The shell provides two extremely simple builtin commands that do nothing except terminate with either a 0 or 1 exit status.
The `true` command always executes successfully, and the `false` command always executes unsuccessfully.

```shell
$ true
$ echo $?
0
$ false
$ echo $?
1
```

We can use these commands to see how the `if` statement works.
What the `if` statement really does is evaluate the success or failure of commands.

```shell
$ if true; then echo "It's true."; fi
It's true
$ if false; then echo "It's true."; fi
$
```

The command `echo "It's true."` is executed when the command following `if` executes successfully
and is not executed when the command following `if` does not execute successfully.

If a list of commands follows `if`, the last command in the list is evaluated.

```shell
$ if false; true; then echo "It's true."; fi
It's true
$ if true; false; then echo "It's true."; fi
$
```

### Using `test`

By far, the command used most frequently with `if` is `test`.
The `test` command performs a variety of checks and comparisons.

It has two equivalent forms. The first, shown here:

`test expression`

And the second, more popular form, here:

`[ expression ]`

where `expression` is an expression that is evaluated as either true or false.

The `test` command returns an exit status of 0 when the expression is true and a status of 1 when the expression is false.

> It is interesting to note that both `test` and `[` are actually commands.
> In bash they are builtins, but they also exist as programs in `/usr/bin` for use with other shells.
> The expression is actually just its arguments with the `[` command requiring that the `]` character be provided
as its final argument.

check | usage
---|---
file1 -ef file2 | file1 and file2 have the same inode numbers (the two filenames refer to the same file by hard linking)
file1 -nt file2 | file1 is newer than file2
file1 -ot file2 | file1 is older than file2
-b file | file exists and is a block-special (device) file
-c file | file exists and is a character-special (device) file
-d file | file exists and is a directory
-e file | file exists
-f file | file exists and is a regular file
-g file | file exists and is set-group-ID
-G file | file exists and is owned by the effective group ID
-k file | file exists and has its "sticky bit" set
-L file | file exists and is a symbolic link
-O file | file exists and is owned by the effective user ID
-p file | file exists and is a named pipe
-r file | file exists and is readable
-s file | file exists and has non-zero size
-S file | file exists and is a network socket
-t fd | fd is a file descriptor directed to/from the terminal. This can be used to determine whether standard input/output/error is being redirected
-u file | file exists and is setuid (if user id is set on a file)
-w file | file exists and is writable
-x file | file exists and is executable

Here we have a script that demonstrates some of the file expressions:

```shell
#!/bin/bash

# test-file: Evaluate the status of a file

FILE=~/.bashrc

if [ -e "$FILE" ]; then
  if [-f "$FILE" ]; then
    echo "$FILE is a regular file."
  fi
  if [ -d "$FILE" ]; then
    echo "$FILE is a directory."
  fi
  if [ -r "$FILE" ]; then
    echo "$FILE is readable."
  fi
  if [ -w "$FILE" ]; then
    echo "$FILE is writable."
  fi
  if [ -x "$FILE" ]; then
    echo "$FILE is executable/searchable."
  fi
else
  echo "$FILE does not exist"
  exit 1
fi

exit
```

The script evaluates the file assigned to the constant FILE and displays its results as the evaluation is performed.

> There are tow interesting things to note about this script.
> First, notice how the parameter $FILE is quoted within the expressions.
> This is not required to syntactically complete the expression;
> rather, it is a defense against the parameter being empty or containing only whitespace.
> If the parameter expansion of $FILE were to result in an empty value, it would cause an error
> (the operators would be interpreted as non-null strings rather than operators).
> Using the quotes around the parameter ensures that the operator is always followed by a string,
> even if the string is empty.
> Second, notice the presence of the `exit` command neawr the end of the script.
> The `exit` command accepts a single, optional argument, which becomes the script's exit status,
> When no argument is passed, the exit status defaults to the exit status of the last command executed.
> Using `exit` in this way allows the script to indicate failure if $FILE expands to the name of a nonexistent file.
> The `exit` command appearing on the last line of the script is there as a formality.
> When a script "runs off the end" (reaches end of file), it terminates with an exit status of the last command executed.

Similarly, shell functions can return an exit status by including an integer argument to the return command.
If we were to convert the previous script to a shell function to include it in a larger program,
we could replace the `exit` commands with return statements and get the desired behavior.

```shell
test_file () {
  # test-file: Evaluate the status of a file

  FILE=~/.bashrc

  if [ -e "$FILE" ]; then
    if [ -f "$FILE" ]; then
      echo "$FILE is a regular file."
    fi
    if [ -d "$FILE" ]; then
      echo "$FILE is a directory."
    fi
    if [ -r "$FILE" ]; then
      echo "$FILE is readable."
    fi
    if [ -w "$FILE" ]; then
      echo "$FILE is writable."
    fi
    if [ -x "$FILE" ]; then
      echo "$FILE is executable/searchable."
    fi
  else
    echo "$FILE does not exist"
    return 1
  fi
}
```

### String Expressions

Expression | Is true if:
---|---
string | `string` is not null
-n string | The length of `string` is greater than zero
-z string | The length of `string` is zero
string1 = string2 | string1 and string2 are equal. Single or double equal signs may be used. The use of double equal signs is greatly preferred, but it is not POSIX compliant
string1 == string2 | Note that both `=` and `==` expressions require a space between the strings and the operator. The evaluation will not work and always evaluate to true if otherwise.
string1 != string2 | string1 and string2 are not equal
string1 > string2 | string1 sorts after string2
string1 < string2 | string1 sorts before string2

> The > and < expression operators must be quoted (or escaped with a backslash) when used with `test`.
> If they are not, they will be interpreted by the shell as redirection operators, with potentially destructive results.
> Also note that while the `bash` documentation states that the sorting order conforms to the collation order of the current locale, it does not.
> ASCII (POSIX) order is used in versions of `bash` up to and including 4.0.
> This problem was fixed in version 4.1.

Here is a script that incorporates string expressions:

```shell
#!/bin/bash

# test-string: evaluate the value of a string

ANSWER=maybe

if [ -z "$ANSWER" ]; then
  echo "There is no answer." >&2
  exit 1
fi

if [ "$ANSWER" = "yes" ]; then
  echo "The answer is YES."
elif [ "$ANSWER" = "no" ]; then
  echo "The answer is NO."
elif [ "$ANSWER" = "maybe" ]; then
  echo "The answer is MAYBE."
else
  echo "The answer is UNKNOWN."
fi
```

In this script, we evaluate the constant `ANSWER`.
We first determine whether the string is empty.
If it is, we terminate the script and set the exit status to 1.

> Notice the redirection that is applied to the `echo` command.
> This redirects the error message "There is no answer." to standard error, which is the proper thing to do with error messages.

If the string is not empty, we evaluate the value of the string to see whether it is equal to either "yes", "no", or "maybe".
We do this by using `elif`, which is short for "else if".
By using `elif`, we are able to construct a more complex logical test.

### Integer Expressions

To compare values as integers rather than as strings, we can use the expressions:

operators | usage
---|---
-ge | greater or equal
-gt | greater than
-eq | equals
-ne | not equal
-lt | less than
-le | less than or equal

Here is a script that demonstrates them:

```shell
#!/bin/bash

# test-integer: evaluate the value of an integer.

INT=-5

if [ -z "$INT" ]; then
  echo "INT is empty." >&2
  exit 1
fi

if [ "$INT" -eq 0 ]; then
  echo "INT is zero."
else
  if [ "$INT" -lt 0 ]; then
    echo "INT is negative."
  else
    echo "INT is positive."
  fi
  if [ $((INT % 2)) -eq 0 ]; then
    echo "INT is even."
  else
    echo "INT is odd."
  fi
fi
```

> The interesting part of the script is how it determines whether an integer is even or odd.
> By performing modulo 2 operation on the number, which divides the number by 2 and returns the remainder, it can tell whether the number is odd or even.

### A More Modern Version of test

Modern versions of `bash` include a compound command that acts as an enhanced replacement for `test`.
It uses the following syntax:

`[[ expression ]]`

where, like `test`, `expression` is an expression that evaluates to either a true or false result.
The `[[  ]]` command is similar to `test` (it supports all of its expressions) but adds an important new string expression.

`string1 =~ regex`

This returns true if `string1` is matched by the extended regular expression `regex`.
This opens up a lot of possibilities for performing such tasks as data validation.
In our earlier example of the integer expressions, the script would fail if the constant `INT` contained anything except an integer.
The script needs a way to verify that the constant contains an integer.
Using the `[[  ]]` with the `=~` string expression operator, we could improve the script this way:

```shell
#!/bin/bash

# test-integer2: evaluate the value of an integer.

INT=-5

if [[ "$INT" =~ ^-?[0-9]+$ ]]; then
  if [ "$INT" -eq 0 ]; then
    echo "INT is zero."
  else
    if [ "$INT" -lt 0 ]; then
      echo "INT is negative."
    else
      echo "INT is positive."
    fi
    if [ $((INT % 2)) -eq 0 ]; then
      echo "INT is even."
    else
      echo "INT is odd."
    fi
  fi
else
  echo "INT is not an integer." >&2
  exit 1
fi
```

> By applying the regular expression, we are able to limit the value of INT to only strings that begin with an optional minus sign, followed by one or more numerals.
> This expression also eliminates the possibility of empty values.

Another feature of `[[  ]]` is that the `==` operator supports pattern matching the same way pathname expansion does.
Here's an example:

```shell
$ FILE=foo.bar
$ if [[ $FILE == foo.* ]]; then
> echo "$FILE matches pattern 'foo.*'"
> fi
foo.bar matches pattern 'foo.*'
```

### ((  )) - Designed for Integers

In addition to the `[[  ]]` compound command, `bash` also provides the `((  ))` compound command, which is useful for operating on integers.
It supports a full set of arithmetic evaluations.

`((  ))` is used to perform arithmetic truth tests.
An *arithmetic truth test* results in true if the result of the arithmetic evaluation is non-zero.

```shell
$ if ((1)); then echo "It is true."; fi
It is true.
$ if ((0)); then echo "It is true."; fi
$
```

Using `((  ))`, we can slightly simplify the test-integer2 script like this:

```shell
#!/bin/bash

# test-integer2a: evaluate the value of an integer.

INT=-5

if [[ "$INT" =~ ^-?[0-9]+$ ]]; then
  if ((INT == 0)); then
    echo "INT is zero."
  else
    if ((INT < 0)); then
      echo "INT is negative."
    else
      echo "INT is positive."
    fi
    if (( ((INT % 2)) == 0 )); then
      echo "INT is even."
    else
      echo "INT is odd."
    fi
  fi
else
  echo "INT is not an integer." >&2
  exit 1
fi
```

> Notice that we use less-than and greater-than signs and that `==` is used to test for equivalence.
> This is a more natural-looking syntax for working with integers.
> Notice too, that because the compound command `((  ))` is part of the shell syntax rather than an ordinary command and it deals only with integers, it is able to recognize variables by name and does not require expansion to be performed.

### Combining Expressions

It's also possible to combine expressions to create more complex evaluations.
Expressions are combined by using logical operators.
There are three logical operations for `test` and `[[  ]]`. They are AND, OR, and NOT.
`test` and `[[  ]]` use different operators to represent these operations:

Operation | test | [[  ]] and ((  ))
---|---|---
AND | -a | &&
OR | -o | \|\|
NOT | ! | !

Here's an example of an AND operation.
The following script determines whether an integer is within a range of values:

```shell
#!/bin/bash

# test-integer3: determine if an integer is within a specified range of values

MIN_VAL=1
MAX_VAL=100

INT=50

if [[ "$INT" =~ ^-?[0-9]+$ ]]; then
  if [[ "$INT" -ge "$MIN_VAL" && "$INT" -le "$MAX_VAL" ]]; then
    echo "$INT is within $MIN_VAL to $MAX_VAL."
  else
    echo "$INT is out of range."
  fi
else
  echo "INT is not an integer." >&2
  exit 1
fi
```

In this script, we determine whether the value of integer INT lies between the values of MIN_VAL and MAX_VAL.
This is performed by a single use of `[[  ]]`, which includes two expressions separated by the `&&` operator.

We could also have coded this using test:

```shell
if [ "$INT" -ge "$MIN_VAL" -a "$INT" -le "$MAX_VAL" ]; then
  echo "$INT is within $MIN_VAL to $MAX_VAL."
else
  echo "$INT is out of range."
fi
```

The `!` negation operator reverses the outcome of an expression.
It returns true if an expression is false, and it returns false if an expression is true.
In the following script, we modify the logic of our evaluation to find values of `INT` that are outside the specified range:

```shell
#!/bin/bash

# test-integer4: determine if an integer is outside a specified range of values

MIN_VAL=1
MAX_VAL=100

INT=50

if [[ "$INT" =~ ^-?[0-9]+$ ]]; then
  if [[ ! ("$INT" -ge "$MIN_VAL" && "$INT" -le "$MAX_VAL") ]]; then
    echo "$INT is outside $MIN_VAL to $MAX_VAL."
  else
    echo "$INT is in range."
  fi
else
  echo "INT is not an integer." >&2
  exit 1
fi
```

We also include parentheses around the expression for grouping.
If these were not included, the negation would only apply to the first expression and not the combination of the two.
Coding this with `test` would be done this way:

```shell
if [ ! \( "$INT" -ge "$MIN_VAL" -a "$INT" -le "$MAX_VAL" \) ]; then
  echo "$INT is outside $MIN_VAL to $MAX_VAL."
else
  echo "$INT is in range."
fi
```

Since all expressions and operators used by `test` are treated as command arguments by the shell (unlike `[[  ]]` and `(( )))`, characters that have special meaning to `bash`, such as <, >, (, and ), must be quoted or escaped.

> `test` is traditional (and part of the POSIX specification for standard shells, which are often used for system startup scripts), whereas `[[  ]]` is specific to `bash` (and a few other modern shells).
> It's important to know how to use `test` since it is widely used, but `[[  ]]` is clearly more useful and is easier to code, so it is preferred for modern scripts.

### Control Operators

`bash` provides two control operators that can perform branching.
The `&&` (AND) and `||` (OR) operators work like the logical operators in the `[[  ]]` compound command.

Here is the syntax for `&&`:

`command1 && command2`

Here is the syntax for `||`:

`command1 || command2`

With the `&&` operator, `command1` is executed, and `command2` is executed if, and only if, `command1` is successful.
With the `||` operator, `command1` is executed and `command2` is executed if, and only if, `command1` is unsuccessful.

In practical terms, it means that we can do something like this:

`mkdir temp && cd temp`

This will create a directory named `temp`, and if it succeeds, the current working directory will be changed to `temp`.
The second command is attempted only if the `mkdir` command is successful.

Likewise, a command like this:

`[[ -d temp ]] || mkdir temp`

will test for the existence of the directory `temp`, and only if the test fails will the directory be created.
This type of construct is handy for handling errors in scripts.

For example, we could do this in a script:

`[ -d temp ] || exit 1`

If the script requires the directory `temp` and it does not exist, then the script will terminate with an exit status of 1.

How could we make our `sys_info_page` script detect whether the user had permission to read all the home directories?
With our knowledge of `if`, we can solve the problem by adding this code to the `report_home_space` function:

```shell
report_home_space() {
  if [[ "$(id -u)" -eq 0 ]]; then
    cat <<- _EOF_
            <h2>Home Space Utilization (All Users)</h2>
            <pre>$(du -sh /home/*)</pre>
_EOF_
  else
    cat <<- _EOF_
            <h2>Home Space Utilization ($USER)</h2>
            <pre>$(du -sh $HOME)</pre>
_EOF_
  fi
  return
}
```

We evaluate the output of the `id` command.
With the `-u` option, `id` outputs the numeric user ID number of the effective user.
The superuser is always ID zero, and every other user is a number greater than zero.
Knowing this, we can construct two different here documents, one taking advantage of superuser privileges and the other restricted to the user's own home directory.

---

## Reading user input

```bash
#! /bin/bash

# Read in user input to variable after prompt
read -p "Enter your age: " AGE
echo "Your age is $AGE"
```

### read - Read Values from Standard Input

The `reawd` builtin command is used to read a single line of standard input.
This command can be used to read keyboard input or, when redirection is employed, a line of data from a file.
The command has the following syntax:

`read [-options] [variable...]`

where `options` is one or more of the available options and `variable` is the name of one or more variables used to hold the input balue.
If no variable name is supplied, the shell variable `REPLY` contains the line of data.

Basically, `read` assigns fields from standard input to the specified variables.
If we modify our integer evaluation script to use `read`, it might look like this:

```shell
#!/bin/bash

# read-integer: evaluate the value of an integer.

echo -n "Please enter an integer -> "
read int

if [[ "$int" =~ ^-?[0-9]+$ ]]; then
  if [ "$int" -eq 0 ]; then
    echo "$int is zero."
  else
    if [ "$int" -lt 0 ]; then
      echo "$int is negative."
    else
      echo "$int is positive."
    fi
    if [ $((int % 2)) -eq 0 ]; then
      echo "$int is even."
    else
      echo "$int is odd."
    fi
  fi
else
  echo "Input value is not an integer." >&2
  exit 1
fi
```

We use `echo` with the -n option (which suppresses the trailing newline on output) to display a prompt,
and then we use `read` to input a value for the variable `int`.

`read` can assign input to multiple variables, as shown in this script:

```shell
#!/bin/bash

# read-multiple: read multiple values from keyboard

echo -n "Enter one or more values > "
read var1 var2 var3 var4 var5

echo "var1 = '$var1'"
echo "var2 = '$var2'"
echo "var3 = '$var3'"
echo "var4 = '$var4'"
echo "var5 = '$var5'"
```

In this scripot, we assign and display up to five values.
Notice how `read` behaves when given different numbers of values, shown here:

```shell
$ read-multiple
Enter one or more values > a b c d e
var1 = 'a'
var2 = 'b'
var3 = 'c'
var4 = 'd'
var5 = 'e'

$ read-multiple
Enter one or more values > a
var1 = 'a'
var2 = ''
var3 = ''
var4 = ''
var5 = ''

$ read-multiple
Enter one or more values > a b c d e f g
var1 = 'a'
var2 = 'b'
var3 = 'c'
var4 = 'd'
var5 = 'e f g'
```

If `read` receives fewer than the expected number, the extra variables are empty,
while an excessive amount of input results in the final variable containing all of the extra input.

If no variables are listed after the `read` command, a shell variable, `REPLY`, will be assigned all the input.

```shell
#!/bin/bash

# read-single: read multiple values into default variable

echo -n "Enter one or more values > "
read

echo "REPLY = '$REPLY'"
```

Running this script results in this:

```shell
$ read-single
Enter one or more values > a b c d
REPLY = 'a b c d'
```

### Options

`read` supports the options described below:

Option | Description
---|---
-a array | Assign the input to `array`, starting with index zero
-d delimiter | The first character in the string `delimiter` is used to indicate the end of input, rather than a newline character
-e | Use Readline to handle input. This permits input editing in the same manner as the command line
-i string | Use `string` as a default reply if the user simply presses `ENTER`. Requires the -e option
-n num | Read `num` characters of input, rather than an entire line
-p prompt | Display a prompt for input using the string `prompt`
-r | Raw mode. Do not interpret backslash characters as escapes
-s | Silent mode. Do not echo characters to the display as they are typed. This is useful when inputting passwords and other confidential information
-t  seconds | Timeout. Terminate input after `seconds`. `read` returns a non-zero exit status if an input times out
-u fd | Use input from file descriptor `fd`, rather than standard input

Using the various options, we can do interesting things with `read`.
For example, with the `-p` option, we can provide a prompt string.

```shell
#!/bin/bash

# read-single: read multiple values into default variable

read -p "Enter one or more values > "

echo "REPLY = '$REPLY'"
```

with the `-t` and `-s` options, we can write a script that reads "secret" input and times out if the input is not completed in a specific time.

```shell
#!/bin/bash

# read-secret: input a secret passphrase

if read -t 10 -sp "Enter secret passphrase > " secret_pass; then
  echo -e "\nSecret passphrase = '$secret_pass'"
else
  echo -e "\nInput timed out" >&2
  exit 1
fi
```

The script prompts the user for a secret passphrase and waits ten seconds for input.
If the entry is not completed within the specified time, the script exits with an error.
Because the `-s` option is included, the characters of the pass phrase are not echoed to the display as they are typed.

It's possible to supply the user with a default response using the `-e` and `-i` options together.

```shell
#!/bin/bash

# read-default: supply a default value if user presses Enter key.

read -e -p "What is your user name?" -i $USER
echo "You answered: '$REPLY'"
```

In this script, we prompt the user to enter a username and use the environment variable `USER` to provide a default value.
When the script is run, it display the default string, and if the user simply presses `ENTER`, `read` will assign the default string to the `REPLY` variable.

### IFS

Normall, the shell performs word-splitting on the input provided to `read`.
As we have seen, this means that multiple words separated by one or more spaces become separate items on the input line and are assigned to separate variables by `read`.
This behavior is configured by a shell variable named `IFS` (for Internal Field Separator).
The default value of `IFS` contains a space, a tab, and a newline character, each of which will separate items from one another.

We can adjust the value of `IFS` to control the separation of fields input to `read`.
For example, the */etc/passwd* file contains lines of data that use the colon character as a field separator.
By changing the value of `IFS` to a single colon, we can use `read` to input the contents of */etc/passwd* and successfully separate fields into different variables.
Here we have a script that does just that:

```shell
#!/bin/bash

# read-ifs: read fields from a file

FILE=/etc/passwd

read -p "Enter a username > " user_name

file_info="$(grep "^$user_name:" $FILE)"

if [ -n "$file_info" ]; then
  IFS=":" read user pw uid gid name home shell <<< "$file_info"
  echo "User =      '$user'"
  echo "UID =       '$uid'"
  echo "GID =       '$gid'"
  echo "Full Name = '$name'"
  echo "Home Dir. = '$home'"
  echo "Shell =     '$shell'"
else
  echo "No such user '$user_name'" >&2
  exit 1
fi
```

This script prompts the user to enter the username of an account on the system and then displays the different fields found in the user's record in the */etc/passwd* file.
The script contains two interesting lines.
The first is as follows:

`file_info=$(grep "^$user_name:" $FILE)

This line assigns the results of a `grep` command to the variable `file_info`.
The regular expression used by `grep` assures that the username will match only a single line in the */etc/passwd* file.

The second interesting line is this one:

`IFS=":" read user pw uid gid name home shell <<< "$file_info"`

The line consists of three parts: a variable assignment, a `read` command with a list of variable names as arguments, and a strange new redirection operator. We'll look at the variable assignment first.

The shell allows one or more variable assignments to take place immediately before a command.
These assignments alter the environment for the command that follows.
The effect of the assignment is temporary, changing the environment only for the duration of the command.
In our case, the value of `IFS` is changed to a colon character.
Alternatively, we could have coded it this way:

```shell
OLD_IFS="$IFS"
IFS=":"
read user pw uid gid name home shell <<< "$file_info"
IFS="$OLD_IFS"
```

where we store the value of `IFS`, assign a new value, perform the `read` command, and then restore `IFS` to its original value.
Clearly, placing the variable assignment in front of the command is a more concise way of doing the same thing.

The `<<<` operator indicates a *here string*. A here string is like a here document, only shorter, consisting of a single string.
In our example, the line of daya from the */etc/passwd* file is fed to the standard input of the `read` command.
We might wonder why this rather oblique method was chosen rather than this:

`echo "$file_info" | IFS=":" read user pw uid gid name home shell`

### You can't pipe `read`

While the `read` command normally takes input from the standard input, you cannot do this:

`echo "foo" | read`

We would expect this to work, but it does not.
The command will appear to succeed, but the `REPLY` variable will always be empty. Why is this?

The explanation has to do with the way the shell handles pipelines.
In bash (and other shells such as sh), pipelines create subshells.
These are copies of the shell and its environment that are used to exectue the command in the pipeline.
In the previous example, `read` is executed in a subshell.

Subshells in Unix-like systems create copies of the environment for the processes to use while they execute.
When the processes finishes, the copy of the environment is destroyed.
This means that a subshell can never alter the environment of its parent process.
`read` assigns variables, which then become part of the environment.
In the previous example, `read` assigns the value foo to the variable REPLY in its subshell's environment, but when the command exits, the subshell and its environment are destroyed, and the effect of the assignment is lost.

Using here strings is one way to work around this behavior.

### Validating Input

With our new ability to have keyboard input comes an additional programming challenge: validating input.
Often the difference between a well-written program and a poorly written one lies in the program's ability to deal with the unexpected.
Frequently, the unexpected appears in the form of bad input.

We've done a little of this with our evaluation programs in the previous chapter, where we checked the values of integers and screened out empty values and non-numeric characters.
It is important to perform these kinds of programming checks every time a program receives input to guard against invalid data.
This is especially important for programs that are shared by multiple users.
Omitting these safeguards in the interests of economy might be excused if a program is to be used once and only by the author to perform some special task.
Even then, if the program performs dangerous tasks such as deleting files, it would be wise to include data validation, just in case.

Here we have an example program that validates various kinds of input:

```shell
#!/bin/bash

# read-validate: validate input

invalid_input () {
  echo "Invalid input '$REPLY'" >&2
  exit 1
}

read -p "Enter a single item > "

# input is empty (invalid)
[[ -z "$REPLY" ]] && invalid_input

# input is multiple items (invalid)
(( "$(echo "$REPLY" | wc -w)"  > 1 )) && invalid_input

# is input a valid filename?
if [[ "$REPLY" =~ ^[-[:alnum:]\._]+$ ]]; then
  echo "'$REPLY' is a valid filename."
  if [[ -e "$REPLY" ]]; then
    echo "And file '$REPLY' exists."
  else
    echo "However, file '$REPLY' does not exist."
  fi

  # is input a floating point number?
  if [[ "$REPLY" =~ ^-?[[:digit:]]*\.[[:digit:]]+$ ]]; then
    echo "'$REPLY' is a floating point number."
  else
    echo "'$REPLY' is not a floating point number."
  fi

  # is input an integer?
  if [[ "$REPLY" =~ ^-?[[:digit:]]+$ ]]; then
    echo "'$REPLY' is not an integer."
  else
    echo "'$REPLY' is not an integer."
  fi
else
  echo "The string '$REPLY' is not a valid filename."
fi
```

This script prompts the user to enter an item.
The item is subsequently analyzed to determine its contents.
As we can see, the script makes use of many of the concepts that we have covered thus far, including shell functions, `[[  ]]`, `((  ))`, the control operator `&&`, and `if`, as well as a healthy dose of regular expressions.

### Menus

A common type of interactivity is called *menu-driven*.
In menu-driven programs, the user is presented with a list of choices and is asked to choose one.
For example, we could imagine a program that presented the following:

```
Please Select:

1. Display System Information
2. Display Disk Space
3. Display Home Space Utilization
0. Quit

Enter selection [0-3] >
```

Using what we learned from writing our `sys_info_page` program, we can construct a menu-driven program to perform the tasks on the previous menu.

```shell
#!/bin/bash

# read-menu: a menu driven system information program

clear
echo "
Please select:

1. Display System Information
2. Display Disk Space
3. Display Home Space Utilization
0. Quit
"

read -p "Enter selection [0-3] > "

if [[ "$REPLY" =~ ^[0-3]$ ]]; then
  if [[ "$REPLY" == 0 ]]; then
    echo "Program terminated."
    exit
  fi
  if [[ "$REPLY" == 1 ]]; then
    echo "Hostname: $HOSTNAME"
    uptime
    exit
  fi
  if [[ "$REPLY" == 2 ]]; then
    df -h
    exit
  fi
  if [[ "$REPLY" == 3 ]]; then
    if [[ "$(id -u)" -eq 0 ]]; then
      echo "Home Space Utilization (All Users)"
      du -sh /home/*
    else
      echo "Home Space Utilization ($USER)"
      du -sh "$HOME"
    fi
    exit
  fi
else
  echo "Invalid entry." >&2
  exit 1
fi
```

This script is logically divided into two parts.
The first part displays the menu and inputs the response from the user.
The second part identifies the response and carries out the selected action.
Notice the use of the `exit` command in this script.
It is used here to prevent the script from executing unnecessary code after an action has been carried out.
The presence of multiple exit points in a program is generally a bad idea (it makes program logic harder to understand), but it works in this script.

---

## Flow Control: Looping with While/Until

In the previous chapter, we developed a menu-driven program to produce various kinds of system information.
The program works, but it still has a significant usability problem.
It executes only a single choice and then terminates.
Even worse, if an invalid selection is made, the program terminates with an error, without giving the user an opportunity to try again.
It would be better if we could somehow construct the program so that it could repeat the menu display and selection over and over, until the user chooses to exit the program.

```bash
#! /bin/bash

LINE=1

while read -r CURRENT_LINE
do
  echo "$LINE: $CURRENT_LINE"
  ((LINE++))
done < "test.txt"
```

### while

Let's say we wanted to display five numbers in sequential order from 1 to 5.
A bash script could be constructed as follows:

```shell
#!/bin/bash

# while-count: display a series of numbers

count=1

while [[ "$count" -le 5 ]]; do
  echo "$count"
  count=$((count + 1))
done
echo "Finished."
```

When executed, this script displays the following:

```
$ while-count
1
2
3
4
5
Finished.
```

The syntax of the `while` command is as follows:

`while commands; do commands; done`

Like `if`, `while` evaluates the exit status of a list of commands.
As long as the exit status is zero, it performs the commands inside the loop.
In the previous script, the variable `count` is created and assigned an initial value of 1.
The `while` command evaluates the exit status of the `[[  ]]` compound command.
As long as the `[[  ]]` command returns an exit status of zero, the commands within the loop are executed.
At the end of each cycle, the `[[  ]]` command is repeated.
After 5 iterations of the loop, the value of `count` has increased to 6, the `[[  ]]` command no longer returns an exit status of zero, and the loop terminates.
The program continues with the next statement following the loop.

We can use a *while loop* to improve the `read-menu` program from the previous chapter.

```shell
#!/bin/bash

# while-menu: a menu driven system information system information program

DELAY=3   # Number of seconds to display results

while [[ "$REPLY" != 0 ]]; do
  clear
  cat <<- _EOF_
          Please Select:

          1. Display System Information
          2. Display Disk Space
          3. Display Home Space Utilization
          0. Quit

_EOF_
  read -p "Enter selection [0-3] > "
  
  if [[ "$REPLY" =~ ^[0-3]$ ]]; then
    if [[ "$REPLY" == 1 ]]; then
      echo "Hostname: $HOSTNAME"
      uptime
      sleep "$DELAY"
    fi
    if [[ "$REPLY" == 2 ]]; then
      df -h
      sleep "$DELAY"
    fi
    if [[ "$REPLY" == 3 ]]; then
      if [[ "$(id -u)" -eq 0 ]]; then
        echo "Home Space Utilization (All Users)"
        du -sh /home/*
      else
        echo "Home Space Utilization ($USER)"
        du -sh "$HOME"
      fi
      sleep "$DELAY"
    fi
  else
    echo "Invalid entry."
    sleep "$DELAY"
  fi
done
echo "Program terminated."
```

By enclosing the menu in a `while` loop, we are able to have the program repeat the meny display after each selection.
The loop continues as long as `REPLY` is not equal to 0 and the menu is displayed again, giving the user the opportunity to make another selection.
At the end of each action, a `sleep` command is executed, so the program will pause for a few seconds to allow the results of the selection to be seen before the screen is cleared and the meny is redisplayed.
Once `REPLY` is equal to 0, indicating the "quit" selection, the loop terminates and execution continues with the line following `done`.

### Breaking Out of a Loop

`bash` provides two builtin commands that can be used to control program flow inside loops.
The `break` command immediately terminates a loop, and program control resumes with the next statement following the loop.
The `continue` command causes the remainder of the loop to be skipped, and program control resumes with the next iteration of the loop.
Here we see a version of the `while-menu` program incorporating both `break` and `continue`:

```shell
#!/bin/bash

# while-menu2: a menu driven system information program

DELAY=3   # Number of seconds to display results

while true; do
  clear
  cat <<- _EOF_
          Please Select:

          1. Display System Information
          2. Display Disk Space
          3. Display Home Space Utilization
          0. Quit

_EOF_
  read -p "Enter selection [0-3] > "

  if [[ "$REPLY" =~ ^[0-3]$ ]]; then
    if [[ "$REPLY" == 1 ]]; then
      echo "Hostname: $HOSTNAME"
      uptime
      sleep "$DELAY"
      continue
    fi
    if [[ "$REPLY" == 2 ]]; then
      df -h
      sleep "$DELAY"
      continue
    fi
    if [[ "$REPLY" == 3 ]]; then
      if [[ "$(id -u)" -eq 0 ]]; then
        echo "Home Space Utilization (All Users)"
        du -sh /home/*
      else
        echo "Home Space Utilization ($USER)"
        du -sh "$HOME"
      fi
      sleep "$DELAY"
      continue
    fi
    if [[ "$REPLY" == 0 ]]; then
      break
    fi
  else
    echo "Invalid entry."
    sleep "$DELAY"
  fi
done
echo "Program terminated."
```

In this version of the script, we set up an *endless loop* (one that never terminates on its own) by using the `true` command to supply an exit status to `while`.
Since `true` will always exit with an exit status of zero, the loop will never end.
This is a surprisingly common scripting technique.
Since the loop will never end on its own, it's up to the programmer to provide some way to break out of the loop when the time is right.
In this script, the `break` command is used to exit the loop when the 0 selection is chosen.
The `continue` command has been included at the end of the other script choices to allow for more efficient execution.
By using `continue`, the script will skip over code that is not needed when a selection is identified.
For example, if the 1 selection is chosen and identified, there is no reason to test for the other selections.

### until

The `until` compound command is much like `while`, except instead of exiting a loop when a non-zero exit status is encountered, it does the opposite.
An *until loop* continues until it receives a zero exit status.
In our `while-count` script, we continued the loop as long as the value of the `count` variable was less than or equal to 5.
We could get the same result by coding the script with `until`.

```shell
#!/bin/bash

# until-count: display a series of numbers

count=1

until [[ "$count" -gt 5 ]]; do
  echo "$count"
  count=$((count + 1))
done
echo "Finished."
```

By changing the test expression to `$count -gt 5`, `until` will terminate the loop at the correct time.
The decision of whether to use the `while` or `until` loop is usually a matter of choosing the one that allows the clearest test to be written.

### Reading Files with Loops

`while` and `until` can process standard input.
This allows files to be processed with `while` and `until` loops.
In the following example, we will display the contents of the *distros.txt* file used in earlier chapters:

```shell
#!/bin/bash

# while-read: read lines from a file

while read distro version release; do
  printf "Distro: %s\tVersion: %s\tReleased: %s\n" \
    "$distro" \
    "$version" \
    "$release"
done < distros.txt
```

To redirect a file to the loop, we place the redirection operator after the `done` statement.
The loop will use `read` to input the fields from the redirected file.
The `read` command will exit after each line is read, with a zero exit status until the end-of-file is reached.
At that point, it will exit with a non-zero exit status, thereby terminating the loop.
It is also possible to pipe standard input into a loop.

```shell
#!/bin/bash

# while-read2: read lines from a file

sort -k 1,1 -k 2n distros.txt | while read distro version release; do
  printf "Distro: %s\tVersion: %s\tReleased: %s\n" \
        "$distro" \
        "$version" \
        "$release"
done
```

Here we take the output of the `sort` command and display the stream of text.
However, it is important to remember that since a pipe will execute the loop in a subshell, any variables created or assigned within the loop will be lost when the loop terminates.

---

## Troubleshooting

### Syntactic Errors

One general class of errors is *syntactic*.
Syntactic errors involve mistyping some element of shell syntax.
The shell will stop executing a script when it encounters this type of error.

In the following discussions, we will use this script to demonstrate common types of errors.

```shell
#!/bin/bash

# trouble: script to demonstrate common errors

number=1

if [ $number = 1 ]; then
  echo "Number is equal to 1."
else
  echo "Number is not equal to 1."
fi
```

As written, this script runs successfully.

```
$ trouble
Number is equal to 1.
```

### Missing Quotes

Let's edit our script and remove the trailing quote from the argument following the first `echo` command.

```shell
#!/bin/bash

# trouble: script to demonstrate common errors

number=1

if [ $number = 1 ]; then
  echo "Number is equal to 1."
else
  echo "Number is not equal to 1."
fi
```

Here is what happens:

```
$ trouble
/home/me/bin/trouble: line 10: unexpected EOF while looking for matching `"'
/home/me/bin/trouble: line 13: syntax error: unexpected end of file
```

It generates two errors.
Interestingly, the line numbers reported by the error messages are not where the missing quote was removed but rather much later in the program.
If we follow the program after the missing quote, we can see why.
`bash` will continue looking for the closing quote until it finds one, which it does, immediately after the second `echo` command.
After that, `bash` becomes very confused.
The syntax of the subsequent `if` command is broken because the `fi` statement is now inside a quoted (but open) string.

In long scripts, this kinds of error can be quite hard to find.
Using an editor with syntax highlighting will help since, in most cases, it will display quoted strings in a distinctive manner from other kinds of shell syntax.
If a complete version of `vim` is installed, syntax highlighting can be enabled by entering this command:

`:syntax on`

### Missing or Unexpected Tokens

Another common mistake is forgetting to complete a compound command, such as `if` or `while`.
Let's look at what happens if we remove the semicolon after `test` in the `if` command:

```shell
# trouble: script to demonstrate common errors

number=1

if [ $number = 1 ] then
  echo "Number is equal to 1."
else
  echo "Number is not equal to 1."
fi
```

The result is this:

```
$ trouble
/home/me/bin/trouble: line 9: syntax error near unexpected token `else'
/home/me/bin/trouble: line 9: `else'
```

Again, the error message points to an error that occurs later than the actual problem.
What happens is really pretty interesting.
As we recall, `if` accepts a list of commands and evaluates the exit code of the last command in the list.
In our program, we intend this list to consist of a single command , `[`, a synonym for `test`.
The `[` command takes what follows it as a list of arguments;
in our case, that's four arguments: `$number`, 1, =, and `]`.
With the semicolon removed, the word `then` is added to the list of arguments, which is syntactically legal.
The following `echo` command is legal, too.
It's interepreted as another command in the list of commands that `if` will evaluate for an exit code.
The `else` is encountered next, but it's out of place since the shell recognizes it as a *reserved word* (a word that has special meaning to the shell) and not the name of a command, which is the reason for the error message.

### Unanticipated Expansions

It's possible to have errors that occur only intermittently in a script.
Sometimes the script will run fine, and other times it will fail because of the results of an expansion.
If we return our missing semicolon and change the value of `number` to an empty variable, we can demonstrate.

```shell
#!/bin/bash

# trouble: script to demonstrate common errors

number=

if [ $number = 1 ]; then
  echo "Number is equal to 1."
else
  echo "Number is not equal to 1."
fi
```

Running this script with this change results in the following output:

```
$ trouble
/home/me/bin/trouble: line 7 [: =: unary operator expected
Number is not equal to 1.
```

We get this rather cryptic error message, followed by the output of the second `echo` command.
The problem is the expansion of the `number` variable within the `test` command.
When the following command:

`[ $number = 1 ]`

undergoes expansion with `number` being empty, the result is this:

`[ = 1 ]`

which is invalid and the error is generated.
The = operator is a binary operator (it requires a value on each side), but the first value is missing, so the `test` command expects a unary operator (such as -z) instead.
Further, since the `test` failed (because of the error), the `if` command receives a non-zero exit code and acts accordingly, and the second `echo` command is executed.

This problem can be corrected by adding quotes around the first argument in the `test` command.

`[ "$number" = 1 ]`

Then when expansion occurs, the result will be this:

`[ "" = 1 ]`

This yields the correct number of arguments.
In addition to empty strings, quotes should be used in cases where a value could expand into multiword strings, as with filenames containing embedded spaces.

> Make it a rule to always enclose variables and command substitutions in double quotes unless word splitting is needed.

### Logical Errors

Unlike syntactic errors, *logical errors* do not prevent a script from running.
The script will run, but it will not produce the desired result because of a problem with its logic.
There are countless numbers of possible logical errors, but here are a few of the most common kinds found in scripots:

+ **Incorrect conditional expressions.** It's easy to incorrectly code an `if/then/else` epxression and have the wrong logic carried out.
Sometimes the logic will be reversed, or it will be incomplete.

+ **"Off by one" errors.** When coding loops that employ counters, it is possible to overlook that the loop may require that the counting start with zero, rather than one, for the count to conclude at the correct point.
These kinds of errors result in either a loop "going off the end" by counting too far or a loop missing the last iteration by terminating one iteration too soon.

+ **Unanticipated situations.** Most logic errors result from a program encountering data or situations that were unforeseen by the programmer.
As we have seen, this can also include unanticipated expansions, such as a filename that contains embedded spaces that expands into multiple command arguments rather than a single filename.

### Defensive Programming

It is important to verify assumptions when programming.
This means a careful evaluation of the exit status of programs and commands that are used by a script.
Here is an example, based on a true story.
An unfortunate system administrator wrote a script to perform a maintenance task on an important server.
The script contained the following two lines of code:

```
cd $dir_name
rm *
```

There is nothing intrinsically wrong with these two lines, as long as the directory named in the variable, `dir_name`, exists.
But what happens if it does not?
In that case, the `cd` command fails, and script continues to the next line and deletes the files in the current working directory.
Not the desired outcome at all!
The hapless administrator destroyed an important part of the server because of this design decision.

Let's look at some ways this design could be improved.
First, it might be wise to ensure that the `dir_name` variable expands into only one word by quoting it and make the execution of `rm` contingent on the success of `cd`.

`cd "$dir_name" && rm *`

This way, if the `cd` command fails, the `rm` command is not carried out.
This is better but still leaves open the possibility that the variable, `dir_name`, is unset or empty, which would result in the files in the user's home directory being deleted.
This could be avoided by checking to see that `dir_name` actually contains the name of an existing directory.

`[[ -d "$dirname" ]] && cd "$dir_name" && rm *`

Often, it is best to include logic to terminate the script and report an error when a situation such as the one shown previously occurs.

```shell
# Delete files in directory $dir_name
if [[ ! -d "$dir_name" ]]; then
  echo "No such directory: '$dir_name'" >&2
  exit 1
fi
if ! cd "$dir_name"; then
  echo "Cannot cd to '$dir_name'" >&2
  exit 1
fi
if ! rm *; then
  echo "File deletion failed. Check results" >&2
  exit 1
fi
```

Here, we check both the name, to see that it is an existing directory, and the success of the `cd` command.
If either fails, a descriptive error message is sent to standard error, and the script terminates with an exit status of one to indicate a failure.

### Watch Out for Filenames

There is another problem with this file deletion script that is more obscure but could be very dangerous.
Unix (and Unix-like operating systems) has, in the opinion of many, a serious design flaw when it comes to filenames.
Unix is extremely permissive about them.
In fact, there are only two characters that cannot be included in a filename.
The first is the / character since it is used to separate elements of a pathname, and the second is the null character (a zero byte), which is used internally to mark the ends of strings.
Everything else is legal including spaces, tabs, line feeds, leading hyphens, carriage returns, and so on.

Of particular concern are leading hyphens.
For example, it's perfectly legal to have a file named -rf~.
Consider for a moment what happens when that filename is passed to rm.

To defend against this problem, we want to change our `rm` command in the file deletion script from this:

`rm *`

to the following:

`rm ./*`

This will prevent a filename starting with a hyphen from being interpreted as a command option.
As a general rule, always precede wildcareds (such as * and ?) with ./ to prevent misinterpretation by commands.
This includes things like `*.pdf` and `???.mp3`, for example.

> There is a standard called the POSIX Portable Filename Character Set that can be used to maximize the chances that a filename will work across different systems.
The standard is pretty simple.
The only characters allowed are the uppercase letters A-Z, the lowercase letters a-z, the numerals 0-9, period (.), hyphen (-), and underscore (_).
The standard further suggests that filenames not begin with a hyphen.

### Verifying Input

A general rule of good programming is that if a program accepts input, it must be able to deal with anything it receives.
This usually means that input must be carefully screened to ensure that only valid input is accepted for further processing.
We saw an example of this in the previous chapter when we studied the `read` command.
One script contained the following test to verify a menu selection:

`[[ $REPLY =~ &[0-3]$ ]]`

This test is very specific.
It will return a zero exit status only if the string entered by the user is a numeral in the range of zero to three.
Nothing else will be accepted.
Sometimes these kinds of tests can be challenging to write, but the effort is necessary to produce a high-quality script.

### Testing

Testing is an important step in every kind of software development, including scripts.
There is a saying in the open source world, "release early, release often," that reflects this fact.
By releasing early and often, software gets more exposure to use and testing.
Experience has shown that bugs are much easier to find, and much less expensive to fix, if they are found early in the development cycle.

We saw how stubs can be used to verify program flow.
From the earliest stages of script development, they are a valuable technique to check the program of our work.

Let's look at the file-deletion problem shown previously and see how this could be coded for easy testing.
Testing the original fragment of code would be dangerous since its purpose is to delete files, but we could modify the code to make the test safe.

```shell
if [[ -d $dir_name ]]; then
  if cd $dir_name; then
    echo rm *   # TESTING
  else
    echo "cannot cd to '$dir_name'" >&2
    exit 1
  fi
else
  echo "no such directory: '$dir_name'" >&2
  exit 1
fi
exit  # TESTING
```

Because the error conditions already output useful messages, we don't have to add any.
The most important chang eis placing an `echo` command just before the `rm` command to allow the command and its expanded argument list to be displayed, rather than the command actually being executed.
This change allows safe execution of the code.
At the end of the code fragment, we place an `exit` command to conclude the test and prevent any other part of the script from being carried out.
The need of this will vary according to the design of the script.

We also include some comments that act as "markers" for our test-related changes.
These can be used to help find and remove the changes when testing is complete.

### Test Cases

To perform useful testing, it's important to develop and apply good *test cases*.
This is done by carefully choosing input data or operating conditions that reflect *edge* and *corner* cases.
In our code fragment (which is simple), we want to know how the code performs under three specific conditions.

+ `dir_name` contains the name of an existing directory.

+ `dir_name` contains the name of a nonexistent directory.

+ `dir_name` is empty.

By performing the test with each of these conditions, good *test coverage* is achieved.

Just as with design, testing is a function of time, as well.
Not every script feature needs to be extensively tested.
It's really a matter of determining what is most importnat.
Since it could be so potentially destructive if it malfunctioned, our code fragment deserves careful consideration during both its design and testing.

### Debugging

If testing reveals a problem with a script, the next step is debugging.
"A problem" usually means that the script is, in some way, not performing to the programmer's expectations.
If this is the case, we need to carefully determine exactly what the script is actually doing and why.
Finding bugs can sometimes involve a lot of detective work.

A well-designed script will try to help. It should be programmed defensively to detect abnormal conditions and provide useful feedback to the user.
Sometimes, however, problems are quite strange and unexpected, and more involved techniques are required.

### Finding the Problem Area

In some scr4ipts, particularly long ones, it is sometimes useful to isolate the area of the script that is related to the problem.
This won't always be the actual error, but isolation will often provide insights into the actual cause.
One technique that can be used to isolate code is "commenting out" sections of a cript.
For example, our file deletion fragment could be modified to determine whether the removed section was related to an error.

```shell
if [[ -d $dir_name ]]; then
  if cd $dir_name; then
    rm *
  else
    echo "cannot cd to '$dir_name'" >&2
    exit 1
  fi
# else
#   echo "no such directory: '$dir_name'" >&2
#   exit 1
fi
```

By placing comment symbols at the beginning of each line in a logical section of a script, we prevent that section from being executed.
Testing can then be performed again to see whether the removal of the code has any impact on the behavior of the bug.

### Tracing

Bus are often cases of unexpected logical flow within a script.
That is, portions of the script either are never being executed or are being executed in the wrong order or at the wrong time.
To view the actual flow of the program, we use a technique called *tracing*.

One tracing method involves placing informative messages in a script that display the location of execution.
We can add messages to our code fragment.


```shell
echo "preparing to delete files" >&2
if [[ -d $dir_name ]]; then
  if cd $dir_name; then
echo "deleting files" >&2
    rm *
  else
    echo "cannot cd to '$dir_name'" >&2
    exit 1
  fi
else
  echo "no such directory: '$dir_name'" >&2
  exit 1
fi
echo "file deletion complete" >&2
```

We send the messages to standard error to separate them from normal output.
We also do not indent the lines containing the messages, so it is easier to find when it's time to remove them.

Now when the script is executed, it's possible3 to see that the file deletion has been performed.

```
$ deletion-script
preparing to delete files
deleting files
file deletion complete
$
```

`bash` also provides a method of tracing, implemented by the `-x` option and the `set` command with the `-x` option.
Using our earlier `trouble` script, we can activate tracing for the entire script by adding the -x option to the first line.

```shell
#!/bin/bash -x

# trouble: script to demonstrate common errors

number=1

if [ $number = 1 ]; then
  echo "Number is equal to 1."
else
  echo "Number is not equal to 1."
fi
```

When executed, the results look like this:

```
$ trouble
+ number=1
+ '[' 1 = 1 ']'
+ echo 'Number is equal to 1.'
Number is equal to 1.
```

With tracing enabled, we see the commands performed with expansions applied.
The leading plus signs indicate the display of the trace to distinguish them from lines of regular output.
The plus sign is the default character for trace output.
It is contained in the PS4 (prompt string 4) shell variable.
The contents of this variable can be adjusted to make the prompt more useful.
Here, we modify the contents of the variable to include the current line number in the script where the trace is performed.
Note that single quotes are required to prevent expansion until the prompt is actually used.

```
$ export PS4='LINENO + '
$ trouble
5 + number=1
7 + '[' 1 = 1 ']'
8 + echo 'Number is equal to 1.'
Number is equal to 1.
```

To perform a trace on a selected portion of a script, rather than the entire script, we can use the set command with the `-x` option.

```shell
#!/bin/bash

# trouble: script to demonstrate common errors

number=1

set -x  # Turn on tracing
if [ $number = 1 ]; then
  echo "Number is equal to 1."
else
  echo "Number is not equal to 1."
fi
set +x  #Turn off tracing
```

We use the `set` command with the `-x` option to activate tracing and the `+x` option to deactivate tracing.
This technique can be used to examine multiple portions of a troublesome script.

### Examining Values During Execution

It is often useful, along with tracing, to display the content of variables to see the internal workings of a script while it is being executed.
Applying additional `echo` statements will usually do the trick.

```shell
#!/bin/bash

# trouble: script to demonstrate common errors

number=1

echo "number=$number" # DEBUG
set -x  # Turn on tracing
if [ $number = 1 ]; then
  echo "Number is equal to 1."
else
  echo "Number is not equal to 1."
fi
set +x  # Turn off tracing
```

In this trivial example, we simply display the value of the variable number and mark the added line with a comment to facilitate its later identification and removal.
This technique is particularly useful when watching the behavior of loops and arithmetic within scripts.

---

## Flow Control: Branching with Case

```bash
#! /bin/bash

read -p "Are you 21 or over? Y/N" ANSWER

case "$ANSWER" in
  [yY] | [yY][eE][sS])
    echo "You can have a beer"
    ;;
  [nN] | [nN][oO])
    echo "Sorry, no drinking"
    ;;
  *)
    echo "Please enter Y/N"
    ;;
esac
```

### The case Command

In `bash`, the multiple-choice compound command is called `case`.
It has the following syntax.

```shell
case word in
  [pattern [| pattern]...) commands ;;]...
esac
```
If we look at the `read-menu` program from previous chapters, we see the logic used to act on a user's selection.

```shell
#!/bin/bash

# read-menu: a menu driven system information program

clear
echo "
Please Select:

1. Display System Information
2. Display Disk Space
3. Display Home Space Utilization
0. Quit
"
read -p "Enter selection [0-3] > "

if [[ "$REPLY" =~ ^[0-3]$ ]]; then
  if [[ "$REPLY" == 0 ]]; then
    echo "Program terminated."
    exit
  fi
  if [[ "$REPLY" == 1 ]]; then
    echo "Hostname: $HOSTNAME"
    uptime
    exit
  fi
  if [[ "$REPLY" == 2 ]]; then
    df -h
    exit
  fi
  if [[ "$REPLY" == 3 ]]; then
    if [[ "$(id -u)" -eq 0 ]]; then
      echo "Home Space Utilization (All Users)"
      du -sh /home/*
    else
      echo "Home Space Utiliazation ($USER)"
      du -sh "$HOME"
    fi
    exit
  fi
else
  echo "Invalid entry." >&2
  exit 1
fi
```

Using `case`, we can replace this logic with something simpler.

```shell
#!/bin/bash

# case-menu: a menu driven system information program

clear
echo "
Please Select:

1. Display System Information
2. Display Disk Space
3. Display Home Space Utilization
0. Quit
"

read -p "Enter selection [0-3] > "

case "$REPLY" in
  0)  echo "Program terminated."
      exit
      ;;
  1)  echo "Hostname: $HOSTNAME"
      uptime
      ;;
  2)  df -h
      ;;
  3)  if [[ "$(id -u)" -eq 0 ]]; then
        echo "Home Space Utilization (All Users)"
        du -sh /home/*
      else
        echo "Home Space Utilization ($USER)"
        du -sh "$HOME"
      fi
      ;;
  *)  echo "Invalid entry." >&2
      exit 1
      ;;
esac
```

The `case` command looks at the value of `word`, which in our example is the value of the `REPLY` variable, and then attempts to match it against one of the specified patterns.
When a match is found, the commands associated with the specified pattern are executed.
After a match is found, no further matches are attempted.

### Patterns

The patterns used by `case` are the same as those used by pathname expansion.
Patterns are terminated with a `)` character.

Pattern | Description
---|---
a) | Matches if `word` equals a.
[[:alpha:]] | Matches if `word` is a single alphabetic character.
???) | Matches if `word` is exactly three characters long. 
*.txt | Matches if `word` ends with characters `.txt`
*) | Matches any value of `word`. It is good practice to include this as the last pattern in a `case` command to catch any values of `word` that did not match a previous pattern, that is, to catch any possible invalid values.

Here is an example of patterns at work:

```shell
#!/bin/bash

read -p "enter word > "

case "$REPLY" in
  [[:alpha:]])  echo "is a single alphabetic character." ;;
  [ABC][0-9])   echo "is A, B, or C followed by a digit." ;;
  ???)          echo "is three characters long." ;;
  *.txt)        echo "is a word ending with '.txt'" ;;
  *)            echo "is something else." ;;
esac
```

It is also possible to combine multiple patterns using the vertical bar character as a separator.
This creates an "or" conditional pattern.
This is useful for such things as handling both uppercase and lowercase characters.
Here's an example:

```shell
#!/bin/bash

# case-menu: a menu driven system information program

clear
echo "
Please Select:

A. Display System Information
B. Display Disk Space
C. Display Home Space Utilization
Q. Quit
"

read -p "Enter selection [A, B, C or Q] > "

case "$REPLY" in
  q|Q)    echo "Program terminated."
          exit
          ;;
  a|A)    echo "Hostname: $HOSTNAME"
          uptime
          ;;
  b|B)    df -h
          ;;
  c|C)    if [[ "$(id -u)" -eq 0 ]]; then
            echo "Home Space Utilization (All Users)"
            du -sh /home/*
          else
            echo "Home Space Utilization ($USER)"
            du -sh "$HOME"
          fi
          ;;
  *)      echo "Invalid entry." >&2
          exit 1
          ;;
esac
```

Here, we modify the `case-menu` program to use letters instead of digits for menu selection.
Notice how the new patterns allow for entry of both uppercase and lowercase letters.

### Performing Multiple Actions

In versions of `bash` prior to 4.0, `case` allowed only one action to be performed on a successful match.
After a successful match, the command would terminate.
Here we see a script that tests a character:

```shell
#!/bin/bash

# case4-1: test a character

read -n 1 -p "Type a character > "
echo
case "$REPLY" in
  [[:upper:]])    echo "'$REPLY' is uppercase." ;;
  [[:lower:]])    echo "'$REPLY' is lowercase." ;;
  [[:alpha:]])    echo "'$REPLY' is alphabetic." ;;
  [[:digit:]])    echo "'$REPLY' is a digit." ;;
  [[:graph:]])    echo "'$REPLY' is a visible character." ;;
  [[:punct:]])    echo "'$REPLY' is a punctuation symbol." ;;
  [[:space:]])    echo "'$REPLY' is a whitespace character." ;;
  [[:xdigit:]])   echo "'$REPLY' is a hexadecimal digit." ;;
esac
```

Running the script produces this:

```
$ case4-1
Type a character > a
'a' is lowercase.
```

The script works for the most part but fails if a character matches more than one of the POSIX character classes.
For example, the character `a` is both lowercase and alphabetic, as well as a hexadecimal digit.
In bash prior to version 4.0, there was no way for `case` to match more than one test.
Modern versions of bash add the `;;&` notation to terminate each action, so now we can do this:

```shell
#!/bin/bash

# case4-2: test a character

read -n 1 -p "Type a character > "
echo
case "$REPLY" in
  [[:upper:]])    echo "'$REPLY' is uppercase." ;;&
  [[:lower:]])    echo "'$REPLY' is lowercase." ;;&
  [[:alpha:]])    echo "'$REPLY' is alphabetic." ;;&
  [[:digit]])     echo "'$REPLY' is a digit." ;;&
  [[:graph:]])    echo "'$REPLY' is a visible character." ;;&
  [[:punct:]])    echo "'$REPLY' is a punctuation symbol." ;;&
  [[:space:]])    echo "'$REPLY' is a whitespace character." ;;&
  [[:xdigit:]])   echo "'$REPLY' is a hexadecimal digit." ;;&
esac
```

When we run this script, we get this:

```
$ case4-2
Type a character > a
'a' is lowercase.
'a' is alphabetic.
'a' is a visible character.
'a' is a hexadecimal digit.
```

The addition of the `;;&` syntax allows `case` to continue to the next test rather than simply terminating.

---

## Positional Parameters

One feature that has been missing from our programs so far is the ability to accept and process command line options and arguments.
In this chapter, we will examine the shell features that allow our programs to get access to the contents of the command line.

### Accessing the Command Line

The shell provides a set of variables called *positional parameters* that contain the individual words on the command line.
The variables are name 0 through 9.
They can be demonstrated this way:

```shell
#!/bin/bash

# posit-param: script to view command line parameters

echo "
\$0 = $0
\$1 = $1
\$2 = $2
\$3 = $3
\$4 = $4
\$5 = $5
\$6 = $6
\$7 = $7
\$8 = $8
\$9 = $9
"
```

This is a simple script that displays the values of the variables `$0-$9`.
When executed with no command line arguments, the result is this:

```
$ posit-param

$0 = /home/me/bin/posit-param
$1 =
$2 =
$3 =
$4 =
$5 =
$6 =
$7 =
$8 =
$9 =
```

Even when no arguments are provided, `$0` will always contain the first item appearing on the command line, which is the pathname of the program being executed.
When arguments are provided, we see these results:

```
$ posit-param a b c d

$0 = /home/me/bin/posit-param
$1 = a
$2 = b
$3 = c
$4 = d
$5 =
$6 =
$7 =
$8 =
$9 =
```

> You can actually access more than nine parameters using parameter expansion.
To specify a number greater than nine, surround the number in braces, as in `${10}`, `${55}`, `${221}`, and so on.

### Determining the Number of Arguments

The shell also provides a variable, `$#`, that contains the number of arguments on the command line.

```shell
#!/bin/bash

# posit-param: script to view command line parameters

echo "
Number of arguments: $#
\$0 = $0
\$1 = $1
\$2 = $2
\$3 = $3
\$4 = $4
\$5 = $5
\$6 = $6
\$7 = $7
\$8 = $8
\$9 = $9
"
```

This is the result:

```
$ posit-param a b c d

Number of arguments: 4
$0 = /home/me/bin/posit-param
$1 = a
$2 = b
$3 = c
$4 = d
$5 =
$6 =
$7 =
$8 =
$9 =
```

### shift - Getting Access to Many Arguments

But what happens when we give the program a large number of arguments such as the following?

```shell
$ posit-param *

Number of arguments: 82
$0 = /home/me/bin/posit-param
$1 = addresses.ldif
$2 = bin
$3 = bookmarks.html
$4 = debian-500-i386-netinst.iso
$5 = debian-500-i386-netinst.jigdo
$6 = debian-500-i386-netinst.template
$7 = debian-cd_info.tar.gz
$8 = Desktop
$9 = dirlist-bin.txt
```

On this example system, the wildcard `*` expands into 82 arguments.
How can we process that many?
The shell provides a method, albeit a clumsy one, to do this.
The `shift` command causes all the parameters to "move down one" each time it is executed.
In fact, by using `shift`, it is possible to get by with only one parameter (in addition to `$0`, which never changes).

```shell
#!/bin/bash

# posit-param2: script to display all arguments

count=1

while [[ $# -gt 0 ]]; do
  echo "Argument $count = $1"
  count=$((count + 1))
  shift
done
```

Each time `shift` is executed, the value of `$2` is moved to `$1`, the value of `$3` is moved to `$2`, and so on.
The value of `$#` is also reduced by one.

In the `posit-param2` program, we create a loop that evaluates the number of arguments remaining and continues as long as there is at least one.
We display the current argument, increment the variable `count` with each iteration of the loop to provide a running count of the number of arguments processed, and, finally, execute a `shift` to load `$1` with the next argument.
Here is the program at work:

```
$ posit-param2 a b c d
Argument 1 = a
Argument 2 = b
Argument 3 = c
Argument 4 = d
```

### Simple Applications

Even without `shift`, it's possible to write useful applications using positional parameters.
By way of example, here is a simple file information program:

```shell
#!/bin/bash

# file-info: simple file information program

PROGNAME="$(basename "$0")"

if [[ -e "$1" ]]; then
  echo -e "\nFile Type:"
  file "$1"
  echo -e "\nFile Status:"
  stat "$1"
else
  echo "$PROGNAME: usage: $PROGNAME file" >&2
  exit 1
fi
```

This program displays the file type (determined by the `file` command) and the file status (from the `stat` command) of a specified file.
One interesting feature of this program is the `PROGNAME` variable.
It is given the value that results from the `basename "$0"` command.
The `basename` command removes the leading portion of a pathname, leaving only the base name of a file.
In our example, `basename` removes the leading portion of the pathname contained in the `$0` parameter, the full pathname of our example program.
  ### Using Positional Parameters with Shell Functions




Just as positional parameters are used to pass arguments to shell scripts, they can also be used to pass arguments to shell functions.
To demonstrate, we will convert the `file_info` script into a shell function.

```shell
file_info () {

  # file_info: function to display file information

  if [[ -e "$1" ]]; then
    echo -e "\nFile Type:"
    file "$1"
    echo -e "\nFile Status:"
    stat "$1"
  else
    echo "$FUNCNAME: usage: $FUNCNAME file" >&2
    return 1
  fi
}
```

Now, if a script that incorporates the `file_info` shell function calls the function with a filename argument, the argument will be passed to the function.

With this capability, we can write many useful shell functions that not only can be used in scripts but also can be used within our `.bashrc` files.

Notice that the `PROGNAME` variable was changed to the shell variable `FUNCNAME`.
The shell automatically updates this variable to keep track of the currently executed shell function.
Note that `$0` always contains the full pathname of the first item on the command line (i.e., the name of the program) and does not contain the name of the shell function as we might expect.

### Handling Positional Parameters en Masse

It is sometimes useful to manage all the positional parameters as a group.
For example, we might want to write a "wrapper" around another program.
This means we create a script or shell function that simplifies the invocation of another program.
The wrapper, in this case, supplies a list of arcane command line options and then passes a list of arguments to the lower-level program.

The shell provides two special parameters for this purpose.
They both expand into the complete list of positional parameters but differ in rather subtle ways.

Parameter | Description
---|---
$* | Expands into the list of positional parameters, starting with 1. When surrounded by double quotes, it expands into a double-quoted string containing all of the positional parameters, each separated by the first character of the IFS shell variable (by default a space character).
$@ | Expands into the list of positional parameters, starting with 1. When surrounded by double quotes, it expands each positional parameter into a separate word as if it was surrounded by double quotes.

Here is a script that shows these special parameters in action:

```shell
#!/bin/bash

# posit-params3: script to demonstrate $* and $@

print_params () {
  echo "\$1 = $1"
  echo "\$2 = $2"
  echo "\$3 = $3"
  echo "\$4 = $4"
}

pass_params () {
  echo -e "\n" '$* :'; print_params $*
  echo -e "\n" '"$*" :'; print_params "$*"
  echo -e "\n" '$@ :'; print_params $@
  echo -e "\n" '"$@" :'; print_params "$@"
}

pass_params "word" "words with spaces"
```

In this rather convoluted program, we create two arguments, called `word` and `words with spaces`,and pass them to the `pass_params` function.
That function, in turn, passes them on to the `print_params` function, using each of the four methods available with the special parameters `$*` and `$@`.
When executed, the script reveals the differences.

```
$ posit_param3

$* :
$1 = word
$2 = words
$3 = with
$4 = spaces

"$*" :
$1 = word words with spaces
$2 =
$3 =
$4 =

$@ :
$1 = word
$2 = words
$3 = with
$4 = spaces

"$@" :
$1 = word
$2 = words with spaces
$3 =
$4 =
```


With our arguments, both `$*` and `$@` produce a four-word result.

`word words with spaces`

`"$*"` produces a one-word result.

`"word words with spaces"`

`"$@"` produces a two-word result.

`"word" "words with spaces"`

This matches our actual intent.
The lesson to take from this is that even though the shell provides four different ways of getting the list of positional parameters, `"$@"` is by far the most useful for most situations because it preserves the integrity of each positional parameter.
To ensure safety, it should always be used, unless we have a compelling reason not to use it.

### A More Complete Application

After a long hiatus, we are going to resume work on our `sys_info_page` program.
Our next addition will add several command line options to the program as follows:

+ **Output file** - We will add an option to specify a name for a file to contain the program's output.
It will be specified as either `-f file` or `--file file`.

+ **Interactive mode** - This option will prompt the user for an output file-name and will determine whether the specified file already exists.
If it does, the user will be prompted before the existing file is overwritten.
This option will be specified by either `-i` or `--interactive`.

+ **Help** - Either `-h` or `--help` may be specified to cause the program to output an informative usage message.

Here is the code needed to implement the command line processing:

```shell
usage () {
  echo "$PROGNAME: usage: $PROGNAME [-f file | -i]"
  return
}

# process command line options

interactive=
filename=

while [[ -n "$1" ]]; do
  case "$1" in
    -f | --file)          shift
                          filename="$1"
                          ;;
    -i | --interactive)   interactive=1
                          ;;
    -h | --help)          usage
                          exit
                          ;;
    *)                    usage >&2
                          exit 1
                          ;;
  esac
  shift
done
```

First, we add a shell function called `usage` to display a message when the help option is invoked or an unknown option is attempted.

Next, we begin the processing loop.
This loop continues while the positional parameter `$1` is not empty.
At the end of the loop, we have a `shift` command to advance the positional parameters to ensure that the loop will eventually terminate.

Within the loop, we have a `case` statement that examines the current positional parameter to see whether it matches any of the supported choices.
If a supported parameter is found, it is acted upon.
If an unknown choice is found, the usage message is displayed, and the script terminates with an error.

The `-f` parameter is handled in an interesting way.
When detected, it causes an additional `shift` to occur, which advances the positional parameter `$1` to the filename argument supplied to the `-f` option.

We next add the code to implement the interactive mode.

```shell
# interactive mode

if [[ -n "$interactive" ]]; then
  while true; do
    read -p "Enter name of output file: " filename
    if [[ -e "$filename" ]]; then
      read -p "'$filename' exists. Overwrite? [y/n/q] > "
      case "$REPLY" in
        Y|y)    break
                ;;
        Q|q)    echo "Program terminated."
                exit
                ;;
        *)      continue
                ;;
      esac
    elif [[ -z "$filename" ]]; then
      continue
    else
      break
    fi
  done
fi
```

If the `interactive` variable is not empty, an endless loop is started, which contains the filename prompt and subsequent existing file-handling code.
If the desired output file already exists, the user is prompted to overwrite, choose another filename, or quit the program.
If the user chooses to overwrite an existing file, a `break` is executed to terminate the loop.
Notice how the `case` statement detects only whether the user chooses to overwrite or quit.
Any other choice causes the loop to continue and prompts the user again.

To implement the output filename feature, we must first convert the existing page-writing code into a shell function, for reasons that will become clear in a moment


```shell
write_html_page () {
  cat <<- _EOF_
  <html>
    <head>
      <title>$TITLE</title>
    </head>
    <body>
      <h1>$TITLE</h1>
      <p>$TIMESTAMP</p>
      $(report_uptime)
      $(report_disk_space)
      $(report_home_space)
    </body>
  </html>
_EOF_
  return
}

# output html page

if [[ -n "$filename" ]]; then
  if touch "$filename" && [[ -f "$filename" ]]; then
    write_html_page > "$filename"
  else
    echo "$PROGNAME: Cannot write file '$filename'" >&2
    exit 1
  fi
else
  write_html_page
fi
```

The code that handles the logic of the `-f` option appears at the end of the previous listing.
In it, we test for the existence of a filename, and if one is found, a test is performed to see whether the file is indeed writable.
To do this, a `touch` is performed, followed by a test to determine whether the resulting file is a regular file.
These two tests take care of situations where an invalid pathname is input (`touch` will fail), and, if the file already exists, that it's a regular file.

As we can see, the `write_html_page` function is called to perform the actual generation of the page.
Its output is either directed to standard output (if the variable `filename` is empty) or redirected to the specified file.
Since we have two possible destinations for the HTML code, it makes sense to convert the `write_html_page` routine to a shell function to avoid redundant code.

### Summary

With the addition of positional parameters, we can now write fairly functional scripts.
For simple, repetitive tasks, positional parameters make it possible to write very useful shell functions that can be placed in a user's `.bashrc` file.

Our `sys_info_page` program has grown in complexity and sophistication.
Here is a complete listing, along with the most recent changes:

```shell
#!/bin/bash

# sys_info_page: program to output a system information page

PROGNAME="$(basename "$0")"
TITLE="System Information Report For $HOSTNAME"
CURRENT_TIME="$(date +"%x %r %Z")"
TIMESTAMP="Generated $CURRENT_TIME, by $USER"

report_uptime () {
  cat <<- _EOF_
          <h2>System Uptime</h2>
          <pre>$(uptime)</pre>
_EOF_
  return
}

report_disk_space () {
  cat <<- _EOF_
          <h2>Disk Space Utilization</h2>
          <pre>$(df -h)</pre>
_EOF_
  return
}

report_home_space () {
  if [[ "$(id -u)" -eq 0 ]]; then
    cat <<- _EOF_
            <h2>Home Space Utilization (All Users)</h2>
            <pre>$(du -sh /home/*)</pre>
_EOF_
  else
    cat <<- _EOF_
            <h2>Home Space Utilization ($USER)</h2>
            <pre>$(du -sh "$HOME")</pre>
_EOF_
  fi
  return
}

usage () {
  echo "$PROGNAME: usage: $PROGNAME [-f file | -i]"
  return
}

write_html_page () {
  cat <<- _EOF_
          <html>
            <head>
              <title>$TITLE</title>
            </head>
            <body>
              <h1>$TITLE</h1>
              <p>$TIMESTAMP</p>
              $(report_uptime)
              $(report_disk_space)
              $(report_home_space)
            </body>
          </html>
_EOF_
  return
}

# process command line options

interactive=
filename=

while [[ -n "$1" ]]; do
  case "$1" in
    -f | --file)        shift
                        filename="$1"
                        ;;
    -i | --interactive) interactive=1
                        ;;
    -h | --help)        usage
                        exit
                        ;;
    *)                  usage >&2
                        exit 1
                        ;;
  esac
  shift
done

# interactive mode

if [[ -n "$interactive"]]; then
  while true; do
    read -p "Enter name of the output file: " filename
    if [[ -e "$filename" ]]; then
      read -p "'$filename' already exists. Overwrite? [y/n/q] > "
      case "$REPLY" in
        Y|y)    break
                ;;
        Q|q)    echo "Program terminated."
                exit
                ;;
        *)      continue
                ;;
      esac
    elif [[ -z "$filename" ]]; then
      continue
    else
      break
    fi
  done
fi

# output html page

if [[ -n "$filename" ]]; then
  if touch "$filename" && [[ -f "$filename" ]]; then
    write_html_page > "$filename"
  else
    echo "$PROGNAME: Cannot write file '$filename'" >&2
    exit 1
  fi
else
  write_html_page
fi
```

---

## Flow Control: Looping with For

```bash
#! /bin/bash

NAMES="Jimmy Jammy Jommy Jemmy Jummy"

for NAME in $NAMES
do
  echo "Hello $NAME"
done
```

The *for loop* differs from the `while` and `until` loops in that it provides a means of processing sequences during a loop.
This turns out to be very useful when programming.
Accordingly, the `for` loop is a popular construct in `bash` scripting.

A `for` loop is implemented, naturally enough, with the `for` compound command.
In `bash`, `for` is available in two forms.

### for: Traditional Shell Form

The original `for` command's syntax is as follows:

```shell
for variable [in words]; do
  commands
done
```

where `variable` is the name of a variable that will increment during the execution of the loop, `words` is an optional list of items that will be sequentially assigned to `variable`, and `commands` are the commands that are to be executed on each iteration of the loop.

The `for` command is useful on the command line. We can easily demonstrate how it works.

```
$ for i in A B C D; do echo $i; done
A
B
C
D
```

In this example, `for` is given a list of four words: A, B, C, and D.
With a list of four words, the loop is executed four times.
Each time the loop is executed, a word is assigned to variable `i`.
Inside the loop, we have an `echo` command that displays the value of `i` to show the assignment.
As with the `while` and `until` loops, the `done` keyword closes the loop.

The really powerful feature of `for` is the number of intersting ways we can create the list of words.
For example, we can do it through brace expansion, like so:

```
$ for i in {A..D}; do echo $i; done
A
B
C
D
```

or we could use a pathname expansion, as follows:

```
$ for i in distros*.txt; do echo "$i"; done
distros-by-date.txt
distros-dates.txt
distros-key-names.txt
distros-key-vernums.txt
distros-names.txt
distros.txt
distros-vernums.txt
distros-versions.txt
```

Pathname expansion provides a nice, clean list of pathnames that can be processed in the loop.
The one precaution needed is to check that the expansion actually matched something.
By default, if the expansion fails to match any files, the widcards themselves (`distros*.txt` in the preceding example) will be returned.
To guard against this, we could code the preceding example in a script this way:

```shell
for i in distros*.txt; do
  if [[ -e "$i" ]]; then
    echo "$i"
  fi
done
```

By adding a test for file existence, we will ignore a failed expansion.
Another common method of word production is command substitution.

```shell
#!/bin/bash

# longest-word: find longest string in a file

while [[ -n "$1" ]]; do
  if [[ -r "$1" ]]; then
    max_word=
    max_len=0
    for i in $(strings "$1"); do
      len="$(echo -n "$i" | wc -c)"
      if (( len > max_len )); then
        max_len="$len"
        max_word="$i"
      fi
    done
    echo "$1: '$max_word' ($max_len characters)"
  fi
  shift
done
```

In this example, we look for the longest string found within a file.
When given one or more filenames on the command line, this program uses the `strings` program (which is included in the GNU binutils package) to generate a list of readable text "words" in each file.
The `for` loop processes each word in turn and determines whether the current word is the longest found so far.
When the loop concludes, the longest word is displayed.

One thing to note here is that, contrary to our usual practice, we do not surround the command substitution `$(sintrs "$1")` with double quotes.
This is because we actually want word splitting to occur to give us our list.
If we had surrounded the command substitution with quotes, it would produce only a single word containing every string in  the file.
That's not exactly what we are looking for.

If the optional `in words` portion of the `for` command is omitted, `for` defaults to processing the positional parameters.
We will modify our `longest-word` script to use this method:

```shell
#!/bin/bash

# longest-word2: find longest string in a file

for i; do
  if [[ -r "$i" ]]; then
    max_word=
    max_len=0
    for j in $(strings "$i"); do
      len="$(echo -n "$j" | wc -c)"
      if (( len > max_len )); then
        max_len="$len"
        max_word="$j"
      fi
    done
    echo "$i: '$max_word' ($max_len characters)"
  fi
done
```

As we can see, we have changed the outermost loop to use `for` in place of `while`.
By omitting the list of words in the `for` command, the positional parameters are used instead.
Inside the loop, previous instances of the variable `i` have been changed to the variable `j`.
The use of `shift` has also been eliminated.

> The variable used with `for` can be any valid variable, but `i` is the most common, followed by `j` and `k`.
> The basis of this tradition comes from the Fortran programming language.
> In Fortran, undeclared variables starting with the letters I, J, K, L and M are automatically typed as integers, while variables beginning with any other letter are typed as reals (numbers with decimal fractions).
> This behavior led programmers to use the variables I, J, and K for loop variables since it was less work to use them when a temporary variable (as loop variables often are) was needed.
> It also led to the following Fortran-based witticism: "GOD is real, unless declared integer."

### for: C Language Form

Recent versions of `bash` have added a second form of the `for` command syntax, one that resembles the form found in the C programming language.
Many other languages support this form, as well.

```shell
for (( expression1; expression2; expression3 )); do
  commands
done
```

Here, `expression1`, `expression2`, and `expression3` are arithmetic expressions, and `commands` are the commands to be performed during each iteration of the loop.

In terms of behavior, this form is equivalent to the following construct.

```shell
(( expression1 ))
while (( expression2 )); do
  commands
  (( expression3 ))
done
```

`expression1` is used to initialize conditions for the loop, `expression2` is used to determine when the loop is finished, and `expression3` is carried out at the end of each iteration of the loop.

Here is a typical application:

```shell
#!/bin/bash

# simpler_counter: demo of C style for command

for (( i=0; i<5; i=i+1 )); do
  echo $i
done
```

When executed, it produces the following output:

```
$ simpler_counter
0
1
2
3
4
```

In this example, `expression1` initializes the variable `i` with the value of zero, `expression2` allows the loop to continue as long as the value of `i` remains less than 5, and `expression3` increments the value of `i` by 1 each time the loop repeats.

The C language form of `for` is useful anytime a numeric sequence is needed.

### Summary

With our knowledge of the `for` command, we will now apply the final improvements to our `sys_info_page` script.
Currently, the `report_home_space` function looks like this:

```shell
report_home_space () {
  if [[ "$(id -u)" -eq 0 ]]; then
    cat <<- _EOF_
            <h2>Home Space Utilization (All Users)</h2>
            <pre>$(du -sh /home)</pre>
_EOF_
  else
    cat <<- _EOF_
            <h2>Home Space Utilization ($USER)</h2>
            <pre>$(du -sh "$HOME")</pre>
_EOF_
  fi
  return
}
```

Next, we will rewrite it to provide more detail for each user's home directory and include the total number of files and subdirectories in each.

```shell
report_home_space () {
  local format="%8s%10s%10s\n"
  local i dir_list total_files total_dirs total_size user_name

  if [[ "$(id -u)" -eq 0 ]]; then
    dir_list=/home/*
    user_name"All Users"
  else
    dir_list="$HOME"
    user_name="$USER"
  fi

  echo "<h2>Home Space Utilization ($user_name)</h2>"

  for i in $dir_list; do
    total_files="$(find "$i" -type f | wc -l)"
    total_dirs="$(find "$i" -type d | wc -l)"
    total_size=="$(du -sh "$i" | cut -f 1)"

    echo "<h3>$i</h3>"
    echo "<pre>
    printf "$format" "Dirs" "Files" "Size"
    printf "$format" "----" "-----" "----"
    printf "$format" "$total_dirs" "$total_files" "$total_size"
    echo "</pre>
  done
  return
}
```

This rewrite applies much of what we have learned so far.
We still test for the superuser, but instead of performing the complete set of actions as part of the `if`, we set some variables used later in a `for` loop.
We have added several local variables to the function and made use of `printf` to format some of the output.

---

## Strings and Numbers

Computer programs are all about working with data.
In past chapters, we have focused on processing data at the file level.
However, many programming problems need to be solved using smaller units of data such as strings and numbers.

In this chapter, we will look at several shell features that are used to manipulate strings and numbers.
The shell provides a variety of parameter expansions that perform string operation.
In addition to arithmetic expansion, there is a well-known command line program called `bc`, which performs higher-level math.

### Parameter Expansion

We have already worked with some forms of parameter expansion, for example, shell variable.
The shell provides many more.

> It's always good practice to enclose parameter expansions in double quotes to prevent unwanted word splitting, unless there is a specific reason not to.
> This is especially true when dealing with filenames since they can often include embedded spaces and other assorted nastiness.

#### Basic Parameters

The simplest form of parameter expansion is reflected in the ordinary use of variables.
Here's an example:

`$a`

When expanded, this becomes whatever the variable `a` contains.
Simple parameters may also be surrounded by braces.

`${a}`

This has no effect on the expansion but is required if the variable is adjacent to other text, which may confuse the shell.
In this example, we attempt to create a filename by appending the string `_file` to the contents of variable `a`:

```
$ a="foo"
$ echo "$a_file"
```

If we perform this sequence of commands, the result will be nothing because the shell will try to expand a variable named `a_file` rather than `a`.
This problem can be solved by adding braces around the "real" variable name.

```
$ echo "${a}_file"
foo_file
```

We have also seen that positional parameters greater than nine can be accessed by surrounding the number in braces.
For example, to access the eleventh positional parameter, we can do this:

`${11}`

#### Expansions to Manage Empty Variables

Several parameter expansions are intended to deal with nonexistent and empty variables.
These expansions are handy for handling missing positional parameters and assigning default values to parameters.
Here is one such expansion

`${parameter:-word}`

If `parameter` is unset (i.e., does not exist) or is empty, this expansion results in the value of `word`.
If `parameter` is not empty, the expansion results in the value of `parameter`.

```
$ foo=
$ echo ${foo:-"substitute value if unset"}
substitute value if unset
$ echo $foo

$ foo=bar
$ echo ${foo:-"substitute value if unset"}
bar
$ echo $foo
bar
```

Here is another expansion, in which we use the equal sign instead of a dash

`${parameter:=word}`

If `parameter` is unset or empty, this expansion results in the value of `word`.
In addition, the value of `word` is assigned to `parameter`.
If `parameter` is not empty, the expansion results in the value of `parameter`.

```
$ foo=
$ echo ${foo:="default value if unset"}
default value if unset
$ echo $foo
default value if unset
$ foo=bar
$ echo ${foo:="default value if unset"}
bar
$ echo $foo
bar
```

> Positional and other special parameters cannot be assigned this way.

Here we use a question mark:

`${parameter:?word}`

If `parameter` is unset or empty, this expansion causes the script to exit with an error, and the contents of `word` are sent to standard error.
If `parameter` is not empty, the expansion results in the value of `parameter`.

```
$ foo=
$ echo ${foo:?"parameter is empty"}
bash: foo: parameter is empty
$ echo $?
1
$ foo=bar
$ echo ${foo:?"parameter is empty"}
bar
$ echo $?
0
```

Here we use a plus sign:

`${parameter:+word}`

If `parameter` is unset or empty, the expansion results in nothing.
If `parameter` is not empty, the value of `word` is substituted for `parameter`;
however, the value of `parameter` is not changed.

```
$ foo=
$ echo ${foo:+"substitue value if set"}

$ foo=bar
$ echo ${foo:+"substitue value if set"}
substitue value if set
```

#### Expansions that Return Variable Names

The shell has the ability to return the names of variables.
This is used in some rather exotic situations.

`${!prefix*}`
`${!prefix@}`

This expansion returns the names of existing variables with names beginning with `prefix`.
According to the `bash` documentation, both forms of the expansion perform identically.
Here, we list all the variables in the environment4 with names that begin with `BASH`:

```
$  echo ${!BASH*}
BASH BASH_ARGC BASH_ARGV BASH_COMMAND BASH_COMPLETION BASH_COMPLETION_DIR
BASH_LINENO BASH_SOURCE BASH_SUBSHELL BASH_VERSINFO BASH_VERSION
```

#### String Operations

There is a large set of expansions that can be used to operate on strings.
Many of these expansions are particularly well suited for operations on pathnames.
The following expansion:

`${#parameter}`

expands into the length of the string contained by `parameter`.
Normally, `parameter` is a string;
however, if `parameter` is either `@` or `*`, then the expansion results in the number of positional parameters.

```
$ foo="This string is long."
$ echo "'$foo' is ${#foo} characters long."
'This string is long.' is 20 characters long.
```

The following expansions are used to extract a portion of the string contained in `parameter`:

`${parameter:offset}`
`${parameter:offset:length}`

The extraction begins at `offset` characters from the beginning of the string and continues until the end of the string, unless `length` is specified.

```
$ foo="This string is long."
$ echo ${foo:5}
string is long.
$ echo ${foo:5:6}
string
```

If the value of `offset` is negative, it is taken to mean it starts from the end of the string rather than the beginning.
Note that negative values must be preceded by a space to prevent confusion with the `${parameter:-word}` expansion.
`length`, if present, must not be less than zero.

If `parameter` is `@`, the result of the expansion is `length` positional parameters, starting at `offset`.

```
$ foo="This string is long."
$ echo ${foo: -5}
long.
$ echo ${foo: -5:2}
lo
```

The following expansions remove a leading portion of the string contained in `parameter` defined by `pattern`.

`${parameter#pattern}`
`${parameter##pattern}`

`pattern` is a wildcard pattern like those used in pathname expansion.
The difference in the two forms is that the `#` form removes the shortest match, while the `##` form removes the longest match.

```
$ foo=file.txt.zip
$ echo ${foo#*.}
txt.zip
$ echo ${foo##*.}
zip
```

The following are the same as the previous `#` and `##` expansions, except they remove text from the end of the string contained in `parameter` rather than from the beginning.

`${parameter%pattern}`
`${parameter%%pattern}`

Here is an example:

```
$ foo=file.txt.zip
$ echo ${foo%.*}
file.txt
$ echo ${foo%%.*}
file
```

The following expansions perform a search-and-replace operation upon the contents of `parameter`:

`${parameter/pattern/string}`
`${parameter//pattern/string}`
`${parameter/#pattern/string}`
`${parameter/%pattern/string}`

If text is found matching widcard `pattern`, it is replaced with the contents of `string`.
In the normal form, only the first occurrence of `pattern` is replaced.
In the `//` form, all occurrences are replaced.
The `/#` form requires that the match occur at the beginning of the string, and the `/%` form requires the match to occur at the end of the string.
In every form, `/string/ may be omitted, casuing the text matched by `pattern` to be deleted.

```
$ foo=JPG.JPG
$ echo ${foo/JPG/jpg}
jpg.JPG
$ echo ${foo//JPG/jpg}
jpg.jpg
$ echo ${foo/#JPG/jpg}
jpg.JPG
$ echo ${foo/%JPG/jpg}
JPG.jpg
```

Parameter expansion is a good thing to know.
The string manipulation expansions can be used as substitutes for other common commands such as `sed` and `cut`.
Expansions can improve the efficiency of scripts by eliminating the use of external programs.
As an example, we will modify the `longest-word` program discussed in the previous chapter to use the parameter expansion `${#j} in place of the command substitution `$(echo -n $j | wc -c) and its resulting subshell, like so:

```shell
#!/bin/bash

# longest-word3: find longest string in a file

for i; do
  if [[ -r "$i" ]]; then
    max_word=
    max_len=0
    for j in $(strings "$i"); do
      len="${#j}"
      if (( len > max_len )); then
        max_len="$len"
        max_word="$j"
      fi
    done
    echo "$i: '$max_word' ($max_len characters)"
  fi
done
```

Next, we will compare the efficiency of the two versions by using the `time` command.

```
$ time longest-word2 dirlist-usr-bin.txt
dirlist-usr-bin.txt: 'scrollkeeper-get-extended-content-list' (38 characters)

real      0m3.618s
user      0m1.544s
sys       0m1.768s
$ time longest-word3 dirlist-usr-bin.txt
dirlist-usr-bin.txt: 'scrollkeeper-get-extended-content-list' (38 characters)

real      0m0.060s
user      0m0.056s
sys       0m0.008s
```

The original version of the script takes 3.618 seconds to scan the text file, while the new version, using parameter expansion, takes only 0.06 seconds, which is a significant improvement.

#### Case Conversion

`bash` has four parameter expansions and two `declare` command options to support the uppercase/lowercase conversion of strings.

So, what is case conversion good for?
Aside from the obvious aesthetic value, it has an important role in programming.
Let's consider the case of a database lookup.
Imagine that a user has entered a string into a data input field that we want to look up in a database.
It's possible the user will enter the value in all uppercase letters or lowercase letters or a combination of both.
We certainly don't want to populate our database with every possible permutation of uppercase and lowercase spellings.
What to do?

A common approach to this problem is to *normalize* the user's input.
That is, convert it into a standardized form before we attempt the database lookup.
We can do this by converting all the characters in the user's input to either lower or uppercase and ensure that the database entries are normalized the same way.

The `declare` command can be used to normalize strings to either uppercase or lowercase.
Using `declare`, we can force a variable to always contain the desired format no matter what is assigned to it.

```shell
#!/bin/bash

# ul-declare: demonstrate case conversion via declare

declare -u upper
declare -l lower

if [[ $1 ]]; then
  upper="$1"
  lower="$1"
  echo "$upper"
  echo "$lower"
fi
```

In the preceding script, we use `declare` to create two variables, `upper` and `lower`.
We assign the value of the first command line argument (positional parameter 1) to each of the variables and then display them on screen.

```
$ ul-declare aBc
ABC
abc
```

As we can see, the command line argument (aBc) has been normalized.

In addition to `declare`, there are four parameter expansions that perform upper/lowercase conversion.

Format | Result
---|---
${parameter,,pattern} | Expand the value of `parameter` into all lowercase. `pattern` is an optional shell pattern that will limit which characters (for example, [A-F]) will be converted. See the `bash` man page for a full description of patterns.
${parameter,pattern} | Expand the value of `parameter`, changing only the first character to lowercase.
${parameter^^pattern} | Expand the value of `parameter` into all uppercase letters.
${parameter^pattern} | Expand the value of parameter, changing only the first character to uppercase (capitalization).

Here is a script that demonstrates these expansions:

```shell
#!/bin/bash

# ul-param: demonstrate case conversion via parameter expansion

if [[ "$1" ]]; then
  echo "${1,,}"
  echo "${1,}"
  echo "${1^^}"
  echo "${1^}"
fi
```

Here is the script in action:

```
$ ul-param aBc
abc
aBc
ABC
ABc
```

Again, we process the first command line argument and output the four variations supported by the parameter expansions.
While this script uses the first positional parameter, `parameter` may be any string, variable or string expression.

### Arithmetic Evaluation and Expansion

Arithmetic Expansion is used to perform various aithmetic operations on integers.
Its basic form is as follows:

`$((expression))`

where `expression` is avalid arithmetic expression.

This is related to the compound command `((  ))` used for arithmetic evaluation (truth tests).

In previous chapters, we saw some of the common types of expressions and operators.
Here, we will look at a more complete list.

#### Number Bases

In arithmetic expressions, the shell supports integer constants in any base.

Notation | Description
---|---
number | By default, numbers without any notation are treated as decimal (base 10) integers.
0number | In arithmetic expressions, numbers with a leading zero are considered octal
0xnumber | Hexadecimal notation.
base#number | `number` is in `base`

Here are some examples:

```
$ echo $((0xff))
255
$ echo $((2#11111111))
255
```

In the previous examples, we print the value of the hexadecimal number `ff` (the largest two-digit number) and the largest eight-digit binary (base 2) number.

#### Unary Operators

There are two unary operators, `+` and `-`, which are used to indicate whether a number is positive or negative, respectively. An example is `-5`.

#### Simple Arithmetic

Operator | Description
---|---
+ | Addition
- | Substitution
* | Multiplication
/ | Integer division
** | Exponentiation
% | Modulo (remainder)

Most of these are self-explanatory, but integer division and modulo require further discussion.

Since the shell's arithmetic operates only on integers, the results of division are always whole numbers.

```
$ echo $(( 5 / 2 ))
2
```

This makes the determination of a remainder in a division operation more important.

```
$ echo $(( 5 % 2 ))
1
```

By using the division and modulo operators, we can determine that 5 divided by 2 results in 2, with a remainder of 1.

Calculating the remainder is useful in loops.
It allows an operation to be performed at specified intervals during the loop's execution.
In the following example, we display a line of numbers, highlighting each multiple of 5:

```shell
#!/bin/bash

# modulo: demonstrate the modulo operator

for ((i = 0; i <= 20; i = i + 1)); do
  remainder=$((i % 5))
  if (( remainder == 0 )); then
    printf "<%d> " "$i"
  else
    printf "%d " "$i"
  fi
done
printf "\n"
```

When executed, the results look like this:

```
$ modulo
<0> 1 2 3 4 <5> 6 7 8 9 <10> 11 12 13 14 <15> 16 17 18 19 <20>
```

#### Assignment

Although its uses may not be immediately apparent, arithmetic expressions may perform assignment.
We have performed assignment many times, though in a different context.
Each time we give a variable a value, we are performing assignment.
We can also do it within arithmetic expressions.

```
$ foo=
$ echo $foo

$ if (( foo = 5 )); then echo "It is true."; fi
It is true.
$ echo $foo
5
```

In the preceding example, we first assign an empty value to the variable `foo` and verify that it is indeed empty.
Next, we perform an `if` with the compound command `(( foo = 5 ))`.
This process does two interesting things: it assigns the value of 5 to the variable `foo`, and it evaluates to true because `foo` was assigned a non-zero value.

> It is important to remember the exact meaning of = in the previous expression.
> A single = performs assignment.
> foo = 5 says "make foo equal to 5", while == evaluates equivalence.
> foo == 5 says "does foo equal to 5?"
> This is a common feature in many programming languages.
> In the shell, this can be a little confusing because the `test` command accepts a single = for string equivalence.
> This is yet another reason to use the more modern `[[  ]]` and `((  ))` compound commands in place of `test`.

In addition to the = notation, the shell also provides notations that perform some very useful assignments.

Notation | Description
---|---
parameter = value | Simple assignment. Assigns `value` to `parameter`.
parameter += value | Addition. Equivalent to `parameter = parameter + value`.
parameter -= value | Subtraction. Equivalent to `parameter = parameter - value`.
parameter *= value | Multiplication. Equivalent to `parameter = parameter * value`.
parameter /= value | Integer division. Equivalent to `parameter = parameter / value`.
parameter %= value | Modulo. Equivalent to `parameter = parameter % value`.
parameter++ | Variable post-increment. Equivalent to `parameter = parameter + 1`.
parameter-- | Variable post-decrement. Equivalent to `parameter = parameter - 1`.
++parameter | Variable pre-increment. Equivalent to `parameter = parameter + 1`.
--parameter | Variable pre-decrement. Equivalent to `parameter = parameter - 1`.

These assignment operators provide a convenient shorthand for many common arithmetic tasks.
Of special intrest are the increment (++) and decrement (--) operators, which increase or decrease the value of their parameters by one.
This style of notation is taken from the C programming language and has been incorporated into a number of other programming languages, including `bash`.

The operators may appear either at the front of a parameter or at the end.
While they both either increment or decrement the parameter by one, the two placements have subtle difference.
If placed at the front of the parameter, the parameter in incremented (or decremented) before the parameter is returned. If placed after, the operation is performed *after* the parameter is returned.
This is rather strange, but it is the intended behavior.
Here is a demonstration:

```
$ foo=1
$ echo $((foo++))
1
$ echo $foo
2
```

If we assign the value of one to the variable `foo` and then increment it with the `++` operator placed after the parameter name, `foo` is returned with the value of one.
However, if we look at the value of the variable a second time, we see the incremented value.
If we place the `++` operator in front of the parameter, we get the more expected behavior.

```
$ foo=1
$ echo $((++foo))
2
$ echo $foo
2
```

For most shell application, prefixing the operator will be the most useful.

The `++` and `--` operators are often used in conjuction with loops.
We will make some improvements to our modulo script to tighten it up a bit.

```shell
#!/bin/bash

# modulo2: demonstrate the modulo operator

for ((i = 0; i <= 20; ++i)); do
  if (( (i % 5) == 0 )); then
    printf "<%d> " "$i"
  else
    printf "%d " "$i"
  fi
done
printf "\n"
```

#### Bit Operations

One class of operator manipulate numbers in an unusual way.
These operators work at the bit level.
They are used for certain kinds of low-level tasks, often involving setting or reading bit flags.

Operator | Description
---|---
~ | Bitwise negation. Negate all the bits in a number.
`<<` | Left bitwise shift. Shift all the bits in a number to the left.
`>>` | Right bitwise shift. Shift all the bits in a number to the right.
& | Bitwise AND. Perform an AND operation on all the bits in two numbers.
`\|` | Bitwise OR. Perform an OR operation on all the bits in two numbers.
^ | Bitwise XOR. Perform an exclusive OR operation on all the bits in two numbers.

Note that there are also corresponding assignment operators (for example, `<<=`) for all but bitwise negation.

Here we will demonstrate producing a list of powers of 2, using the left bitwise shift operator:

```
$ for ((i=0;i<8;++i)); do echo $((1<<i)); done
1
2
4
8
16
32
64
128
```

#### Logic

The `((  ))` compound command supports a variety of comparison operators.
There are a few more that can be used to evaluate logic.

Operator | Description
---|---
`<=` | Less than or equal to.
`>=` | Greater than or equal to.
`<` | Less than.
`>` | Greater than.
== | Equal to.
!= | Not equal to.
&& | Logical AND.
`\|\|` | Logical OR.
expr1?expr2:expr3 | Comparison (ternary) operator. If expression `expr1` evaluates to be nonzero (arithmetic true), then `expr2`; else `expr3`.

When used for logical operations, expressions follow the rule of arithmetic logic;
that is, expressions that evaluate as zero are considered false, while non-zero expressions are considered true.
The `((  ))` compound command maps the results into the shell's normal exit codes.

```
$ if ((1)); then echo "true"; else echo "false"; fi
true
$ if ((0)); then echo "true"; else echo "false"; fi
false
```

The strangest of the logical operators is the ternary operator.
This operator (which is modeled after the one in the C programming language) performs a stand-alone logical test.
It can be used as a kind of `if/then/else` statement.
It acts on three arithmetic expressions (strings won't work), and if the first expression is true (or non-zero), the second expression is performed.
Otherwise, the third expression is performed.
We can try this on the command line:

```
$ a=0
$ ((a<1?++a:--a))
$ echo $a
1
$ ((a<1?++a:--a))
0
```

Here we see a ternary operator in action.
This example implements a toggle.
Each time the operator is performed, the value of the variable `a` switches from zero to one or vice versa.

Please note that performing assignment within the expressions is not straightforward.
When attempted, `bash` will declare an error.

```
$ a=0
$ ((a<1?a+=1:a-=1))
bash: ((: a<1?a+=1:a-=1: attempted assignment to non-variable (error token is "-=1")
```

This problem can be mitigated by surrounding the assignment expression with parentheses

```
$ ((a<1?(a+=1):(a-=1)))
```

Next is a more complete example of using arithmetic operators in a script that produces a simple table of numbers:

```shell
#!/bin/bash

# arith-loop: script to demonstrate arithmetic operators

finished=0
a=0
printf "a\ta**2\ta**3\n"
printf "=\t====\t====\n"

until ((finished)); do
  b=$((a**2))
  c=$((a**3))
  printf "%d\t%d\t%d\n" "$a" "$b" "$c"
  ((a<10?++a:(finished=1)))
done
```

In this script, we implement an `until` loop based on the value of the `finished` variable.
Initially, the variable is set to zero (arithmetic false), and we continue the loop until it becomes non-zero.
Within the loop, we calculate the square and cube of the counter variable `a`.
At the end of the loop, the value of the counter variable is evaluated.
If it is less than 10 (the maximum number of iterations), it si incremented by one, or else the variable `finished` is given the value of one, making `finished` arithmetically true, thereby terminating the loop.
Running the script gives this result:

```
$ arith-loop
a  a**2   a**3
=  ====   ====
0  0      0
1  1      1
2  4      8
3  9      27
4  16     64
5  25     125
6  36     216
7  49     343
8  64     512
9  81     729
10 100    1000
```

### bc - An Aribitrary Precision Calculator Language

We have seen how the shell can handle many types of integer arithmetic, but what if we need to perform higher math or even just use floating-point numbers?
The answer is, we can't.
At least not directly with the shell.
To do this, we need to use an external program.
There are several approaches we can take.
Embedding Perl or AWK programs is one possible solution.

Another approach is to use a specialized calculator program.
One such program found on many Linux systems is called `bc`.

The `bc` program reads a file written in its own C-like language and executes it.
A `bc` script may be a separate file, or it may be read from standard input.
The `bc` language supports quite a few features including variables, loops, and programmer-defined functions.
We won't cover `bc` entirely here, just enough to get a taste.
`bc` is well documented by its man page.

Let's start with a simple example.
We'll write a `bc` script to add 2 plus 2.

```bc
/* A very simple bc script */

2 + 2
```

The first line of the script is a comment.
`bc` uses the same syntax for comments as the C programming language.
Comments, which may span multiple lines, begin with `/*` and end with `*/`.

#### Using bc

If we save the previous script as `foo.bc`, we can run it this way:

```
$ bc foo.bc
bc 1.06.94
Copyright 1991-1994, 1997, 1998, 2000, 2004, 2006 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type `warranty'.
4
```

If we look carefully, we can see the result at the very bottom, after the copyright message.
This message can be suppressed with the `-q` (quiet) option.

`bc` can also be used interactively.

```
$ bc -q
2 + 2
4
quit
```

When using `bc` interactively, we simply type the calculations we want to perform, and the results are immediately displayed.
The `bc` command `quit` ends the interactive session.

It is also possible to pass a script to `bc` via standard input.

```
$ bc < foo.bc
4
```

The ability to take standard input means that we can use here documents, here strings, and pipes to pass scripts.
This is a here string example:

```
$ bc <<< "2+2"
4
```

#### An Example Script

As a real-world example, we will construct a script that performs a common calculation, monthly loan payments.
In the script that follows, we use a here document to pass a script to `bc`:

```shell
#!/bin/bash

# loan-calc: script to calculate monthly loan payments

PROGNAME="${0##/*}"   # Use parameter expansion to get basename

usage () {
  cat <<- EOF
  Usage: $PROGNAME PRINCIPLE INTEREST MONTHS

  Where:

  PRINCIPAL is the amount of the loan.
  INTEREST is the APR as a number (7% = 0.07)
  MONTHS is the length of the loan's term.

EOF
}

if (($# != 3)); then
  usage
  exit 1
fi

principal=$1
interest=$2
months=$3

bc <<- EOF
        scale = 10
        i = $interest / 12
        p = $principal
        n = $months
        a = p * ((i * ((1 + i) ^ n)) / (((1 + i) ^ n) - 1))
        print a, "\n"
EOF
```

When executed, the results look like this:

```
$ loan-calc 135000 0.0775 180
1270.7222490000
```

This example calculates the monthly payment for a $135,000 loan at 7.75 percent APR for 180 months (15 years).
Notice the precision of the answer.
This is determined by the value given to the special `scale` variable in the `bc` script.
A full description of the `bc` scripting language is provided by the `bc` man page.
While its mathematical notation is slightly different from that of the shell (`bc` more closely resembles C), most of it will be quite familiar, based on what we have learned so far.

---

## Arrays

In the previous chapter, we looked at how the shell can manipulate strings and numbers.
The data types we have looked at so far are known in computer science circles as *scalar variables*;
that is, they are variables that contain a single value.

In this chapter, we will look at another kind of data structure called an *array*, which holds multiple values.
Arrays are a feature of virtually every programming language.
The shell supports them, too, though in a rather limited fashion.
Even so, they can be very useful for solving some types of programming problems.

### What Are Arrays?

Arrays are variables that hold more than one value at a time.
Arrays are organized like a table.
Let's consider a spreadsheet as an example.
A spreadsheet acts like a *two-dimensional array*.
It has both rows and columns, and an individual cell in the spreadsheet can be located according to its row and column address.
An array behaves the same way.
An array has cells, which are called *elements*, and each element contains data.
An individual array element is accessed using an address called an *index* or *subscript*.

Most programming languages support *multidimensional arrays*.
A spreadsheet is an example of a multidimensional array with two dimensions, width and height.
Many languages support arrays with an arbitrary number of dimensions, though two- and three-dimensional arrays are probably the most commonly used.

Arrays in `bash` are limited to a single dimension.
We can think of them as a spreadsheet with a single column.
Even with this limitation, there are many applications for them.
Array support first appeared in `bash` version 2.
The original Unix shell program, `sh`, did not support arrays at all.

### Creating an Array

Array variables are named just like other `bash` variables and are created automatically when they are accessed.
Here is an example:

```
$ a[1]=foo
$ echo ${a[1]}
foo
```

Here we see an example of both the assignment and access of an array element.
With the first command, element 1 of array `a` is assigned the value `foo`.
The second command displays the stored value of element 1.
The use of braces in the second command is required to prevent the shell from attempting pathname expansion on the name of the array element.

An array can also be created with the `declare` command.

```
$ declare -a a
```

Using the `-a` option, this example of `declare` creates the array `a`.

### Assigning Values to an Array

Values may be assigned in one of two ways.
Single values may be assigned using the following syntax:

`name[subscript]=value`

where `name` is the name of the array and `subscript` is an integer (or arithmetic expression) greater than or equal to zero.
Note that an array's first element is subscript zero, not one.
`value` is a string or integer assigned to the array element.

Multiple values may be assigned using the following syntax:

`name=(value1 value2 ...)`

where `name` is the name of the array and the `value` placeholders are values assigned sequentially to elements of the array, starting with element zero.
For example, if we wanted to assign abbreviated days of the week to the array `days`, we could do this:

```
$ days=(Sun Mon Tue Wed Thu Fri Sat)
```

it is also possible to assign values to a specific element by specifying a subscript for each value.

```
$ days=([0]=Sun [1]=Mon [2]=Tue [3]=Wed [4]=Thu [5]=Fri [6]=Sat)
```

### Accessing Array Elements

So, what are arrays good for?
Just as many data-management tasks can be performed with a spreadsheet program, many programming tasks can be performed with arrays.

let's consider a simple data-gathering and presentation example.
We will construct a script that examines the modification times of the files in a specified directory.
From this data, our script will output a table showing at what hour of the day the files were last modified.
Such a script could be used to determine when a system is most active.
This script, called `hours`, produces this result:

```
$ hours .
Hour  Files Hour  Files
----  ----- ----  -----
00    0     12    11
01    1     13    7
02    0     14    1
03    0     15    7
04    1     16    6
05    1     17    5
06    6     18    4
07    3     19    4
08    1     20    1
09    14    21    0
10    2     22    0
11    5     23    0

Total files = 80
```

We execute the `hours` program, specifying the current directory as the target.
It produces a table showing, for each hour of the day (0-23), how many files were last modified.
The code to produce this is as follows:

```shell
#!/bin/bash

# hours: script to count files by modification time

usage () {
  echo "usage: ${0##*/} directory" >&2
}

# Check that argument is a directory
if [[ ! -d "$1" ]]; then
  usage
  exit 1
fi

# Initialize array
for i in {0..23}; do hours[i]=0; done

# Collect data
for i in $(stat -c %y "$1"/* | cut -c 12-13); do
  j="${i#0}"
  ((++hours[j]))
  ((+count))
done

# Display data
echo -e "Hour\tFiles\tHour\tFiles"
echo -e "----\t-----\t----\t-----"
for i in {0..11}; do
  j=$((i + 12))
  printf "%02d\t%d\t%02d\t%d\n" \
        "$i"  \
        "${hours[i]}" \
        "$j"  \
        "${hours[j]}"
done
printf "\nTotal files = %d\n" $count
```

The script consists of one function (`usage`) and a main body with four sections.
In the first section, we check that there is a command line argument and that it is a directory.
If it is not, we display the usage message and exit.

The second section initializes the array `hours`.
It does this by assigning each element a value of zero.
There is no special requirement to prepare arrays prior to use, but our script needs to ensure that no element is empty.
Note the interesting way the loop is constructed.
By employing brace expansion ({0..23}), we are able to easily generate a sequence of words for the `for` commands.

The next section gathers the data by running the `stat` program on each file in the directory.
We use cut to extract the two-digit hour from the result.
Inside the loop, we need to remove leading zeros from the hour field since the shell will try (and ultimately fail) to interpret values 00 through 09 as octal numbers.
Next, we increment the value of the array element corresponding with the hour of the day.
Finally, we increment a counter (`count`) to track the total number of files in the directory.

The last section of the script displays the contents of the array.
We fist output a couple of header lines and then enter a loop that produces four columns of output.
Lastly, we output the final tally of files.

### Array Operations

There are many common array operations.
Such things as deleting arrays, determining their size, sorting, and so on, have many applications in scripting.

#### Outputting the Entire Contents of an Array

The subscripts `*` and `@` can be used to access every element in an array.
As with positional parameters, the `@` notation is the more useful of the two.
Here is a demonstration:

```
$ animals=("a dog" "a cat" "a fish")
$ for i in ${animals[*]}; do echo $i; done
a
dog
a
cat
a
fish
$ for i in ${animals[@]}; do echo $i; done
a
dog
a
cat
a
fish
$ for i in "${animals[*]}"; do echo $i; done
a dog a cat a fish
$ for i in "${animals[@]}"; do echo $i; done
a dog
a cat
a fish
```

We create the array `animals` and assign it three two-word strings.
We then execute four loops to see the effect of word splitting on the array contents.
The behavior of notations `${animals[*]}` and `${animals[@]}` is identical until they are quoted.
The `*` notation results in a single word containing the array's contents, while the `@` notation results in three two-word strings, which matches the array's "real" contents.

#### Determining the Number of Array Elements

Using parameter expansion, we can determine the number of elements in an array in much the same way as finding the length of a string.
Here is an example:

```
$ a[100]=foo
$ echo ${#a[@]}   # number of array elements
1
$ echo ${#a[100]} # length of element 100
3
```

We create array `a` and assign the string `foo` to element 100.
Next, we use parameter expansion to examine the length of the array, using the `@` notation.
Finally, we look at the length of element 100, which contains the string `foo`.
It is interesting to note that while we assigned our string to element 100, `bash` reports only one element in the array.
This differs from the behavior of some other languages in which the unused elements of the array (elements 0-99) would be initialized with empty values and counted.
In `bash`, array elements exist only if they have been assigned a value regardless of their subscript.

#### Finding the Subscripts Used by an Array

As `bash` allows arrays to contain "gaps" in the assignment of subscripts, it is sometimes useful to determine which elements actually exist.
This can be done with a parameter expansion using the following forms:

`${!array[*]}`

`${!array[@]}`

where `array` is the name of an array variable.
Like the other expansions that use `*` and `@`, the `@` form enclosed in quotes is the most useful, as it expands into separate words.

```
$ foo=([2]=a [4]=b [6]=c)
$ for i in "${foo[@]}"; do echo $i; done
a
b
c
$ for i in "${!foo[@]}"; do echo $i; done
2
4
6
```

#### Adding Elements to the End of an Array

Knowing the number of elements in an array is no help if we need to append values to the end of an array since the values returned by the `*` and `@` notations do not tell us the maximum array index in use.
Fortunately, the shell provides us with a solution.
By using the `+=` assignment operator, we can automatically append values to the end of an array.
Here, we assign three values to the array `foo` and then append three more:

```
$ foo=(a b c)
$ echo ${foo[@]}
a b c
$ foo+=(d e f)
$ echo ${foo[@]}
a b c d e f
```

#### Sorting an array

Just as with spreadsheets, it is often necessary to sort the values in a column of data.
The shell has no direct way of doing this, but it's not hard to do with a little coding.

```shell
#!/bin/bash

# array-sort: Sort an array

a=(f e d c b a)

echo "Original array: ${a[@]}"
a_sorted=($(for i in "${a[@]}"; do echo $i; done | sort))
echo "Sorted array: ${a_sorted[@]}"
```

When executed, the script produces this:

```
$ array-sort
Original array: f e d c b a
Sorted array: a b c d e f
```

The script operates by copying the contents of the original array (`a`) into a second array (`a_sorted`) with a tricky piece of command substitution.
This basic technique can be used to perform many kinds of operations on the array by changing the design of the pipeline.

#### Deleting an Array

To delete an array, use the `unset` command.

```
$ foo=(a b c d e f)
$ echo ${foo[@]}
a b c d e f
$ unset foo
$ echo ${foo[@]}

$
```

`unset` may also be used to delete single array elements.

```
$ foo=(a b c d e f)
$ echo ${foo[@]}
a b c d e f
$ unset 'foo[2]'
$ echo ${foo[@]}
a b d e f
```

In this example, we delete the third element of the array, subscript 2.
Remember, arrays start with subscript zero, not one!
Notice also that the array element must be quoted to prevent the shell from performing pathname expansion.

Interestingly, the assignment of an empty value to an array does not empty its contents.

```
$ foo=(a b c d e f)
$ foo=
$ echo ${foo[@]}
b c d e f
```

Any reference to an array variable without a subscript refers to element zero of the array.

```
$ foo=(a b c d e f)
$ echo ${foo[@]}
a b c d e f
$ foo=A
$ echo ${foo[@]}
A b c d e f
```

### Associative Arrays

`bash` versions 4.0 and greater support *associative arrays*.
Associative arrays use strings rather than integers as array indexes.
This capability allow interesting new approaches to managing data.
For example, we can create an array called `colors` and use color names as indexes.

```shell
declare -A colors
colors["red"]="#ffoooo"
colors["green"]="#00ff00"
colors["blue"]="0000ff"
```

Unlike integer indexed arrays, which are created by merely referencing them, associative arrays must be created with the `declare` command using the new `-A` option.
Associative array elements are accessed in much the same way as integer-indexed arrays.

```
echo ${colors["blue"]}
```

### Summary

If we search the `bash` man page for the word *array*, we find many instances of where `bash` makes use of array variables.
Most of these are rather obscure, but they may provide occasional utility in some special circumstances.
In fact, the entire topic of arrays is rather under-utilized in shell programming owing largely to the fact that the traditional Unix shell programs (such as `sh`) lacked any support for arrays.
This lack of popularity is unfortunate because arrays are widely used in other programming languages and provide a powerful tool for solving many kinds of programming problems.

Arrays and loops have a natural affinity and are often used together.
The following form of loop is particularly well-suited to calculating array subscripts:

`for ((expr; expr; expr))`

---

## Exotica

While we have covered a lot of ground in the previous chapters, there are many `bash` features that we have not covered.
Most are fairly obscure and useful mainly to those integrating `bash` into a Linux distribution.
However, there are a few that, while not in common use, are helpful for certain programming problems.
We will cover them here

### Group Commands and Subshells

`bash` allows commands to be grouped together.
This can be done in one of two ways, either with a *group command* or with a *subshell*.

Here is the syntax of a group command:

`{ command1; command2; [command3; ...] }`

Here is the syntax of a subshell:

`(command1; command2; [command3;...])`

The two forms differ in that a group command surrounds its commands with braces and a subshell uses parentheses.
It is important to note that because of the way `bash` implements group commands, the braces must be separated from the commands by a space and the last command must be terminated with either a semicolon or a newline prior to the closing brace.

So, what are group commands and subshells good for?
While they have an important difference, they are both used to manage redirection.
Let's consider a script segment that performs redirections on multiple commands.

```shell
ls -l > output.txt
echo "Listing of foo.txt" >> output.txt
cat foo.txt >> output.txt
```

This is pretty straightforward.
Three commands have their output redirected to a file named *output.txt*.
Using a group command, we could code this as follows:

`{ ls -l; echo "Listing of foo.txt"; cat foo.txt; } > output.txt`

Using a subshell is similar.

`(ls -l; echo "Listing of foo.txt"; cat foo.txt) > output.txt`

Using this technique we have saved ourselves some typing, but where a group command or subshell really shines is with pipelines.
When constructing a pipeline of commands, it is often useful to combine the results of several commands into a single stream.
Group commands and subshells make this easy.

`{ ls -l; echo "Listing of foo.txt"; cat foo.txt; } | lpr`

Here we have combined the output of our three commands and piped them into the input of `lpr` to produce a printed report.

In the script that follows, we will use groups commands and look at several programming techniques that can be employed in conjunction with associative arrays.
This script, called `array-2`, when given the name of a directory, prints a listing of the files in the directory along with the names of the file's owner and group owner.
At the end of the listing, the script prints a tally of the number of files belonging to each owner and group.
Here we see the results (condensed for brevity) when the script is given the directory */usr/bin*:

```
$ array-2 /usr/bin
/usr/bin/2to3                               root       root
/usr/bin/a2p                                root       root
/usr/bin/abrowser                           root       root
/usr/bin/aconnect                           root       root
/usr/bin/acpi_fakekey                       root       root
/usr/bin/acpi_listen                        root       root
/usr/bin/add-apt-repository                 root       root
--snip--
/usr/bin/zipgrep                            root       root
/usr/bin/zipinfo                            root       root
/usr/bin/zipnote                            root       root
/usr/bin/zip                                root       root
/usr/bin/zipsplit                           root       root
/usr/bin/zjsdecode                          root       root
/usr/bin/zsoelim                            root       root

File owners:
daemon  :      1 file(s)
root    :   1394 file(s)

File group owners:
crontab :      1 file(s)
daemon  :      1 file(s)
lpadmin :      1 file(s)
mail    :      4 file(s)
mlocat  :      1 file(s)
root    :   1380 file(s)
shadow  :      2 file(s)
ssh     :      1 file(s)
tty     :      2 file(s)
utmp    :      2 file(s)
```

Here is a listing (with line numbers) of the script:

```shell
#!/bin/bash

# array-2: Use arrays to tally file owners

declare -A files file_group file_owner groups owners

if [[ ! -d "$1" ]]; then
  echo "Usage: array-2 dir" >&2
  exit 1
fi

for i in "$1"/*; do
  owner="$(stat -c %U "$i")"
  group="$(stat -c %G "$i")"
  files["$i"]="$i"
  file_owner["$i"]="$owner"
  file_group["$i"]="$group"
  ((++owners[$owner]))
  ((++groups[$group]))
done

# List the collected files
{ for i in "${files[@]}"; do
  printf "%-40s %-10s %-10s\n"  \
      "$i" "${file_owner["$i"]}" "${file_group["$i"]}"
  done } | sort
echo

# List owners
echo "File ownders:"
{ for i in "${!owners[@]}"; do
    printf "%-10s: %5d file(s)\n" "$i" "${owners["$i"]}"
  done } | sort
echo

# List groups
echo "File group owners"
{ for i in "${!groups[@]}"; do
    printf "%-10s: %5d file(s)\n" "$i" "${groups["$i"]}"
  done } | sort
```

Let's take a look at the mechanics of this script.

**Line 5:** Associative arrays must be created with the `declare` command using the `-A` option.
In this script, we create five arrays as follows:

+ `files` contains the names of the files in the directory, indexed by filename.

+ `file_group` contains the group owner of each file, indexed by filename.

+ `file_owner` contains the owner of each file, indexed by filename.

+ `groups` contains the number of files belonging to the indexed group.

+ `owners` contains the number of files belonging to the indexed owner.

**Lines 7-10:** These lines check to see that a valid directory name was passed as a positional parameter.
If not, a usage message is displayed, and the script exits with an exit status of 1.

**Lines 12-20:** These lines loop through the files in the directory.
Using the `stat` command, lines 13 and 14 extract the names of the file owner and group owner and assign the values to their respective arrays (lines 16 and 17) using the name of the file as the array index. Likewise, the filename itself is assigned to the `files` array (line 15).

**Lines 18-19:** The total number of files belonging to the file owner and group owner are incremented by one.

**Lines 22-27:** The list of files is output.
This is done using the `"${array[@]}"` parameter expansion, which expands into the entire list of array elements with each element treated as a separate word.
This allows for the possibility that a filename may contain embedded spaces.
Also note that the entire loop is enclosed in braces thus forming a group command.
This permits the entire output of the loop to be piped into the `sort` command.
This is necessary because the expansion of the array elements is not sorted.

**Lines 29-40:** These two loops are similar to the file list loop except that they use the `"${!array[@]}"` expansion, which expands into the list of array indexes rather than the list of array elements.

#### Process Substitution

While they look similar and can both be used to combine streams for redirection, there is an important difference between group commands and subshells.
Whereas a group command executes all of its commands in the current shell, a subshell (as the name suggests) executes its commands in a child copy of the current shell.
This means the environment is copied and given to a new instance of the shell.
When the subshell exits, the copy of the environment is lost, so any changes made to the subshell's environment (including variable assignment) are lost as well.
Therefore, in most cases, unless a script requires a subshell, group commands are preferable to subshells.
Group commands are both faster and require less memory.

We saw an example of the subshell environment problem when we discovered that a `read` command in a pipeline does not work as we might intuitively expect.
To recap, if we construct a pipeline like this:

```shell
echo "foo" | read
echo $REPLY
```

the content of the `REPLY` variable is always empty because the `read` command is executed in a subshell, and its copy of `REPLY` is destroyed when the subshell terminates.

Because commands in pipelines are always executed in subshells, any command that assigns variables will encounter this issue.
Fortunately, the shell provides an exotic form of expansion called *process substitution* that can be used to work around this problem.

Process substitution is expressed in two ways.

For processes that produce standard output, it looks like this:

`<(list)`

For processes that intake standard input, it looks like this:

`>(list)`

where `list` is a list of commands.

To solve our problem with `read`, we can employ process substitution like this.

```shell
read < <(echo "foo")
echo $REPLY
```

Process substitution allows us to treat the output of a subshell as an ordinary file for purposes of redirection.
In fact, since it is a form of expansion, we can examine its real value.

```
$ echo <(echo "foo")
/dev/fd/63
```

By using `echo` to view the result of the expansion, we see that the ouput of the subshell is being provided by a file name */dev/fd/63*.

Process substitution is often used with loops containing `read`.
Here is an example of a `read` loop that processes the contents of a directory listing created by a subshell:

```shell
#!/bin/bash

# pro-sub: demo of process substitution

while read attr links owner group size date time filename; do
  cat << EOF
        Filename:   $filename
        Size:       $size
        Owner:      $owner
        Group:      $group
        Modified:   $date $time
        Links:      $links
        Attributes: $attr

EOF
done < <(ls -l | tail -n +2)
```

The loop executes `read` for each line of a directory listing.
The listing itself is produced on the final line of the script.
This line redirects the output of the process substitution into the standard input of the loop.
The `tail` command is included in the process substitution pipeline to eliminate the first line of the listing, which is not needed.

When executed, the script produces output like this:

```
$ pro-sub | head -n 20
Filename:   addresses.ldif
Size:       14540
Owner:      me
Group:      me
Modified:   2009-04-02 11:12
Links:      1
Attributes: -rw-r--r--

Filename:   bin
Size:       4096
Owner:      me
Group:      me
Modified:   2009-07-10 07:31
Links:      2
Attributes: drwxr-xr-x

Filename:   bookmarks.html
Size:       394213
Owner       me
Group:      me
```

### Traps

We saw how programs can respond to signals.
We can add this capability to our scripts, too.
While the scripts we have written so far have not needed this capability (because they have very short execution times and do not create temporary files), larger and more complicated scripts may benefit from having a signal handling routine.

When we design a large, complicated script, it is important to consider what happens if the user logs off or shuts down the computer while the script is running.
When such an event occurs, a signal will be sent to all affected processes.
In turn, the programs representing those processes can perform actions to ensure a proper and orderly termination of the program.
Let's say, for example, that we wrote a script that created a temporary file during its execution.
In the course of good design, we would have the script delete the file when the script finishes its work.
It would also be smart to ahve the script delete the file if a signal is received indicating that the program was going to be terminated prematurely.

`bash` provides a mechanism for this purpose known as a `trap`.
Traps are implemented with the appropraitely named builtin command, `trap`.
`trap` uses the following syntax:

`trap argument signal [signal...]`

where `argument` is a string that will be read and treated as a command and `signal` is the specification of a signal that will trigger the execution of the interpreted command.

Here is a simple example:

```shell
#!/bin/bash

# trap-demo: simple signal handling demo

trap "echo 'I am ignoring you.'" SIGINT SIGTERM

for i in {1..5}; do
  echo "Iteration $i of 5"
  sleep 5
done
```

This script defines a trap that will execute an `echo` command each time either the `SIGINT` or `SIGTERM` signal is received while the script is running.

Execution of the program looks like this when the user attempts to stop the script by pressing `CTRL-C`:

```
$ trap-demo
Iteration 1 of 5
Iteration 2 of 5
^CI am ignoring you.
Iteration 3 of 5
^CI am ignoring you.
Iteration 4 of 5
Iteration 5 of 5
```

As we can see, each time the user attempts to interrupt the program, the message is printed instead.

Constructing a string to form a useful sequence of commands can be awkward, so it is common practice to specify a shell function as the command.
In this example, a separate shell function is specified for each signal to be handled:

```shell
#!/bin/bash

# trap-demo2: simple signal handling demo

exit_on_signal_SIGINT () {
  echo "Script interrupted." 2>&1
  exit 0
}

exit_on_signal_SIGTERM () {
  echo "Script terminated." 2>&1
  exit 0
}

trap exit_on_signal_SIGINT SIGINT
trap exit_on_signal_SIGTERM SIGTERM

for i in {1..5}; do
  echo "Iteration $i of 5"
  sleep 5
done
```

This script features two `trap` commands, one for each signal.
Each trap, in turn, specifies a shell function to be executed when the particular signal is received.
Note the inclusion of an `exit` command in each of the signal handling functions.
Without an `exit`, the script would continue after completing the function.

When the user presses `CTRL-C` during the execution of this script, the results look like this:

```
$ trap-demo2
Iteration 1 of 5
Iteration 2 of 5
^CScript interrupted.
```

#### Temporary files

One reason signal handlers are included in scripts is to remove temporary files that the script may create to hold intermediate results during execution.
There is something of an art to naming temporary files.
Traditionally, programs on Unix-like systems create their temporary files in the `/tmp` directory, a shared directory intended for such files.
However, since the directory is shared, this poses certain security concerns, particularly for programs running with superuser privileges.
Aside from the obvious step of setting proper permissions for files exposed to all users of the system, it is important to give temporary files nonpredictable filenames.
This avoids an exploit known as a *temp race attack*.
One way to create a nonpredictable (but still descriptive) name is to do something like this:

`tempfile=/tmp/$(basename $0).$$.$RANDOM`

This will create a filename consisting of the program's name, followed by its process ID (PID), followed by a random integer.
Note, however that the `$RANDOM` shell variable returns a value only in the range of 1-32767, which is not a large range in computer terms, so a single instance of the variable is not sufficient to overcome a determined attacker.

A better way is too use the `mktemp` program (not to be confused with the `mktemp` standard library function) to both name and create the temporary file.
The `mktemp` program accepts a template as an argument that is used to build the filename.
The template should include a series of X characters, which are replaced by a corresponding number of random letters and numbers.
The longer the series of X characters, the longer the series of random characters.
Here is an example:

`tempfile=$(mktemp /tmp/foobar.$$.XXXXXXXXXX)`

This creates a temporary file and assigns its name to the variable `tempfile`.
The X characters in the template are replaced with random letters and numbers so that the final filename (which, in this example, also includes the expanded value of the special parameter `$$` to obtain the PID) might be something like this:

`/tmp/foobar.6593.UOZuvM6654`

For scripts that are executed by regular uses, it may be wise to avoid the use of the `/tmp` directory and create a directory for temporary files within the user's home directory, with a line of code such as this:

`[[ -d $HOME/tmp ]] || mkdir $HOME/tmp`

### Asynchronous Execution with wait

It is sometimes desirable to perform more than one task at the same time.
We have seen how all modern operating systems are at least multitasking if not multiuser as well.
Scripts can be constructed to behave in a multitasking fashion.

Usually this involves launching a script that, in turn, launches one or more child scripts to perform an additional task while the parent script continues to run.
However, when a series of scripts runs this way, there can be problems keeping the parent and child coordinated.
That is, what if the parent or child is dependent on the other and one script must wait for the other to finish its task before finishing its own?

`bash` has a builtin command to help manage *asynchronous execution* such as this.
The `wait` command causes a parent script to pause until a specified process (i.e., the child script) finishes.
To demonstrate this, we will need two scripts.
The first is a parent script.

```shell
#!/bin/bash

# async-parent: Asynchronous execution demo (parent)

echo "Parent: starting..."

echo " Parent: launching child script..."
async-child &
pid=$!
echo "Parent: child (PID=$pid) launched."

echo "Parent: continuing..."
sleep 2

echo "Parent: pausing to wait for child to finish..."
wait "$pid"

echo "Parent: child is finished. Continuing..."
echo "Parent: parent is done. Exiting."
```

This second is a child script.

```shell
#!/bin/bash

# async-child: Asynchronous execution demo (child)

echo "Child: child is running..."
sleep 5
echo "Child: child is done. Exiting."
```

In this example, we see that the child script is simple.
The real action is being performed by the parent.
In the parent script, the child script is launched and put into the background.
The process ID of the child script is recorded by assigning the `pid` variable with the value of the `$!` shell parameter, which will always contain the process ID of the last job put into the background.

The parent script continues and then executes a `wait` command with the PID of the child process.
This causes the parent script to pause until the child script exits, at which point the parent script concludes.

When executed, the parent and child scripts produce the following output:

```
$ async-parent
Parent: starting...
Parent: launching child script...
Parent: child (PID= 6741) launched.
Parent: continuing...
Child: child is running...
Parent: pausing to wait for child to finish...
Child: child is done. Exiting.
Parent: child is finished. Continuing...
Parent: parent is done. Exiting.
```

### Named Pipes

In most Unix-like systems, it is possible to create a special type of file called a *named pipe*.
Named pipes are used to create a connection between two processes and can be used just like other types of files.
They are not that popular, but they're good to know about.

There is a common programming architecture called *client-server*, which can make use of a communication method such as named pipes, as well as other kinds of *interprocess communication* such as network connections.

The most widely used type of client-server system is, of course, a web browser communicating with a web server.
The web browser acts as the client, making requests to the server, and the server responds to the browser with web pages.

Named pipes behave like files but actually form first-in-first-out (FIFO) buffers.
As with ordinary (unnamed) pipes, data goes in one end and emerges out the other.
With named pipes, it is possible to set up something like this:

`process1 > named_pipe`

and this:

`process2 < named_pipe`

and it will behave like this:

`process1 | process2`

#### Setting Up a Named Pipe

First, we must create a named pipe.
This is done using the mkfifo command.

```
$ mkfifo pipe1
$ ls -l pipe1
prw-r--r-- 1 me   me    0 2018-07-17  06:41 pipe1
```

Here we use `mkfifo` to create a named pipe called `pipe1`.
Using `ls`, we examine the file and see that the first letter in the attributes field is `p`, indicating that it is a named pipe.

#### Using Named Pipes

To demonstrate how the named pipe works, we will need two terminal windows (or alternately, two virtual consoles).
In the first terminal, we enter a simple command and redirect its output to the named pipe.

`$ ls -l > pipe1`

After we press `ENTER`, the command will appear to hang.
This is because there is nothing receiving data from the other end of the pipe yet.
When this occurs, it is said that the pipe is *blocked*.
This condition will clear once we attach a process to the other end and it begins to read input from the pipe.
Using the second terminal window, we enter this command:

`$ cat < pipe1`

The directory listing produced from the first terminal window appears in the second terminal as the output from the `cat` command.
The `ls` command in the first terminal successfully completes once it is no longer blocked.

### Summary

Even though we covered a lot of ground in our trek, we barely scratched the surface as far as the command line goes.
There are still thousands of command line programs left to be discovered and enjoyed.
Start digging around in */usr/bin* and you'll see!

---
