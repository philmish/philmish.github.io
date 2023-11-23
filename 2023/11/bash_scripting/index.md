# Bash Scripting Reference


This post is a small summary of the basics of bash scripting. It contains some useful code snippets as well. 


## Parsing Arguments to a script with `getopts`

```bash
ARGS=$@
if [[ ${#ARGS} -ne 0 ]]; then
	while getopts ":sT:h" ARG; do
	case "$ARG" in
		s) echo "-s flag set" ;;
		T) echo "-T flag set"
		   echo "The value was $OPTARG" ;;
		h) help ;;
		:) echo "Argument missing for $ARG" ;;
		*) echo "Invalid flag $ARG."
	esac
	done
else
	help
	exit 1
fi
shift "$((OPTIND-1))"
```

The above script takes a list of required arguments. Should no arguments be provided the script defaults to calling the `help` function.
Adding the `:` after the `T` flag in the declaration of the while loop will tell the script that we cant to pass a value with the flag.
After parsing is done we reset `OPTIND`. It is used by `getopts` to point to the next argument to be processed.
## Reference Overview

- `$1` bis `$9` reference the first 9 arguments passed to a script
- `$0` is the script itself
- `$#` holds the number of arguments passed to the script
- `$@` can be used to retrieve the list of command line arguments passed
- `$$` process ID of the currently executing process
- `$?` exit status of the script, is 0 if the script ran successfully and 1 if there was a failure
- variable declarations do not contain white spaces between the name, the assign operator and the value, example: `var=val`
- all variables are treated as string characters, there are no actual data types
- arrays of values are declared with brackets and delimit the values with white spaces, example `myarray=(val1 val2 val3)`
- array index starts at 0
- values are retrieved with brackets containing the index, example `$myarray[0]`
- there are 4 different comparison operators
	1. string operators
	2. integer operators
	3. file operators
	4. boolean operators
- arithmetic operations can be performed on values by wrapping them in double brackets, example: `val=$((10 + 10))`
- by adding the `-x` (for `xtrace`) flag to an invocation of `bash` we can get verbose output from our scripts containing information about the executed code for debugging purposes
- to get more details while debugging we can also add the `-v` flag for even more verbose output

### String comparison operators

| Operator | Description |
|-----------|--------------|
| == | is equal to |
| != | is not equal to |
| < | is less than in ASCII alphabetical order |
| > | is greater than in ASCII alphabetical order |
| -z | if the string is empty (null) |
| -n | if the string is not null |
| =~ | if the left hand side contains the right hand side |

### Integer comparison operators

| Operator | Description |
|-----------|--------------|
| -eq | is equal to |
| -ne | is not equal to |
| -lt | is less than |
| -le | is less than or equal to |
| -gt | is greater than |
| -ge | is greater than or equal to |


### File Operators

| Operator | Description |
|-----------|--------------|
| -e | if the file exists |
| -f | test if it is a file |
| -d | test if it is a directory |
| -L | test if it is a symbolic link |
| -N | checks if the file was modified after it was last read |
| -O | if the current user owns the file |
| -G | if the file's group id matches the current users' |
| -s | test if the file has a size greater than 0 |
| -r | test if the file has read permissions |
| -w | test if the file has write permissions |
| -x | test if the file has execute permissions |


### Arithmetic operators

| Operator | Description |
|-----------|--------------|
| + | Addition |
| - | Subtraction |
| * | Multiplication |
| / | Division |
| % | Modulus |
| variable++ | Increment variable by 1 |
| variable-- | Decrement variable by 1 |


### Return Codes

| Return Code | Description |
|-----------|--------------|
| 1 | General Errors |
| 2 | Misuse of shellbuiltins |
| 126 | Command invoked can not be executed |
| 127 | Command not found |
| 128 | Invalid arguments to exit |
| 128+n | Fatal error signal "n" |
| 130 | Script terminated by Control-C |
| 255/* | Exit status out of range |


