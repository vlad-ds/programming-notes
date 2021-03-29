# Bash scripting

The first line will specify where to find Bash. Usually `#!/usr/bash`. To find where Bash is you can use `which bash `. To run a script use `bash script_name.sh`. 

```bash
#!/usr/bash
echo "Hello world"
echo "Goodbye world"
```

A script renaming two fields in a CSV and saving the data in a new CSV. 

```bash
#!/bin/bash
# Create a sed pipe to a new file
cat soccer_scores.csv | sed 's/Cherno/Cherno City/g' | sed 's/Arda/Arda United/g' > soccer_scores_edited.csv
```

There are 3 streams for your program:

* STDIN (standard input). Data coming into the program.
* STDOUT (standard output). Data coming out of the program.
* STDERR (standard error). Errors in your program.

In the script, 1 is STDOUT and 2 is STDERR. `2> /dev/null` ridirects STDERR to be deleted. 

When you use the piping operator `|` , the STDOUT of a program goes into the STDIN of another. 

`cat sports.txt 1> new_sports.txt` writes STDOUT to a new file. 

`ARGV` is the array of all arguments given to the Bash script. The first argument is `$1`, the second is `$2` etc. `$@` and `$*` give all the arguments. `$#` gives the number of arguments. 

```bash
#!/usr/bash
echo $1
echo $2
echo $@
echo "There are " $# "arguments"
```

Running `bash args.sh one two three four five` will result in: 

```
one
two 
one two three four five
There are 5 arguments
```

Define variables like this: `fistname='Cyntha'` and access them like this: `$firstname`. 

* Single ticks: Bash interprets what is between them literally
* Double ticks: Bash interprets literally except `$` and backticks
* Backticks: creates a shell within a shell. The content is sent into a new shell and the STDOUT goes into a variable. 

```bash
#illustrating shell within a shell
#date is a shell command, backticks run the command and return the result
rightnow_doublequote="The date is `date`."
echo $rightnow_doublequote
```

To invoke a shell within a shell you can also do: 

```bash
right_now_parantheses="The date is $(date)."
```

Numbers are not natively supported in Bash. We can use `expr 1 + 4` to do calculations. It only handles integers. You can start `bc` to do calculations. 

```bash
#pipe to bc
echo "5 + 7.5" | bc 
>> 12.5
#define decimal places
echo "scale=3; 10 / 3" | bc
>> 3.333
```

```bash
#using shell within a shell for calculations
model1=87.65
model2=89.20
echo "The total score is $(echo "$model1 + $model2" | bc)"
echo "The average score is $(echo "($model1 + $model2) / 2" | bc)"
```

A script to convert from Fahrenheit: 

```bash
# Get first ARGV into variable
temp_f=$1

# Subtract 32
temp_f2=$(echo "scale=2; $temp_f - 32" | bc)

# Multiply by 5/9 and print
temp_c=$(echo "scale=2; $temp_f2 * 5 / 9" | bc)

# Print the celsius temp
echo $temp_c
```

This script uses shell in shell to load the contents of files in variables: 

```bash
# Create three variables from the temp data files' contents
temp_a=$(cat temps/region_A)
temp_b=$(cat temps/region_B)
temp_c=$(cat temps/region_C)

# Print out the three variables
echo "The three temperatures were $temp_a, $temp_b, and $temp_c"
```

Two types of arrays in Bash:

* 'Normal': numerical-indexed structure. Like a Python list. 
* Associative: key-value pairs. Like a dict in Python. 

```bash
#numerical-indexed array
my_first_array=(1 2 3)
#return all elements
echo ${my_array[@]}
#return the array length
echo ${#my_array[@]}
#change third element
my_first_array[2]=99
#slice array
array[@]:N:M
#append
array+=(elements)
```

```bash
#create an associative array
declare -A city_details
city_details=([city_name]="New York" [population]=14)
echo ${city_details[city_name]}
#return all the keys
echo ${!city_details[@]}
```

```bash
# Create variables from the temperature data files
temp_b="$(cat temps/region_B)"
temp_c="$(cat temps/region_C)"

# Create an array with these variables as elements
region_temps=($temp_b $temp_c)

# Call an external program to get average temperature
average_temp=$(echo "scale=2; (${region_temps[0]} + ${region_temps[1]}) / 2" | bc)

# Append average temp to the array
region_temps+=($average_temp)

# Print out the whole array
echo ${region_temps[@]}
```

IF statements: 

```bash
x="Queen"
if [ $x == "King"]; then
	echo "$x is a King!"
else
	echo "$x is not a King!"
fi
```

```bash
x=10
if (($x > 5)); then
	echo "$x is more than 5!"
fi
#alternative with greater than
if [ $x -gt 5 ]; then
	echo "$x is more than 5!"
fi
#testing for multiple conditions
x=10
if [[ $x -gt 5 && $x -lt 11]]; then
	echo "$x is more than 5 and less than 11!"
fi
#conditional with command-line program
if grep -q Hello words.txt; then
	echo "Hello is inside!"
fi
```

There are plenty more flags to use in Bash conditional expressions. 

