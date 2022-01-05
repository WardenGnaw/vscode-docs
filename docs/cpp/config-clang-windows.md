---
Order: 5
Area: cpp
TOCTitle: Clang on Windows
ContentId: 7efec972-6556-4526-8aa8-c73b3319d612
PageTitle: Get Started with C++ and llvm-mingw in Visual Studio Code
DateApproved: 7/15/2021
MetaDescription: Configuring the C++ extension in Visual Studio Code to target clang and lldb-mi on a llvm-mingw installation
---
# Using Clang with LLVM-MinGW

In this tutorial, you configure Visual Studio Code to use the Clang C++ compiler and LLDB-MI debugger from [lldb-mingw](https://github.com/mstorsjo/llvm-mingw) to create programs that run on Windows.

After configuring VS Code, you will compile and debug a simple Hello World program in VS Code. This tutorial does not teach you about Clang, LLVM, LLDB, lldb-mingw, or the C++ language. For those subjects, there are many good resources available on the Web.

If you have any problems, feel free to file an issue for this tutorial in the [VS Code documentation repository](https://github.com/microsoft/vscode-docs/issues).

## Prerequisites

To successfully complete this tutorial, you must do the following steps:

1. Install [Visual Studio Code](/download).

1. Install the [C/C++ extension for VS Code](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools). You can install the C/C++ extension by searching for 'c++' in the Extensions view (`kb(workbench.view.extensions)`).

    ![C/C++ extension](images/cpp/cpp-extension.png)

1. Get the latest version of llvm-mingw via [GitHub Releases](https://github.com/mstorsjo/llvm-mingw/releases), which provides up-to-date native builds of Clang, LLVM, LLDB, and other helpful C++ tools and libraries.  [Click here](https://github.com/mstorsjo/llvm-mingw/releases). Then download the `llvm-mingw-*-ucrt-x86_64.zip`.

1. Add the path to your lldb-mingw `bin` folder to the Windows `PATH` environment variable by using the following steps:
   1. In the Windows search bar, type 'settings' to open your Windows Settings.
   1. Search for **Edit environment variables for your account**.
   1. Choose the `Path` variable and then select **Edit**.
   1. Select **New** and add the lldb-mingw destination folder path to the system path. The exact path depends on where you have extracted the zip.
   1. Select **OK** to save the updated PATH. You will need to reopen any console windows for the new PATH location to be available.

### Check your lldb-mingw installation

To check that your lldb-mingw tools are correctly installed and available, open a **new** Command Prompt and type:

```bash
clang --version
lldb-mi --version
```

If you don't see the expected output or `clang` or `lldb-mi` is not a recognized command, make sure your PATH entry matches the lldb-mingw binary location where the compilers are located. If the compilers do not exist at that PATH entry, make sure you followed the instructions above.

## Create Hello World

From a Windows command prompt, create an empty folder called `projects` where you can place all your VS Code projects. Then create a sub-folder called `helloworld`, navigate into it, and open VS Code in that folder by entering the following commands:

```cmd
mkdir projects
cd projects
mkdir helloworld
cd helloworld
code .
```

The "code ." command opens VS Code in the current working folder, which becomes your "workspace". As you go through the tutorial, you will see three files created in a `.vscode` folder in the workspace:

- `tasks.json` (build instructions)
- `launch.json` (debugger settings)
- `c_cpp_properties.json` (compiler path and IntelliSense settings)

### Add a source code file

In the File Explorer title bar, select the **New File** button and name the file `helloworld.cpp`.

![New File title bar button](images/mingw/new-file-button.png)

### Add hello world source code

Now paste in this source code:

```cpp
#include <iostream>
#include <vector>
#include <string>

using namespace std;

int main()
{
    vector<string> msg {"Hello", "C++", "World", "from", "VS Code", "and the C++ extension!"};

    for (const string& word : msg)
    {
        cout << word << " ";
    }
    cout << endl;
}
```

Now press `kb(workbench.action.files.save)` to save the file. Notice how the file you just added appears in the **File Explorer** view (`kb(workbench.view.explorer)`) in the side bar of VS Code:

![File Explorer](images/mingw/file-explorer-mingw.png)

You can also enable [Auto Save](/docs/editor/codebasics.md#saveauto-save) to automatically save your file changes, by checking **Auto Save** in the main **File** menu.

The Activity Bar on the far left lets you open different views such as **Search**, **Source Control**, and **Run**. You'll look at the **Run** view later in this tutorial. You can find out more about the other views in the VS Code [User Interface documentation](/docs/getstarted/userinterface.md).

>**Note**: When you save or open a C++ file, you may see a notification from the C/C++ extension about the availability of an Insiders version, which lets you test new features and fixes. You can ignore this notification by selecting the `X` (**Clear Notification**).

## Explore IntelliSense

In your new `helloworld.cpp` file, hover over `vector` or `string` to see type information. After the declaration of the `msg` variable, start typing `msg.` as you would when calling a member function. You should immediately see a completion list that shows all the member functions, and a window that shows the type information for the `msg` object:

![Statement completion IntelliSense](images/wsl/msg-intellisense.png)

You can press the `kbstyle(Tab)` key to insert the selected member; then, when you add the opening parenthesis, you will see information about any arguments that the function requires.

## Build helloworld.cpp

Next, you'll create a `tasks.json` file to tell VS Code how to build (compile) the program. This task will invoke the clang compiler to create an executable file based on the source code.

Create a `tasks.json` file in a `.vscode` folder and open it in the editor.

Your will want to create a `tasks.json` file that is similar to the JSON below:

```json
{
    "tasks": [
        {
            "type": "cppbuild",
            "label": "C/C++: clang++.exe build active file",
            "command": "C:\\llvm-mingw-20211002-ucrt-x86_64\\bin\\clang++.exe",
            "args": [
                "-g",
                "${file}",
                "-o",
                "${fileDirname}\\${fileBasenameNoExtension}.exe"
            ],
            "options": {
                "cwd": "${fileDirname}"
            },
            "problemMatcher": [
                "$gcc"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "detail": "compiler: C:\\llvm-mingw-20211002-ucrt-x86_64\\bin\\clang++.exe"
        }
    ],
    "version": "2.0.0"
}
```

The `command` setting specifies the program to run; in this case that is clang. The `args` array specifies the command-line arguments that will be passed to g++. These arguments must be specified in the order expected by the compiler. This task tells g++ to take the active file (`${file}`), compile it, and create an executable file in the current directory (`${fileDirname}`) with the same name as the active file but with the `.exe` extension (`${fileBasenameNoExtension}.exe`), resulting in `helloworld.exe` for our example.

>**Note**: You can learn more about `tasks.json` variables in the [variables reference](/docs/editor/variables-reference.md).

The `label` value is what you will see in the tasks list; you can name this whatever you like.

The `"isDefault": true` value in the `group` object specifies that this task will be run when you press `kb(workbench.action.tasks.build)`. This property is for convenience only; if you set it to false, you can still run it from the Terminal menu with **Tasks: Run Build Task**.

### Running the build

1. Go back to `helloworld.cpp`. Your task builds the active file and you want to build `helloworld.cpp`.
1. To run the build task defined in `tasks.json`, press `kb(workbench.action.tasks.build)` or from the **Terminal** main menu choose **Run Build Task**.
1. When the task starts, you should see the Integrated Terminal panel appear below the source code editor. After the task completes, the terminal shows output from the compiler that indicates whether the build succeeded or failed. For a successful g++ build, the output looks something like this:

   ![G++ build output in terminal](images/mingw/mingw-build-output.png)

1. Create a new terminal using the **+** button and you'll have a new terminal with the `helloworld` folder as the working directory. Run `dir` and you should now see the executable `helloworld.exe`.

    ![Hello World in PowerShell terminal](images/mingw/mingw-dir-output.png)

1. You can run `helloworld` in the terminal by typing `helloworld.exe` (or `.\helloworld.exe` if you use a PowerShell terminal).

>**Note**: You might need to press `kbstyle(Enter)` a couple of times initially to see the PowerShell prompt in the terminal. This issue should be fixed in a future release of Windows.

### Modifying tasks.json

You can modify your `tasks.json` to build multiple C++ files by using an argument like `"${workspaceFolder}\\*.cpp"` instead of `${file}`. This will build all `.cpp` files in your current folder. You can also modify the output filename by replacing `"${fileDirname}\\${fileBasenameNoExtension}.exe"` with a hard-coded filename (for example `"${workspaceFolder}\\myProgram.exe"`).

## Debug helloworld.cpp

Next, you'll create a `launch.json` file to configure VS Code to launch the LLDB debugger when you press `kb(workbench.action.debug.start)` to debug the program.

1. From the main menu, choose **Run** > **Add Configuration...** and then choose **C++ (GDB/LLDB)**.
1. You'll then see a dropdown for various predefined debugging configurations. Choose **g++.exe build and debug active file**.

![C++ debug configuration dropdown](images/mingw/build-and-debug-active-file.png)

VS Code creates a `launch.json` file, opens it in the editor, and builds and runs 'helloworld'.

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "clang.exe - Build and debug active file",
            "type": "cppdbg",
            "request": "launch",
            "program": "${fileDirname}\\${fileBasenameNoExtension}.exe",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${fileDirname}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "lldb",
            "miDebuggerPath": "C:\\llvm-mingw-20211002-ucrt-x86_64\\bin\\lldb-mi.exe",
            "preLaunchTask": "C/C++: clang.exe build active file"
        }
    ]
}
```

The `program` setting specifies the program you want to debug. Here it is set to the active file folder `${fileDirname}` and active filename with the `.exe` extension `${fileBasenameNoExtension}.exe`, which if `helloworld.cpp` is the active file will be `helloworld.exe`.

By default, the C++ extension won't add any breakpoints to your source code and the `stopAtEntry` value is set to `false`.

Change the `stopAtEntry` value to `true` to cause the debugger to stop on the `main` method when you start debugging.

>**Note**: The `preLaunchTask` setting is used to specify task to be executed before launch. Make sure it is consistent with the `tasks.json` file `label` setting.

### Start a debugging session

1. Go back to `helloworld.cpp` so that it is the active file.
2. Press `kb(workbench.action.debug.start)` or from the main menu choose **Run > Start Debugging**. Before you start stepping through the source code, let's take a moment to notice several changes in the user interface:

- The Integrated Terminal appears at the bottom of the source code editor. In the **Debug Output** tab, you see output that indicates the debugger is up and running.
- The editor highlights the first statement in the `main` method. This is a breakpoint that the C++ extension automatically sets for you:

   ![Initial breakpoint](images/mingw/stopAtEntry.png)

- The Run view on the left shows debugging information. You'll see an example later in the tutorial.

- At the top of the code editor, a debugging control panel appears. You can move this around the screen by grabbing the dots on the left side.

## Next steps

- Explore the [VS Code User Guide](/docs/editor/codebasics.md).
- Review the [Overview of the C++ extension](/docs/languages/cpp.md).
- Create a new workspace, copy your `.vscode` JSON files to it, adjust the necessary settings for the new workspace path, program name, and so on, and start coding!
