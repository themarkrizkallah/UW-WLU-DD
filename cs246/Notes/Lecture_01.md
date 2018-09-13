# CS 246 Lecture 1

### Variables

- `x=1` no spaces
- if you want to use `echo x` you need to put `echo $x` to fetch the value of the variable
- you dont use it when you're setting the variable
##### good practice
- when you fetch a variable use `${x}` so that you know where the variable ends e.g. `echo ${x} y`

- every variable has type *String*

EG:

```java
dir = ~/cs246
echo ${dir}
/u/...
ls ${dir}
(contents of dir)

```

- There are some global variables available
- most are irrelevant

##### *Important - PATH*
- when you typre a command shell searches these directories in order for a program of that name

```java
echo "${PATH}"
(the path)
echo '${PATH}'
${PATH}
```
- $ expansion happens in " not '

#### Special Variables

$1, $2, $3 etc.. - Command line args

eg: Check whether a word is in the dictionary

```java
./isItAWord hello

#!/bin/bash

egrep "^$1$" /usr/share/dict/words
```

- *^...$ finds only exactly that word*

Eg: a good password should not be in the dictionary

- answer whether a word is a good password.

```java
#!/bin/bash

 egrep "^$1$" /usr/share/dict/words > /dev/null 
 
```

/dev/null is a black hole that supresses output

- every program returns a status code when finished
- egrep will return 0 if the pattern is found and 1 if the pattern is not found
- (In Linux) 0 means success and non-zero means failure
- `$?` is the status of the most recently executed command

### Conditionals

```java
if [ $? -eq 0 ]; then
 echo Bad password
else
 echo Maybe a good password
fi
```

or

```
if egrep "^$1$" /usr/share/dict/words; then
 
fi
```

- This program could be used wrong
- Varify the # of arguments

```java
#!/bin/bash
usage(){
  echo "usage: $0 password" >&2
}

if [ $# -ne 1 ]; then
 usage
 exit 1
fi
```
- 2 means standard error
- `$0` is the name of the script

### Loops

- print the numbers from `1` to `$1`

```
#!/bin/bash

x=1

while [ $x -le $1 ]; do

 echo $x
 x = $((x+1))
 
done
```

- `$((...))` is for arithmetic

#### Looping over a list
- eg rename all .cpp files to .cc

```
#!/bin/bash

for name in *.cpp; do
 mv ${name} ${name%cpp}cc
```
- `${name%cpp}` removes trailing cpp

- How many times does the word `$1` occur in the file `$2`

```
x=0
for word in $(cat "$2"); do
 if [ $word = " $1" ]; then
   x = $((x+1))
 fi
 done
 
 echo $x
 
```

- `$(...)` runs command putting in `"$2"`

eg: Payday is the last day of the month, when is this mon'ths payday?

2 tasks
- compute payday
- report answer

```
#!/bin/bash

answer(){
 if [ $1 -eq 31 ]; then
  echo "This month: the 31st"
 else
  echo "This month: the ${1}th"
fi
  
}

answer $(cal | awk '${print $6}'| egrep "[0-9]" | tail -1)

```

- now generalize to any month
- you can use `cal October 2017`
- let ./payday October 2017 give that payday

```
answer $(cal $1 $2 | awk '${print $6}'| egrep "[0-9]" | tail -1)
```

- `$1` and `$2` will be empty so it will actually work

```
if [ $2 ]; then
 prefix="This month"
else
 prefix=$2
fi

if [ $1 -eq 31 ]; then
 echo "${prefix}: the 31st"
 
```

#### *This concludes the topic of the shell*
