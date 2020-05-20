# Bash primer

A quick primer to learn bash to automate stuff in Unix/Linux.

Shell or Bash in this case is used as a universal glue language to issue a string of unix commands to the system.

> **Note: Dont use Bash everywhere!**
> 
> Although Bash is universal and elegant to programmatically execute unix commands, and apply logic to them, it can quickly get cumbersome once you start doing complex things like working with floating point numbers or find yourself needing arrays or other sophisticated datastructures. Once your script gets more complex, stop and consider writing your program in higher level programming languages like Python or Go.


## A simple bash script
```bash
#!/bin/bash 
echo 'Hello World'
```

+ `#!/bin/bash` is called a shebang, which tells the system how to run this script in case it is executed directly on the shell.

+ Save the above script as hello.sh and run it as below..

```bash
chmod +x hello.sh
./hello.sh
```
or 

```bash
bash hello.sh
```

## Comments
```bash
#!/bin/bash 
# This is a comment
echo 'Hello World'
```

## Variables

Bash does not need variables to be declared of certain type. They may be numbers or characters.

```bash
#!/bin/bash 
# This is how you set a value to a variable
myvariable=Hello
## Note : Variables are globally scoped by default. 
## There is _no_ space before and after = sign.

# This is how you get the value of the variable later in the script
echo $myvariable
# Another variable holding a directory path
dir=/usr/home/qgg/asampath
ls -alh $dir

# Local variable
local local_var="I'm a local value"

```
You can also set _environment_ variables that are accessible to any program running in the current shell session.

There are also special environment variables that are already available for your use. Some examples below.

```bash
#!/bin/bash 
export SECRET="I'm a secret string used later"
echo $SECRET
# User's home directory
echo $HOME
# A colon-separated list of directories where
#         the shell looks for programs or commands to execute from.
echo $PATH
# Current working directory
echo $PWD
# User ID(numeric UID) of the current user
$UID
```

## Quotes

```bash
#!/bin/bash 

# Single quote and Double quote can be used interchangeably.
firstname="Aravindh"
lastname='Sampathkumar'
echo $firstname
echo $lastname
echo $firstname $lastname

# Escaping quotes - use alternative quotes to escape the ones you need. 
echo "'a'"    # Will print 'a'  
echo '"b"'    # Will print "b"

# If you need both quotes in a string Escape it with backslash
echo "'aaa' \"bbb\""   # will print 'aaa' "bbb"
```

## Command substitution

For when you want the output of a command to be stored as value of a variable.

```bash
#!/bin/bash 

myvar=$( ls /etc | wc -l )
echo There are $myvar entries in the directory /etc
now=$(date +%T)
echo $now # 19:08:26
```

## Exit codes

Every command returns an exit code (return status or exit status).

A `0` exit code means successful completion of the command.

A non-zero (1 to 255) exit code means a failed return from the command.

Exit code is assigned to the $? environment variable. We can use this to test whether a script or a command finished successfully or not.

__Example__:
```bash
uid=$(id -u asampath)
if [ $? -eq 0 ]; then
        echo "INFO: User account verified of existence. "
else
        echo "ERROR: User account was not created properly. " >&2
        exit 1
fi
```

`exit` keyword can be used to exit a function or terminate a script. In both cases it returns the exit code of the last command executed before `exit`.

## Arithmetic operations
Arithmetic expression must enclosed by `$(( ))`
```bash
x=1
y=2
echo $(( 1 + 3 ))     # 4
echo $(( ++x + y++ )) # 4
echo $(( x + y ))     # 5
```

## Commandline arguments

Sometimes, you need arguments to be passed to your script. For example copy <src_file> <dest_file>
```bash
#!/bin/bash 

cp $1 $2
echo Copied $1 to $2
```
Save the above script as copy.sh and execute the above script passing the parameters as 
```bash
./copy.sh test1 test2
```
### Streams

Streams are simply sequences of characters that may be redirected into files or other streams. 

| Code   | Descriptor      | Description      |
| ------ | --------------- | ---------------- | 
| 0      | stdin           | Standard input   | 
| 1      | stdout          | Standard output  | 
| 2      | stderr          | Standard error   | 

__Redirections__ are what makes streams useful. 

