---
title: "ROOT in C++ and Python"
teaching: 10
exercises: 5
questions:
- "Should I use ROOT in C++ or Python?"
objectives:
- "Run C++ code interactively with ROOT!"
- "Compile C++ programs using ROOT!"
- "Use ROOT in Python!"
keypoints:
- "The choice of interactive C++, compiled C++ or Python is based on the use case!"
- "Usage of C++ code, compiled with optimization flags, may save you hours of computing time!"
- "PyROOT lets you use C++ from Python but offers many more advanced features to speed up your analysis in Python. Details about the dynamic Python bindings provided by PyROOT can be found on [https://root.cern/manual/python](https://root.cern/manual/python/)."
---

> This section shows you the difference between using ROOT with interactive C++, compiled C++ and Python.
{: .testimonial}

## Interactive C++

One of the main features of ROOT is the possibility to use C++ interactively thanks to the C++ interpreter [Cling](https://root.cern/cling/). Cling lets you use C++ just like Python either from the prompt or in scripts.

### The ROOT prompt

By just typing `root -l` (the `-l` switch prevents the ROOT **l**ogo from popping up) in the terminal you will enter the ROOT prompt. Like the Python prompt, the ROOT prompt is well suited to fast investigations.

```bash
$ root -l
root [0] 1+1
(int) 2
root [1] .q
```

Note that you can exit by typing `.q` (a "dot" followed by a "q") and pressing enter.

Now, if you pass a file as argument to `root`, the file will be opened when entering the prompt and put in the variable `_file0`. ROOT typically comes with support for reading files remotely via HTTP (and [XRootD](https://xrootd.slac.stanford.edu/)), which we will use for the following example:

> ## No support for remote files?
> Although unlikely, your ROOT build may not be configured to support remote file access. In that case, in order to continue with the examples below, you can just download the file with `curl -O https://root.cern/files/tmva_class_example.root` and point to your local file. No other changes required!
{: .solution}

```bash
$ root -l https://root.cern/files/tmva_class_example.root

root [0]
Attaching file https://root.cern/files/tmva_class_example.root as _file0...
(TFile *) 0x555f82beca10

root [1] _file0->ls() // Show content of the file, all objects are accessible via the prompt!
TWebFile**              https://root.cern/files/tmva_class_example.root
 TWebFile*              https://root.cern/files/tmva_class_example.root
  KEY: TTree    TreeS;1 TreeS
  KEY: TTree    TreeB;1 TreeB

root [2] TreeS->GetEntries() // Number of events in the dataset

root [3] TreeS->Print() // Show dataset structure
******************************************************************************
*Tree    :TreeS     : TreeS                                                  *
*Entries :     6000 : Total =           98896 bytes  File  Size =      89768 *
*        :          : Tree compression factor =   1.00                       *
******************************************************************************
*Br    0 :var1      : var1/F                                                 *
*Entries :     6000 : Total  Size=      24641 bytes  One basket in memory    *
*Baskets :        0 : Basket Size=      32000 bytes  Compression=   1.00     *
*............................................................................*
*Br    1 :var2      : var2/F                                                 *
*Entries :     6000 : Total  Size=      24641 bytes  One basket in memory    *
*Baskets :        0 : Basket Size=      32000 bytes  Compression=   1.00     *
*............................................................................*
*Br    2 :var3      : var3/F                                                 *
*Entries :     6000 : Total  Size=      24641 bytes  One basket in memory    *
*Baskets :        0 : Basket Size=      32000 bytes  Compression=   1.00     *
*............................................................................*
*Br    3 :var4      : var4/F                                                 *
*Entries :     6000 : Total  Size=      24641 bytes  One basket in memory    *
*Baskets :        0 : Basket Size=      32000 bytes  Compression=   1.00     *
*............................................................................*

root [4] TreeS->Draw("var1") // Draw a histogram of the variable var1
```

<kbd>
<img src="../fig/trees_draw.png">
</kbd>

{% include links.md %}

Now exit (in the future, we will not repeat the exit statement as it will be understood from the context):

~~~
root [5] .q
~~~
{: .language-bash}


### ROOT scripts

A unique feature of ROOT is the possibility to use C++ scripts, also called "ROOT macros". A ROOT script contains valid C++ code and uses as entrypoint a function with the same name as the script. Let's take as example the file `myScript.C` with the following content.

```cpp
void myScript() {
    auto file = TFile::Open("https://root.cern/files/tmva_class_example.root");
    for (auto key : *file->GetListOfKeys()) {
        const auto name = key->GetName();
        const auto entries = ((TTree*)file->Get(name))->GetEntries();
        std::cout << name << " : " << entries << std::endl;
    }
    auto TreeS = (TTree*)file->Get("TreeS");    
    TCanvas c; TreeS->Draw("var1"); c.SaveAs("test.png");
}
```

Scripts can be processed by passing them as argument to the `root` executable:

```bash
$ root -l myScript.C

root [0]
Processing myScript.C...
TreeS : 6000
TreeB : 6000
Info in <TCanvas::Print>: file test.png has been created
```

The advantage of such scripts is the simple interaction with C++ libraries (such as ROOT) and running your code at C++ speed with the convenience of a script.

## Compiled C++

You can improve the runtime of your programs if you compile them upfront. Therefore, ROOT tries to make the compilation of ROOT macros as convenient as possible!

### ACLiC

ROOT provides a mechanism called ACLiC to compile the script in a shared library and call the compiled code from interactive C++, all automatically!

The only change required to our script is that we need to include all required headers:

```cpp
#include "TFile.h"
#include "TTree.h"
#include "TCanvas.h"
#include <iostream>

void myScript() {
    // The body of the myScript function goes here
}
```

Now, let's compile and run the script again. Note the `+` after the name which is a way to tell to root to use myScript.C but compile it! (Note that you do not have to actually change the name of the file itself to add the `+`, but only append that sign to the end of the `root` command.)

```bash
$ root -l myScript.C+

root [0]
Processing myScript.C+...
Info in <TUnixSystem::ACLiC>: creating shared library /path/to/myScript_C.so
TreeS : 6000
TreeB : 6000
Info in <TCanvas::Print>: file test.png has been created
```

ACLiC has many more features, for example compiling your program with debug symbols using `+g`. You can find the documentation [here](https://root.cern/manual/first_steps_with_root/#compiling-a-root-macro-with-aclic).

### C++ compilers

Of course, the C++ code can also just be compiled with C++ compilers such as `g++` or `clang++` with the advantage that you have full control of all compiler settings, most notable the optimization flags such as `-O3`!

To do so, we have to add the `main` function to the script, which is the default entrypoint for C(++) programs.

```cpp
#include "TFile.h"
#include "TTree.h"
#include "TCanvas.h"
#include <iostream>

void myScript() {
    // The body of the myScript function goes here
}

int main() {
    myScript();
    return 0;
}
```

Now, you can use the following command with your C++ compiler of choice to compile the script into an executable.

```bash
$ g++ -O3 -o myScript myScript.C $(root-config --cflags --libs)
$ ./myScript
TreeS : 6000
TreeB : 6000
Info in <TCanvas::Print>: file test.png has been created
```

Computationally heavy programs and long running analyses may benefit greatly from the optimized compilation with `-O3` and can save you hours of computing time!

## Python

ROOT provides the Python bindings called PyROOT. PyROOT is not just ROOT from Python, but a full-featured interface to call C++ libraries in a pythonic way. This lets you import the ROOT module from Python and makes all features dynamically available. Let's rewrite the C++ example from above and put the code in the file `myScript.py`!

```python
import ROOT

f = ROOT.TFile.Open('https://root.cern/files/tmva_class_example.root')
for key in f.GetListOfKeys():
    name = key.GetName()
    entries = f.Get(name).GetEntries()
    print('{} : {}'.format(name, entries))
TreeS = f.Get('TreeS')
c = ROOT.TCanvas()
TreeS.Draw('var1')
c.SaveAs('test.png')
```

Calling the Python script works as expected:

```bash
$ python myScript.py
TreeS : 6000
TreeB : 6000
Info in <TCanvas::Print>: file test.png has been created
```

But PyROOT can do much more for you than simply providing access to C++ libraries from Python. You can also inject efficient C++ code into your Python program to speed up potentially slow parts of your program!  You can insert the following code in another file, like `heavy.py`, for instance, and run with python; or run interactively by opening the python program in your terminal and entering the commands one by one (in the future, this will be assumed with the all the code snippets.)

```python
import ROOT

ROOT.gInterpreter.Declare('''
int my_heavy_computation(std::string x) {
    // heavy computation goes here
    return x.length();
}
''')

# Functions and object made available via the interpreter are accessible from
# the ROOT module
y = ROOT.my_heavy_computation("the ultimate answer to life and everything")
print(y) # Guess the result!
```

A guide to such advanced features of PyROOT can be found in the official manual at [https://root.cern/manual/python](https://root.cern/manual/python/). Feel free to investigate!

> ## Try using ROOT with interactive C++, compiled C++ and Python!
> Make yourself familiar with the different ways you can run an analysis with ROOT!
{: .challenge}
