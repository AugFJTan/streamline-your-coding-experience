# Streamline Your Coding Experience on Linux

Sometimes, it is just easier to work directly on a Linux terminal than go back and forth between an IDE and the terminal itself just to debug a programme. This guide is aimed at helping beginners (and maybe some intermediates) learn how to set up, configure and customize their coding experience without ever leaving the terminal.

Before we begin, you will require a few things:
* Some basic knowledge on writing and compiling in the programming language of your choice
* Some basic knowledge on writing bash scripts
* A reasonable text editor installed (Vim is used in this guide)

## Guide Outline

This guide is divided into 3 parts:
* Creating a `bin` directory
* Creating directories for source files
* Writing bash scripts

## Creating a 'bin' directory

Let's start by creating a directory called `bin` in your home directory (i.e. /home/your-username). Not to be confused with `/bin`, where the system bash scripts are located. This directory is where you will be saving the bash scripts you will be writing later in the guide. Access your terminal and create the directory with the command:

```shell
mkdir bin
```

Simple enough. If you open your file manager, you will find the directory alongside your standard `Documents`, `Pictures`, `Music`, etc. directories. It should look something like this:

![Bin Directory](/images/bindir.png)

Any bash scripts in the `bin` directory will be accessible in other directories, allowing you to run them without having to leave the directory you are currently on. This is especially useful because it will save you on **a lot** of typing. 

Let’s test out this `bin` directory, shall we? Write a simple script and save it in `bin`. Maybe something as simple as:

```shell
#!/bin/bash

echo "It works!"
```

Name it whatever you want. We'll name this script `bashtest`. Before you do anything, you need to change the permissions of the bash script so you can execute it. Open your terminal, go to `bin` and type:

```shell
chmod 755 test
```

Go to another directory and type `bashtest`. It should display "It works!" on the terminal: 

![It Works!](/images/itworks.png)

Now that's done, on to the next step!

## Creating directories for source files

Organization is key when you want to locate your files quickly. That's why arranging your source files based on programming language should be a priority. For example, having separate directories named `C`, `Cpp`, `Java`, etc. is much better than a directory filled with source files with different extensions. Granted, you could always use the search function on your file manager to find the files you need, but you wouldn't have to if you bothered to organize everything in the first place.

