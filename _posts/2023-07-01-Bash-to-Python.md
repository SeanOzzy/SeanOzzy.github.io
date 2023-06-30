---
layout: post
published: False
title: Migrating from Bash to Python3 - An Expert Guide
---

## Introduction
If you find yourself in a role where operational and on-call is involved a key objective is not to build complex software systems, but rather to automate tasks and streamline operations. By doing this, I aim to significantly reduce TOIL (Toil is work tied to running a service that tends to be manual, repetitive, automatable, tactical, devoid of enduring value, and that scales linearly as a service grows), as well as minimize human error. Over time, I have found that writing in code has proven to be a valuable approach to not only simplify task execution but also provide a clearer and more reproducible method of documenting software defects or test plans. Traditional documentation methods such as screenshots and step-by-step instructions can often lead to misunderstandings or mistakes due to human error, but code serves as a precise and unambiguous representation of each step in the process. You can run a set of tasks check the code in and someone else can repeat exactly what you did months or years later.

## Bash vs Python:

Choosing the right tool for the job is an important decision, especially when it comes to scripting and automation. Bash and Python are two popular choices for such tasks, each with their own pros and cons. Bash, the default shell for Linux-based systems, is a powerful tool for file manipulation and system commands. Its syntax is tailored for concise file and process operations, making it a natural choice for simple scripts. However, Bash has limitations, especially when it comes to more complex operations or when working with data structures other than plain text.

On the other hand, Python is a fully fledged programming language with a wide range of capabilities. It has robust support for different data types, libraries for tasks ranging from web development to machine learning, and a syntax that is consistent and easy to read. However, Python is heavier than Bash and can be overkill for simple tasks. It may also not be available on every system by default, unlike Bash.

It's important to consider the nature and complexity of the task at hand, the systems the script will run on, and the maintainability of the code when deciding between Bash and Python.

## Guide Introduction:

This guide is intended for experienced Bash scripters who are looking to expand their toolset by learning Python. The guide will provide an overview of how to accomplish common scripting tasks in Python that a Bash user would be familiar with, from reading input and printing output, to manipulating text, controlling flow with conditions and loops, defining functions, and handling command-line arguments. By leveraging your existing Bash knowledge and understanding the corresponding Python constructs, you'll be able to quickly get up to speed with Python and appreciate its features. This guide will ultimately help you become a more versatile and effective programmer, able to choose the right tool for your tasks and write scripts that are more maintainable and robust.

### 1. Simple text input and setting default values

In Bash, you can read input using `read -p`. In Python, you can use `input()` for the same purpose.
Python's built-in [`input`](https://docs.python.org/3/library/functions.html#input) function

```bash
# Bash
read -p "Enter your name: " name
```
```python
# Python
name = input("Enter your name: ")
```
Default values can be set in Python like so:

```python
name = input("Enter your name: ") or "Little Bobby Tables"
print (f'{name')
```

#### 1b. Outputting text to the terminal

Output in Python can be achieved with `print()` function. 
Python's built-in [`print`](https://docs.python.org/3/library/functions.html#print) function

```bash
# Bash
echo "Hello, world!"
```
```python
# Python
print("Hello, world!")
```

#### 1c. Sending output to a log file

In Bash, you might use `tee` to output to both the console and a file. In Python, you can achieve similar functionality with logging.
Python's [`logging`](https://docs.python.org/3/library/logging.html) module

```bash
# Bash
echo "Hello, world!" | tee output.log
```
```python
# Python
import logging

logging.basicConfig(filename='output.log', level=logging.DEBUG)
logging.info("Hello, world!")
print("Hello, world!")
```

### 2. Text manipulation in Python

In Bash, you might use `awk` and `sed` for text manipulation. Python has powerful string manipulation capabilities, and regex (via the `re` module) for more complex patterns.
Python's [`string`](https://docs.python.org/3/library/string.html) methods and [`re`](https://docs.python.org/3/library/re.html) module for regular expressions

```bash
# Bash - replacing text with sed
echo "Hello, world!" | sed 's/world/people/'
```
```python
# Python
print("Hello, world!".replace("world", "people"))
```

### 3. Simple if-then-else statements

Python uses `if`, `elif`, and `else` keywords, and doesn't require brackets.
Python's [`if` statements](https://docs.python.org/3/tutorial/controlflow.html#if-statements)

```bash
# Bash
if [ $a -eq $b ]; then
   echo "a is equal to b"
fi
```
```python
# Python
if a == b:
    print("a is equal to b")
```

### 4. More complex if-then-elif statements
Python's [`if` statements](https://docs.python.org/3/tutorial/controlflow.html#if-statements) and [`elif` keyword](https://docs.python.org/3/tutorial/controlflow.html#if-statements)

```bash
# Bash
if [ $a -eq $b ]; then
   echo "a is equal to b"
elif [ $a -gt $b ]; then
   echo "a is greater than b"
else
   echo "a is less than b"
fi
```
```python
# Python
if a == b:
    print("a is equal to b")
elif a > b:
    print("a is greater than b")
else:
    print("a is less than b")
```

### 5. For-do loops

In Python, `for` loops iterate over lists, ranges, or other iterable objects.
Python's [`for` loops](https://docs.python.org/3/tutorial/controlflow.html#for-statements)

```bash
# Bash
for i in $(seq 1 5); do
   echo "This is number $i"
done
```
```python
# Python
for i in range(1, 6):
    print(f"This is number {i}")
```

### 6. While-do loops

Python also supports `while` loops.
Python's [`while` loops](https://docs.python.org/3/tutorial/introduction.html#first-steps-towards-programming)

```bash
# Bash
i=1
while [ $i -le 5 ]
do
   echo "This is number $i"
   ((i++))
done
```
```python
# Python
i = 1
while i <= 5:
    print(f"This is number {i}")
    i += 1
```