| Operator   | Description                                            |
| ---------- | ------------------------------------------------------ |
| `>`          | Redirect output                                      |
| `2>`         | Redirect error                                       |
| `&>`         | Redirect output and error                            |  
| `>>`         | Redirect output but append to destination            |
| `2>>`        | Redirect error but append to destination             |
| `&>>`        | Redirect output and error but append to destination  |
| `<`          | Redirect input to this command/script                |  

```bash
command > out.log
command >> out.log
command 2> err.log
command 2>> err.log
command &> out_and_err.log
command &>> out_and_err.log
command 2> err.log 1> out.log
command 2>> err.log 1>> out.log
```
**How about a T junction in streams?**

`| tee` is used to create T break in a stream - to redirect the same stream to 2 destinations. 

```bash
echo $(cmd) | tee file1 file2

# Redirect both stdout and stderr to console and write them to a file
command 2>&1 | tee out.log
```

## Pipes
Pipes let us use the output of a program as the input of another.

```bash
# output of command1 is fed as input to command2....
command1 | command2 | command3

# Pipe the output of myst command to grep and filter on pattern R (running jobs)
myst | grep R
```
> Note: exit status of a pipeline is the exit code of the last command in the pipeline. If you need exit status to be the failure code if _any_ of the commands in the pipe had failed, use the following shell option at the beginning of the script. 
```bash
set -o pipefail
```

## Command sequences
To execute multiple commands conditionally based on whether the prior command succeeded or not, you can use `;`, `&`, `&&` or `||` operators.

command2 will be executed after command1 regardless of success or failure of command1
```bash
command1 ; command2
```

command2 will be executed if, and only if, command1 finishes successfully (returns 0 exit status)
```bash
command1 && command2
```

command2 will be executed if, and only if, command1 finishes unsuccessfully (returns code of error)
```bash
command1 || command2
```

If a command is terminated by the control operator &, the shell executes the command asynchronously in a subshell (in the background)
```bash
command1 &
command2 &
command3 &
```
In the above example, all three commands run _concurrently_ in the background in their own subshells.

What if you want the "parent" script to wait until or some of the background tasks to finish before doing something else? 

`wait` command is your answer.

https://stackoverflow.com/questions/1131484/wait-for-bash-background-jobs-in-script-to-be-finished

## Conditional execution

`if` statement and `case` statement - decide to perform an action or not.

Expression should be enclosed in `[[ ]]`

**String based expressions**

| Expression         | Meaning                           |
| ------------------ | --------------------------------- |
| [[ -z STR ]]       | STR is empty                      | 
| [[ -n STR ]]       | STR is not empty                  | 
| [[ STR1 == STR2 ]] | STR1 and STR2 are equal           | 
| [[ STR1 != STR2 ]] | STR1 and STR2 are not equal       | 

**Numeric expressions**

| Expression           | Meaning                               |
| -------------------- | ------------------------------------- |
| [[ NUM1 -eq NUM2 ]]  | NUM1 and NUM2 are equal               | 
| [[ NUM1 -ne NUM2 ]]  | NUM1 and NUM2 are not equal           |
| [[ NUM1 -lt NUM2 ]]  | NUM1 is lesser than NUM2              |
| [[ NUM1 -le NUM2 ]]  | NUM1 is less than or equal to NUM2    |
| [[ NUM1 -gt NUM2 ]]  | NUM1 is greater than NUM2             |
| [[ NUM1 -ge NUM2 ]]  | NUM1 is greater than or equal to NUM2 |

**Logical evaluations**

| Expression                   | Meaning                                               |
| ---------------------------- | ----------------------------------------------------- |
| [[ ! EXPR ]]                 | NOT operator. True if EXPR is false                   | 
| [[ EXPR1 ]] && [[ EXPR2 ]]   | AND operator. True if both EXPR1 and EXPR2 are true   |
| [[ EXPR1 -a EXPR2 ]]         | AND operator. True if both EXPR1 and EXPR2 are true   |
| [[ EXPR1 ]] || [[ EXPR2 ]]   | OR operator. True if either EXPR1 or EXPR2 is true    |
| [[ EXPR1 -o EXPR2 ]]         | OR operator. True if either EXPR1 or EXPR2 is true    |

**File condition based expressions**

