# Automated TERMINAL editor and compiler 

---

---

During our 3rd semester we were introduced to CLI (Command Line Interface) editor, *nano*. In it we were to write our programs and execute them, doing which was a pretty tedious task. We had to create the source files, write the same basic initial lines of code, save it, run another command to compile it and another to execute it. Also when the number of programs piled up, our directories were clustered. A solution was need to tackle this problem. The solution had to have the following aspects:

- The solution should compile the source code after saving and exiting the editor.
- The solution should execute the runnable produced by the compiler or show the errors generated during compilation.
- The solution should keep each project under it’s own directory to avoid cluster of programs under one directory.
- The solution should generate the basic template of programming language used on first creation.
- (Bonus) The solution should add the current date and time stamp as comment in the program.

With those points in mind, Amitabh, a friend of mine, and I started the project. At first I went to him with the basic project which had the first two solution, and then gradually we started to add more functionalities to it.

We named our project **crun** (because it was made to handle only C and C++ programs). **Crun** is a bash script written for Linux environment that had the **nano** text editor and **gcc** and **gpp** compilers. To get the templates for first instance of the programs we had to create the templates that we’ll use and a default location from which **crun** will get from. We created a **template** directory at the root level under which we kept out templates, custom designed for our needs.

What **crun** does is, when executed with the program name you want to create, it check the directory, where you currently are in, whether the program exits or not. If it doesn’t, it creates a folder named after the name of the program and copies the required template to that directory with the name of the program you typed in. Then it appends the date and time stamp into the program's source code as a comment and opens up the file in the editor. After you have written your program, saved it and exited, **crun** removes the previous runnable, if there was any, and performs compilation in a cleared CLI window. If there was compilation error it would be shown or the code would be executed.

It is possible to keep **crun** in any directory and use it only under that directory. If however the user wishes to have it throughout the Linux environment, he or she can keep it under the bin directory. To enable execution of **crun**, the program should be given execution permission.

Codes to install files:
```shell
sudo chmod 755 /bin/crun
```

Location hierarchy of files (or 'where to copy your files to'):
- **crun:** /bin/crun
- **c template:** /template/ctemp.c
- **cpp template:** /template/cpptemp.cpp

How to execute: On the directory you want to create the project in run the following command and let the magic work
```shell
crun project_name.c
```

**crun** script:
```shell 
# group="crun" 
# title="crun.sh"

#!/bin/bash
name=$1
fol=${name%.*}
path=`pwd`
dstr="Date: `date`"
ctpath="/template/ctemp.c"
cpptpath="/template/cpptemp.cpp"
source=$path/$fol/$name
exe=$path/$fol/$fol

if [ ${name: -2} == ".c" ]
then
    if [ ! -f $source ]
    then
        mkdir $fol
        cp $ctpath $source
        sed "3i\ $dstr" "$source" -i
    fi
    nano $source -c
    rm $exe
    clear
    gcc $source -g -lm -o $exe
    ./$fol/$fol

elif [ ${name: -4} == ".cpp" ]
then
    if [ ! -f $source ]
    then
        mkdir $fol
        cp $cpptpath $source
        sed "3i\ $dstr" "$source" -i
    fi
    nano $source -c
    rm $exe
    clear
    g++ $source -g -lm -o $exe
    ./$fol/$fol
   
else
    echo "UNRECOGNIZED FORMAT!"
fi
```

Template Codes:
```c
/*
 group="Template"
 title="ctemp.c"
 Author: Spandan
*/

#include<stdioh>
#include<stdlib.h>
#define in(a,b) {\
                           printf("%s",a);\
                           scanf("%d",&amp;b);\
                           getchar();\
                       }

int main(){

    return 0;
}
```

```cpp 
/*
 group="Template"
 title="cpptemp.cpp"
 Author: Spandan
*/

#include<iostream>
using namespace std;

class Test{
    private:

    public:

};

int main(){

    return 0;
}
```

To get the project from GitHub follow this [link](https://github.com/SpandanBG/crun/)

---

Spandan Buragohain,
2017-04-05 17:36:28