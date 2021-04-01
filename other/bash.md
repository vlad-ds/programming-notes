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

FOR loop:

```bash
#for loop
for x in 1 2 3
do
	echo $x
done
#brace expansion
for x in {1..5..2}
do
	echo $x
done
#three expression syntax
for((x=2;x<=4;x+=2))
do
	echo $x
done
#glob expansion
for book in books/*
do
	echo $book
done
#shell within a shell
for book in $(ls books/ | grep -i 'air')
do
	echo $book
done
```

While loops:

```bash
x=1
while [ $x -le 3 ]
do
	echo $x
	((x+=1))
done
```

Here's a script that identifies Python files which contain the `RandomForestClassifier`:

```bash
# Create a FOR statement on files in directory
for file in robs_files/*.py
do  
    # Create IF statement using grep
    if grep -q 'RandomForestClassifier' $file ; then
        # Move wanted files to to_keep/ folder
        mv $file to_keep/
    fi
done
```

CASE statements:

```
case MATCHVAR in
  PATTERN1)
  COMMAND1;;
  PATTERN2)
  COMMAND2;;
  *)
  DEFAULT COMMAND;;
esac
```

Example:

```bash
# Create a CASE statement matching the first ARGV element
case $1 in
  # Match on all weekdays
  Monday|Tuesday|Wednesday|Thursday|Friday)
  echo "It is a Weekday!";;
  # Match on all weekend days
  Saturday|Sunday)
  echo "It is a Weekend!";;
  # Create a default
  *) 
  echo "Not a day!";;
esac
```

Find all tree models and move them in a folder, delete the others:

```bash
# Use a FOR loop for each file in 'model_out/'
for file in model_out/*
do
    # Create a CASE statement for each file's contents
    case $(cat $file) in
      # Match on tree and non-tree models
      *"Random Forest"*|*GBM*|*XGBoost*)
      mv $file tree_models/ ;;
      *KNN*|*Logistic*)
      rm $file ;;
      *)
      # Create a default
      DEFAULT COMMAND
      echo "Unknown model in $file" ;;
    esac
done
```

Functions: 

```bash
#define
function print_hello(){
	echo "Hello world!"
}
print_hello
```

```bash
#converting fahrenheit to celsius
temp_f=30
function convert_temp(){
	temp_c=$(echo "scale=2; ($tempf - 32) * 5 / 9" | bc)
	echo $temp_c
}
```

```bash
# Create function
function upload_to_cloud () {
  # Loop through files with glob expansion
  for file in output_dir/*results*
  do
    # Echo that they are being uploaded
    echo "Uploading $file to cloud"
  done
}

# Call the function
upload_to_cloud
```

In Bash, all vars are global by default. You can make a variable local like this:

```bash
local first_filename=$1
```

The `return` option in Bash is only used to determine if the function was a success (0) or a failure(1-255). To "return" vars, we can assign them to a global variable or echo the result and access the function with a shell within a shell. 

Using a function with shell-in-shell:

```bash
# Create a function 
function return_percentage () {

  # Calculate the percentage using bc
  percent=$(echo "scale=2; 100 * $1 / $2" | bc)

  # Return the calculated percentage
  echo $percent
}

# Call the function with 456 and 632 and echo the result
return_test=$(return_percentage 456 632)
echo "456 out of 632 as a percent is $return_test%"
```

Using a function with global var:

```bash
# Create a function
function get_number_wins () {

  # Filter aggregate results by argument
  win_stats=$(cat soccer_scores.csv | cut -d "," -f2 | egrep -v 'Winner'| sort | uniq -c | egrep "$1")

}

# Call the function with specified argument
get_number_wins "Etar"

# Print out the global variable
echo "The aggregated stats are: $win_stats"
```

Scheduling scripts with CRON: 

```bash
#run myscript every day at 1:05 am
5 1 * * * bash myscript.sh
#run at 2:15pm every Sunday
15 14 * * 7 bash myscript.sh
```

To schedule a script: 

* type `crontab -e` to edit your list of cronjobs
* write the cronjob on a blank line
* save and exit the editor
* check the job is there by running `crontab -l`