| Expression           | Meaning                                      |
| -------------------- | -------------------------------------------- |
| [[ -e FILE ]]        | True if FILE exists                          | 
| [ -f FILE ]          | True if FILE exists and is a regular file    |
| [ -d FILE ]          | True if FILE exists and is a directory       |
| [ -r FILE ]          | True if FILE exists and is readable          |
| [ -w FILE ]          | True if FILE exists and is writable          |
| [ -x FILE ]          | True if FILE exists and is executable        |
| [ -L FILE ]          | True if FILE exists and is symbolic link     |
| [ FILE1 -nt FILE2 ]  | FILE1 is newer than FILE2.                   |
| [ FILE1 -ot FILE2 ]  | FILE1 is older than FILE2.                   |

**Conditional execution**

```bash
if [[ 1 -eq 1 ]]; then echo "true"; fi
```

```bash
if [[ 1 -eq 1 ]]; then
  echo "true"
fi
```

```bash
if [[ 2 -ne 1 ]]; then echo "true"; else echo "false"; fi
```

```bash
if [[ 2 -ne 1 ]]; then
  echo "true"
else
  echo "false"
fi
```

```bash
case "$action" in
  start | up)
    programname start
    ;;

  *)
    echo "Error: Expected {start|up}"
    ;;
esac
```

## Flow control - for and while loops
**Range based for loops**

```bash
for i in {1..5}; do echo $i; done
```

```bash
for (( i = 0; i < 10; i++ )); do
  echo $i
done
```

```bash
for i in {5..50..5}; do
    echo $i
done
```

**Looping action on files**

```bash
for FILE in $HOME/*.py; do
  mv "$FILE" "${HOME}/src/"
  chmod +x "${HOME}/scripts/${FILE}"
done
```
**While loop**

```bash
x=0
while [[ $x -lt 10 ]]; do # value of x is less than 10
  echo $x
  x=$(( x + 1 )) # increment x
done
```

**Infinite loop**
```bash
while true; do
  echo "Please hit ctrl+c to stop me!"
done
```

**Reading a file line-by-line**
```bash
file="$HOME/data.txt"
while IFS= read -r line
do
	printf '%s\n' "$line"
done <"$file"
```
Split by fields
```bash
file="$HOME/data.txt"
while IFS=: read -r f1 f2 f3 f4 f5 f6 f7
do
        # display fields using f1, f2,..,f7
        printf 'ID: %s, Name: %s, Email: %s\n' "$f1" "$f7" "$f6"
done <"$file"
```
**Command output can be treated as files**
```bash
emails=$(ipa user-find all|grep Email|awk '{print $2}')
while IFS= read -r emailaddr
do
    printf 'Emailing to %s...\n' "$emailaddr"
    ghpcmail -t $emailaddr "Hello"
done <<< "$emails"
```
**Using while loop to read file line-by-line**
```bash
cat data.txt | while read line; do
  echo $line
done
```
**Loop control**

`break` statement exits the current loop before its ending. 

`continue` statement steps over one iteration.

## Functions

Functions must be declared before they are invoked.

Functions can take arguments and return an [exit code](#exit-codes).
```bash
hellofn() {
    echo "hello $1"
}

hellofn $USER
```
```bash
# Fn with explicit return codes
is_root_user(){
 [ $(id -u) -eq 0 ] && return 0 || return 1
}

is_root_user && echo "run as root... OK" || echo "Err: Need to be root"
```
```bash
# Fn that returns a string instead of just exit code
in_users(){
        users=$(w|awk '{print $1}'|tail --lines=+3)
        echo $users
}
current_users=$(in_users)
echo $current_users
```
## Libraries and functions from other scripts

So, you want to create a library of functions that can be re-used in many scripts. 

Add a file such as mylib.sh that contains your re-usable functions.
```bash
#Purpose: Make a directory and enter into it
#Args: Name of directory
#Returns: nothing
mkcd() { mkdir -p $1; cd $1 }

#Purpose: Tar GZ compress a directory or file
#Args: Name of file or directory
#Returns: nothing
targz() { tar -zcvf $1.tar.gz $1; }

#Purpose: Extract a Tar GZ archive to current location
#Args: Name of tar.gz archive as xyz.tar.gz
#Returns: nothing
untargz() { tar -zxvf $1; }
```
Now, you can use these functions in any script as shown below..
```bash
#!/bin/bash
. $HOME/mylib.sh
mkcd new_dir_for_data
targz DATA 
untargz DATA.tar.gz
```