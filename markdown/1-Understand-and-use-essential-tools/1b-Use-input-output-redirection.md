1.b Use input-output redirection (>, >>, |, 2>, etc.)
===

stdin, stdout, stderr
---

Under normal circumstances every Linux program has three streams opened when it starts; one for input; one for output; and one for printing diagnostic or error messages. These are typically attached to the user's terminal (see man tty(4)) but might instead refer to files or other devices, depending on what the parent process chose to set up.

+ stdin (0) - Keyboard
+ stdout (1) - Screen
+ stderr (2) - Device reserved for error output


Input Redirection
---

Redirection is used to redirect the stdout/stdin/stderr.

| Operator     | Redirect          |
| ------------ |:----------------- |
| `1>`, `>`    | stdout            |
| `1>>`, `>>`  | Append stdout     |
| `2>`,        | stderr            |
| `2>>`        | Append stderr     |
| * `2>&1`, `&>` | stderr and stdout |

\* *Here are two ways on how we would redirect stderr to stdout*

    command > output_file 2>&1
    #or
    command &> output_file

### Examples

Cat the contents of myfirstscript (same as `cat myfirstscript`)

    $ cat < myfirstscript

Create longlisting with the output of `ls -al myfirstscript`

    $ ls -al myfirstscript > longlisting

Copy the contents of myfirstscript to mynewscript

    $ cat < myfirstscript > mynewscript

Pipe
---

Pipes are used to give the output of a command as input to another command

    $ echo a b c | cut -f 2 -d ' '
    b




---
[⬅️ Back](1-Understand-and-use-essential-tools.md)