### 7. Case statements 

Python doesn't have `case` statements, but can use a dictionary to map cases to functions, or `if-elif-else` chains for simple cases.
Using [`if-elif-else` chains](https://docs.python.org/3/tutorial/controlflow.html#if-statements) and [`dictionary`](https://docs.python.org/3/tutorial/datastructures.html#dictionaries) for simulating case statements

```bash
# Bash
case "$variable" in
    case1) command1;;
    case2) command2;;
    *) default_command;;
esac
```
```python
# Python
if variable == "case1":
    command1()
elif variable == "case2":
    command2()
else:
    default_command()
```

Python can simulate `case` statements using a dictionary:

```python
def case1():
    return "Case 1 function"

def case2():
    return "Case 2 function"

def default_case():
    return "Default function"

cases = {
    "case1": case1,
    "case2": case2
}

variable = "case1"  # This could be any string
case_function = cases.get(variable, default_case)
print(case_function())
```

### 8. Creating and calling functions

Python uses `def` to define a function.
Python's [`def` keyword](https://docs.python.org/3/tutorial/controlflow.html#defining-functions) for defining functions

```bash
# Bash
function greet {
   echo "Hello, $1"
}

greet "World"
```
```python
# Python
def greet(name):
    print(f"Hello, {name}")

greet("World")
```

### 9. Command-line parameters

In Bash, you can use `$1`, `$2`, etc., to refer to command-line arguments. Python has several ways to handle command-line arguments, with `argparse` module being the most powerful.
Python's [`argparse`](https://docs.python.org/3/library/argparse.html) module for command-line option and argument parsing

```bash
# Bash
echo "Hello, $1"
```
```python
# Python
import argparse

parser = argparse.ArgumentParser(description='Process some integers.')
parser.add_argument('names', metavar='N', type=str, nargs='+',
                    help='Names for the greeting')

args = parser.parse_args()
for name in args.names:
    print(f"Hello, {name}")
```

### 10. Required arguments and default values for optional arguments

Python function arguments can be required or optional. Optional arguments are given default values:
Python's [function definitions](https://docs.python.org/3/tutorial/controlflow.html#defining-functions) and [`argparse`](https://docs.python.org/3/library/argparse.html#required) module


```python
# Python
def greet(name, greeting="Hello"):
    print(f"{greeting}, {name}")

greet("World")  # Uses default greeting
greet("World", "Hi")  # Overrides default greeting
```

In command-line scripts, `argparse` can handle required and optional arguments:

```python
# Python
import argparse

parser = argparse.ArgumentParser()
parser.add_argument("name", help="Name for the greeting")
parser.add_argument("--greeting", default="Hello", help="Optional greeting")

args = parser.parse_args()
print(f"{args.greeting}, {args.name}")
```

### 11. Passing values between functions

Python functions can return values that are then passed to other functions.
Python's [function call](https://docs.python.org/3/tutorial/controlflow.html#defining-functions) and [return statement](https://docs.python.org/3/reference/simple_stmts.html#the-return-statement)

```python
# Python
def gather_data():
    return "World"

def greet(name):
    print(f"Hello, {name}")

data = gather_data()
greet(data)
```

### 12. Exception and error handling

[Python Official Documentation on Errors and Exceptions](https://docs.python.org/3/tutorial/errors.html)

```
import logging

logging.basicConfig(filename='app.log', level=logging.DEBUG, format='%(asctime)s - %(levelname)s - %(message)s')

def divide_numbers(numerator, denominator):
    try:
        result = numerator / denominator
    except ZeroDivisionError:
        logging.error("Error: You tried to divide by zero.")
        result = None
    except Exception as e:
        logging.error(f"Error: An unexpected error occurred: {str(e)}")
        result = None

    return result

print(divide_numbers(10, 2))  # Outputs: 5.0
print(divide_numbers(10, 0))  # Writes: Error: You tried to divide by zero. to app.log
print(divide_numbers(10, "two"))  # Writes: Error: An unexpected error occurred: unsupported operand type(s) for /: 'int' and 'str' to app.log
```

### 13. Lucky 13! The ```__name__ == "__main__"``` string

In Python, if __name__ == "__main__": is a common idiom for specifying the entry point of a script. When you execute a Python script directly, the special variable __name__ is set to "__main__". So, if __name__ == "__main__": is a way of checking if the script is being run directly, rather than being imported as a module by another script.

In the context of the script, main() is a function that is called if and only if the script is being run directly. If the script were imported as a module in another script, if __name__ == "__main__": would evaluate to False and main() would not be called.

For example, we have a simple script called greet.py.

```
# greet.py

def greet(name):
    print(f"Hello, {name}!")

if __name__ == "__main__":
    greet("World")
```

Now, if you were to execute this script directly from the command line with the command python greet.py, you would see the output Hello, World!. This is because __name__ is set to "__main__" when the script is run directly, so the if __name__ == "__main__": condition is True, and the greet function is called.

However, if you were to import this script as a module in another script called hello.py, the greet function would not be called automatically. Here's an example:

```
# hello.py

import greet

greet.greet("Alice")
```

When you run main.py with python hello.py, you'll see Hello, Alice! as the output. This is because when greet.py is imported, __name__ is set to the name of the module (in this case, "greet"), not "__main__", so the if __name__ == "__main__": condition is False and the greet function is not automatically called. But you can still call the greet function manually, as done in main.py.

So to summarize, if __name__ == "__main__": main() is used to specify a block of code that should be executed when the script is run directly, but not when the script's functions are imported and used in another script. It's a good practice to use this idiom to keep the top-level of your script tidy and to make it clear what happens when the script is executed.