Within these directories there should be subdirectories that contain the actual source files and their executable. (This would be the case most of the time, unless there's *only* the source file itself and no associated executable, like in Python). Unless, of course, you don't mind having a messy C directory full of `.c`, `.h` and `.o` files mingling with one another. Usually, you would want to use the same name for the directory and the source file it contains. For example, a Java source file called `BookCatalogue.java` would be located under a directory called `BookCatalogue`.

If you haven't already made your files neat, I would advise doing so as arranging them based on programming language will be important when writing code for the bash scripts later.

Here's a look at my directory layout:

![Directory Layout](/images/dirlayout.png)

## Writing bash scripts

Now for the fun part. Automating processes on bash scripts can be really satisfying when you get what you want. In our case, we are going to write three separate bash scripts: one to simplify the editing of source files; another to simplify the compilation of said source files; and a final one to run the executable. You will be able to run all three bash scripts regardless of your current directory. 

### Writing an 'edit' script

Go to your `bin` directory that you created and create a bash script that opens your preferred text editor to edit your source files. You can name it whatever you want. Here, we'll call the script `edit`. Notice that any extension (i.e. `.sh`, `.bash`, etc.) is left out. This is to make the script appear like a command, albeit a user-defined one, when you access it in the terminal. If you are using Vim, type:

```
vim edit
```

On your text editor, write the bash script according to your needs. In the example below, the subdirectories (named after the programming languages) to the source files are located under the `Programming` subdirectory in the `Documents` directory. The subdirectories here are named `C`, `Cpp`, `Java` and `Python` respectively. 

```shell
#!/bin/bash
# Edit source file with Vim based on language and file name

# Jump to home directory and then go to subdirectory containing files
cd; cd "Documents/Programming/$1"	

if [ $1 != "Python" ]
then
	if [ -d $2 ]
	then
		cd $2
	else
		mkdir $2
		cd $2
	fi

	if [ $1 = "C" ]
	then
		EXT="c"
	elif [ $1 = "Cpp" ]
	then
		EXT="cpp"
	elif [ $1 = "Java" ]
	then
		EXT="java"
	fi
else 
	EXT="py"
fi

# Open file with Vim
vim $2.$EXT	

if [ ! -f $2.$EXT ] && [ $1 != "Python" ]
then 
	cd ..
	rmdir $2
fi

exit 0
```

Note that `$1` and `$2` are command line arguments. Depending on what you key in, you will be able to access a particular file in the subdirectory named after the programming language. For example, let’s say you wanted to edit a C file called `Hello.c`. Instead of going all the way to the directory where the file is located just to edit it, in your terminal simply type:

```
edit C Hello
```

… and now you have your file opened in your text editor (in this case, Vim)!

To break things down, here's what the example bash script does:
* Changes from your current directory to your home directory and then to the directory where your source files are located.
* If the language specified (the command line argument `$1`) isn't Python, check if the directory for the source file exists. If it doesn't, create a new directory and name it after what the user keyed in for command line argument `$2` and go into it (this is usually the case when you want to make a new file). If it does exist, go into the directory.
* Based on the language, declare a string variable called `EXT` and assign it with the file extension
* Open the text editor for the specified file.
* In case you decided not to create a new file, the directory created earlier will be removed.

Before we proceed, don't forget to change the permissions of the script with: 

```shell
chmod 755 edit
```

Go on and test the script. If there are any issues, just edit the code accordingly.

### Writing a 'compile' script

In the same `bin` directory, you want to create another bash script to enable you to compile your source file. Here, we’ll name the example script below `compile`.

```shell
#!/bin/bash
# Compile the source file based on language and file name

cd; cd "Documents/Programming/$1"

if [ $1 != "Python" ] 
then
	cd $2
	
	if [ -f compile.sh ]
	then
		 ./compile.sh 
		exit 0
	fi

	if [ $1 = "C" ]
	then	
		COMPILER="gcc"
		EXT="c"
	elif [ $1 = "Cpp" ] 
	then
		COMPILER="g++"
		EXT="cpp"
	elif [ $1 = "Java" ]
	then
		COMPILER="javac"
		EXT="java"
	fi
else 
	echo "No files to compile in '$1' directory."
	exit 0
fi

if [ $1 = "C" ] || [ $1 = "Cpp" ]
then
	$COMPILER $2.$EXT -o $2 && ./$2
else
	$COMPILER $2.$EXT && $EXT $2
fi

exit 0
```

Here's what’s going on under the hood:
* Using the same commands as we used in the `edit` script, we go to the directory containing the source files.
* If the language specified isn't Python, proceed to the directory. If it is Python, display a message to inform the user that no files need to be compiled (since Python can be debugged at runtime)
* Check if there is a special `compile.sh` script. If it exists, run the script. This bash script contains extra compilation commands than the usual `gcc main.c -o main`, just to make things flexible. The ‘compile.sh’ script may contain something like `gcc -std=c99 -o main main.c file1.c file2.c` to compile multiple files together with .o object files using the C99 standard.
* Declare and assign the values to two string variables for compiler and extension respectively based on language.
* Finally compile the source file with the specified compiler and run the executable so you can test it. The `&&` between the commands ensures that if the compilation fails (i.e. there are errors in your source code) the executable won’t run.

Let’s say we want to compile a C++ file called `Sample.cpp`. To execute the script, simply type:

```
compile Cpp Sample
```

If there are no compilation errors, the executable should run. Be sure to change the script's permissions and test the script. If you are happy with your `compile` script, it’s time to write the final `run` script.

### Writing a 'run' script

This script is by far the easiest to write. This script will enable you to run your executable file no matter your current directory. Again, in your `bin` directory, create a new batch script. This example will be called `run`:

```shell
#!/bin/bash
# Run executable based on language and file name

cd; cd "Documents/Programming/$1"

if [ $1 != "Python" ] 
then
	cd $2
	
	if [ $1 = "C" ] || [ $1 = "Cpp" ]
	then
		./$2
	else
		java $2
	fi
else
	python3 $2.py
fi

exit 0
```

How this works is relatively straightforward:
* Go to the directory where your executable files are located
* Based on the language, go to the subdirectory (if there is one) and run the executable

For running Python files, the Python3 interpreter is used in this example, but you could always opt for Python2. 

Anyway, say you wanted to run a Python file called ‘StudentList.py’. In order to *run* the `run` script, type:

```
run Python StudentList
```

Once again, change the permissions of the `run` script and test it. 

If you made it this far, congratulations! You now have three new batch scripts at your disposal to help you speed up your coding.

## Recap

The way you can run your scripts is simply to follow this format:

```
command language filename
```

where `command` is either `edit`, `compile` or `run`.

The example scripts provided were written based on my own personal coding needs. You are highly encouraged to edit the scripts to your liking. That being said, I hope you learned a thing or two on how bash scripts can make your coding experience faster and more enjoyable.

If you would like to add more to this repository, please take a look at the next section before making a pull request. Happy coding!

## Author’s Note:

Contributions are more than welcome for this repository. If you would like to contribute, you could make a pull request to specify a variation of the bash scripts that supports a certain language (PHP, Go, Assembly, etc.) or suggest ways to improve the existing bash scripts. It would be helpful if people had more examples to refer to.
