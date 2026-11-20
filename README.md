# Mini-Bash

**An interactive Unix command line interpreter and system automation toolkit, written in C, that makes cryptic UNIX commands easy to learn.**

> B.Tech Semester-3 | UNIX Lab (24B15CS214) | Jaypee Institute of Information Technology, Noida | Batch F-7

Mini-Bash is a real, working shell built on POSIX system calls (`fork`, `execvp`, `waitpid`, `pipe`, `dup2`, `open`, `chdir`). It translates beginner-friendly command names such as `whereami`, `showfiles` and `findtext` into the genuine UNIX utilities (`pwd`, `ls`, `grep`), so learners practise real UNIX behaviour while using words they can actually understand.

---

## Table of Contents

1. [The Problem](#the-problem)
2. [Our Solution](#our-solution)
3. [Features](#features)
4. [Quick Start](#quick-start)
5. [System Requirements](#system-requirements)
6. [Command Reference (Original vs Mini-Bash)](#command-reference-original-vs-mini-bash)
7. [Example Session](#example-session)
8. [How It Works](#how-it-works)
9. [What We Did and Why (Design Decisions)](#what-we-did-and-why-design-decisions)
10. [Mapping to UNIX Lab Outcomes](#mapping-to-unix-lab-outcomes)
11. [Known Limitations](#known-limitations)
12. [Future Scope](#future-scope)
13. [Repository Structure](#repository-structure)
14. [Documentation](#documentation)
15. [Team](#team)

---

## The Problem

UNIX is powerful, but its command vocabulary is hard for a beginner. Names like `pwd`, `ls`, `grep`, `ps`, `kill` and `chmod` are abbreviations from the 1970s whose purpose is not visible from the name:

- `grep` comes from the old `ed` command `g/re/p`.
- `awk` is made from the initials of its three authors.
- `touch` really updates timestamps; creating a file is a side effect.
- `kill` actually *sends signals*, not only terminates.

On top of the names, a beginner must also learn shell syntax (quoting, redirection, pipes, `&`), the idea of a process ID, and octal permission modes. One wrong `rm` can also destroy real work.

## Our Solution

Mini-Bash keeps the real UNIX utilities underneath but changes **how the learner talks to them**:

1. **Friendly command vocabulary:** self-describing names with a consistent verb-object pattern (`makefile`, `makefolder`, `removefolder`, `readfile`, `topview`, `bottomview`, ...), translated to the real utility before it runs.
2. **Safe `sandbox`:** one command creates a `playground/` folder with sample data and an automation script and moves the learner inside it, so experiments cannot harm personal files.
3. **Built-in tutor:** `guide` shows the full command menu and `explain <command>` says which UNIX command and system call stand behind each friendly command.
4. **Nothing is lost:** it is a real shell, so pipes, redirection, background jobs, `vi` editing, environment variables and `.sh` scripting all work.

## Features

- Friendly names for 30+ UNIX commands, with all flags passed straight through (`showfiles -l` is exactly `ls -l`)
- Standard UNIX names still work (typing `ls` or `grep` directly also runs them)
- Parent built-ins for commands that must change the shell itself: `goto`, `whereami`, `setvar`, `shortcut`, `sandbox`, `guide`, `explain`, `bye`
- Input (`<`) and output (`>`, `>>`) redirection using `open()` + `dup2()`
- Pipe (`cmd1 | cmd2`) using `pipe()` + `dup2()`, also combinable with redirection (`cmd1 | cmd2 > file`)
- Background jobs with `runbg <cmd>` or `<cmd> &`, reaped with `waitpid(..., WNOHANG)` so no zombie processes remain
- `goto -` returns to the previous directory
- `shortcut name=command` creates in-session aliases
- Quote-aware tokenizer (supports `sed` and `awk` programs with spaces)
- Friendly error message for unknown commands

## Quick Start

```bash
git clone https://github.com/<your-username>/mini-bash.git
cd mini-bash

# build
make                # or: gcc -Wall -Wextra minibash.c -o minibash

# run
./minibash
```

Then try:

```text
minibash> guide
minibash> sandbox
minibash> showfiles -l
minibash> explain vi
minibash> bye
```

Use `make clean` to remove the binary and the generated `playground/` folder.

## System Requirements

| Component | Requirement |
|---|---|
| Hardware | x86_64 PC or laptop, 2 GB RAM, 1 GB free disk space |
| Operating system | Ubuntu Linux 22.04 / 24.04 LTS (native) or Ubuntu on WSL 2. Any POSIX system should work |
| Compiler | GCC, built with `-Wall -Wextra` (GNU C99 or later) |
| Build tool | `make` (optional) |
| C headers | `<unistd.h>`, `<sys/wait.h>`, `<fcntl.h>`, `<sys/stat.h>`, `<sys/types.h>`, `<string.h>`, `<stdlib.h>`, `<ctype.h>`, `<stdio.h>` |
| Utilities used at run time | GNU Coreutils (`ls`, `cat`, `cp`, `mv`, `rm`, `head`, `tail`, `wc`, `sort`, `cut`, ...), `grep`, `awk`, `sed`, `find`, `ps`, `kill`, `vi`, `bash` |
| Terminal | Any VT100-compatible terminal (Ubuntu Terminal, Windows Terminal with WSL 2) |

Install the toolchain on Ubuntu / WSL:

```bash
sudo apt update
sudo apt install build-essential vim make
```

> **Windows users:** run Mini-Bash inside WSL 2 (it needs `fork()`, so it will not run natively on Windows).

## Command Reference (Original vs Mini-Bash)

### File system and directories

| Mini-Bash | UNIX equivalent | Description |
|---|---|---|
| `sandbox` | `mkdir` + `fopen` + `chdir` | Creates `playground/` with sample data and a script, then enters it |
| `whereami` | `pwd` (`getcwd()`) | Shows the current directory |
| `goto <dir>` | `cd` (`chdir()`) | Changes directory; supports `..` and `-` |
| `showfiles` | `ls` | Lists files and folders |
| `makefolder` | `mkdir` | Creates a folder |
| `removefolder` | `rmdir` | Removes an empty folder |
| `searchpath <path> <name>` | `find <path> -name <name>` | Searches for files by name |

### File handling and editing

| Mini-Bash | UNIX equivalent | Description |
|---|---|---|
| `makefile` | `touch` | Creates empty files |
| `readfile` | `cat` | Displays a file |
| `topview` | `head` | Shows the first lines |
| `bottomview` | `tail` | Shows the last lines |
| `pageview` | `less` | Views a file page by page |
| `vi <file>` | `vi` | Opens the real vi editor (deliberately not renamed) |
| `copyitem` | `cp` | Copies files and folders |
| `moveitem` | `mv` | Moves or renames |
| `deleteitem` | `rm` | Deletes |
| `setperms <mode> <file>` | `chmod` | Changes permissions |
| `countstats` | `wc` | Counts lines, words and bytes |

### Text processing, pipes and redirection

| Mini-Bash | UNIX equivalent | Description |
|---|---|---|
| `say` | `echo` | Prints text |
| `findtext` | `grep` | Searches for a pattern |
| `sortlines` | `sort` | Sorts lines |
| `pickfield` | `cut` | Extracts fields or columns |
| `textfilter` | `awk` | Pattern scanning and processing |
| `replaceword` | `sed` | Stream editing and substitution |
| `>` | `open(O_TRUNC)` + `dup2()` | Redirects output, overwriting the file |
| `>>` | `open(O_APPEND)` + `dup2()` | Redirects output, appending |
| `<` | `open(O_RDONLY)` + `dup2()` | Redirects input from a file |
| `\|` | `pipe()` + `dup2()` | Connects one process's output to another's input |

### Processes and system information

| Mini-Bash | UNIX equivalent | Description |
|---|---|---|
| `runningprocs` | `ps` | Lists processes and PIDs |
| `stopproc <pid>` | `kill` | Terminates a process |
| `priorityrun` | `nice` | Runs a command at lower priority |
| `runbg <cmd>` / `cmd &` | background `fork()` | Runs without blocking the shell |
| `clocknow` | `date` | Current date and time |
| `myuser` | `whoami` | Current user |
| `machinename` | `hostname` | Machine name |
| `activeusers` | `who` | Logged-in users |
| `manual` | `man` | Manual page |
| `printdoc` | `lpr` | Print a document |

### Scripting, automation and tutor

| Mini-Bash | UNIX equivalent | Description |
|---|---|---|
| `runscript <file.sh>` | `bash` | Runs a shell script |
| `shortcut <name>=<cmd>` | in-memory alias table | Creates a custom alias for the session |
| `setvar <VAR=val>` | `setenv()` / `export` | Sets an environment variable |
| `explain <cmd>` | built-in tutor | Explains how a command works |
| `guide` | built-in help | Shows the complete command menu |
| `bye` | `exit(0)` | Exits (`exit` also works) |

## Example Session

The automated student-record workflow uses `students.txt` and `auto_report.sh`, both created by `sandbox`.

```text
minibash> sandbox
[OK] Sandbox ready at 'playground/' with students.txt & auto_report.sh!
minibash> showfiles -l
total 8
-rwxr-xr-x 1 student student 144 Nov 20 10:12 auto_report.sh
-rw-r--r-- 1 student student  66 Nov 20 10:12 students.txt
minibash> findtext CSE students.txt | sortlines
101,Aarav,CSE,88
103,Rohan,CSE,76
minibash> pickfield -d, -f2,4 students.txt > scores.txt
minibash> say Report-Complete >> scores.txt
minibash> readfile scores.txt
Aarav,88
Diya,92
Rohan,76
Meera,95
Report-Complete
minibash> runbg sleep 20
[Background Job Started] PID: 4521
minibash> stopproc 4521
minibash> runscript auto_report.sh
--- Automated Student Summary ---
Total Records: 4
CSE Students:
101,Aarav,CSE,88
103,Rohan,CSE,76
minibash> explain vi
vi -> Launches the interactive vi editor. Press 'i' to type, 'Esc' for command mode, ':wq' to save & quit.
minibash> bye
Exiting Mini-Bash. Goodbye!
```

(PIDs, dates and file sizes will differ on your machine.)

## How It Works

Mini-Bash follows the classic UNIX layered design:

```text
Interactive User Interface   minibash> prompt, guide, explain
        |
Utilities + Mini-Bash        tokenizer, command mapper, built-ins, pipe/redirect engine
        |
Kernel + POSIX syscalls      fork, execvp, waitpid, pipe, dup2, open, chdir, getcwd
        |
Hardware
```

Every line typed at the prompt goes through this pipeline:

1. Reap finished background children with `waitpid(-1, NULL, WNOHANG)`, print the prompt and read a line.
2. If it is a **parent built-in** (`bye`, `guide`, `sandbox`, `whereami`, `goto`, `setvar`, `shortcut`), run it inside the shell process.
3. Detect a background request (`runbg` prefix or trailing `&`).
4. If the line contains `|`, split it, tokenize and translate both sides, create a `pipe()`, `fork()` two children and wait for both.
5. Otherwise tokenize; `explain` is answered by the shell itself.
6. `map_command()` expands any shortcut and translates the friendly name to the real utility (`searchpath` also rewrites its arguments to `find <path> -name <name>`).
7. `fork()`: the child applies redirection (`open()` + `dup2()`) and calls `execvp()`; the parent waits (foreground) or prints the PID (background).

### Why `goto` must run in the parent

The working directory belongs to a process. If `goto` ran in a forked child, the child would change *its own* directory and exit, leaving the shell where it started. So `goto` calls `chdir()` in the parent. The same reasoning applies to `setvar` (children cannot change the parent's environment) and `shortcut`.

### How the pipe and redirection work

- `pipe()` gives two descriptors: a write end and a read end of a kernel buffer.
- Child 1 does `dup2(write_end, STDOUT_FILENO)`; child 2 does `dup2(read_end, STDIN_FILENO)`.
- The parent closes both ends. If it did not, the reader would never see end-of-file.
- `>` opens with `O_WRONLY | O_CREAT | O_TRUNC`, `>>` with `O_APPEND`, `<` with `O_RDONLY`; then `dup2()` points descriptor 0 or 1 at the file before `execvp()`. The program never knows its output goes to a file.

### Why there are no zombies

A finished child whose status is never collected stays in the process table as a zombie. Each loop iteration calls `waitpid(-1, NULL, WNOHANG)`, which collects every finished child but returns immediately when none is ready, so the shell never blocks.

## What We Did and Why (Design Decisions)

These are the questions we asked ourselves while building the project and the answers we chose.

**Why a shell, and why in C?**
The UNIX Lab teaches process management, file descriptors, pipes and redirection. A shell is the one program that uses all of them together. Writing it in C means we call the system calls directly instead of through a language runtime, which is the clearest way to see what the kernel does.

**Why rename commands instead of writing a new tool?**
The goal was to remove the memorisation barrier, not to replace UNIX. We kept the real utilities, so output and edge cases are authentic, and anything learned here (flags, behaviour) carries over to standard Bash.

**What rules did we use for naming?**
- Self-describing: the name states the action (`showfiles`, `stopproc`).
- Consistent verb-object pattern: `make...` creates, `remove...` / `delete...` deletes, `...view` shows a file, `...proc` concerns processes.
- Flags are never changed, only the first word is translated.
- Hard cases are simplified in translation (`searchpath <path> <name>` becomes `find <path> -name <name>`).

**Why is `vi` not renamed?**
`vi` is a lab topic in its own right (Command mode, Insert mode, `:wq`), and its name is already short and unavoidable on real systems. We launch the real editor and let the `explain` tutor describe the keys.

**Why a `sandbox`?**
Beginners hesitate to experiment because commands like `rm` are destructive. The sandbox gives them sample data and a ready-made script in a throw-away folder.

**Why an `explain` tutor and a `guide` menu?**
Friendly names alone hide the real command. The tutor deliberately names the UNIX command and the system call behind each one, so learners graduate to real UNIX instead of staying dependent on Mini-Bash.

**Why only a single pipe?**
We built one pipe stage first to get the file-descriptor plumbing (`pipe()`, `dup2()`, closing the unused ends) completely correct. Extending to `a | b | c` needs an array of pipes and is listed under future scope.

**Why parse in place?**
The tokenizer overwrites separators with `'\0'` and stores pointers into the input buffer, so no memory allocation is needed. Quoted arguments are kept together, which is what lets `sed` and `awk` programs with spaces work.

**Why does the unknown-command message come from the child?**
After a failed `execvp()` the child prints a friendly message and exits with `_exit(1)`, so the beginner sees plain English instead of a raw system error.

## Mapping to UNIX Lab Outcomes

| Lab topic | How Mini-Bash applies it |
|---|---|
| Starter commands and GCC | Compiled with `gcc -Wall -Wextra`; all starter commands reachable under friendly names |
| `vi` editor modes | `vi <file>` runs the real editor; the tutor explains Command and Insert modes |
| File system and wildcards | Directory navigation, file creation, copying, moving, deleting, `chmod` modes; `searchpath . '*.txt'` passes the pattern to `find` |
| Process management | `fork()`, `execvp()`, `waitpid()`, background jobs, `WNOHANG` reaping, `ps` and `kill` |
| Redirection and pipes | `open()` + `dup2()` for `>`, `>>`, `<`; `pipe()` + `dup2()` for `\|` |
| Shell scripts and utilities | `auto_report.sh` (shebang, command substitution, `wc`, `grep`), environment variables, `grep`, `sort`, `cut`, `awk`, `sed` |

## Known Limitations

We list these honestly because understanding them was part of the project.

- **One pipe per line** (`a | b`), not chains. Pipelines always run in the foreground.
- **No wildcard expansion by the shell.** `*.txt` is passed literally; it only works where the utility handles it (for example `searchpath . '*.txt'`).
- **No `$VAR` expansion by the shell.** `setvar` sets the environment correctly and child programs see it (`runscript -c 'echo $VAR'`), but `say $VAR` prints the literal text.
- **Shortcut targets are a single word** (the target becomes the command name; arguments are not supported).
- **Basic quoting:** a `|` inside quotes is still treated as a pipe, and there is no backslash escaping.
- Finished background jobs are reaped when the next command is entered, not instantly.
- No command history or tab completion, and no `Ctrl+C` handling.

## Future Scope

- Multi-stage pipelines `cmd1 | cmd2 | cmd3` using an array of pipes, and background pipelines
- Wildcard globbing (`*.txt`) with `<glob.h>`
- Shell-level `$VAR` expansion and shortcuts with arguments
- Command history and line editing with GNU `readline`
- Signal handling for `Ctrl+C`
- A build-and-test script that compiles with `gcc` and analyses logs with `grep`, `awk` and `sed`
- Unit tests for the tokenizer and command mapper

## Repository Structure

```text
mini-bash/
├── minibash.c        # complete shell source (about 330 lines)
├── Makefile          # make / make run / make clean
├── README.md         # this file
├── .gitignore
├── LICENSE           # add one (see below)
└── docs/
    ├── synopsis.pdf  # project synopsis
    ├── report.pdf    # full project report with code analysis
    └── report.tex    # LaTeX source of the report
```

## Documentation

- **Project synopsis:** [`docs/synopsis.pdf`](docs/synopsis.pdf): objectives, module design and lab-outcome mapping
- **Project report:** [`docs/report.pdf`](docs/report.pdf): full architecture, original-versus-friendly command table, line-by-line code analysis and demonstration sessions

## Team

| Name | Enrollment No. |
|---|---|
| Kushagra Garg | 992501030406 |
| Yash Agnihotri | 992501030407 |
| Kunal Chawla | 992501030417 |

**Batch:** F-7 &nbsp;|&nbsp; **Course:** UNIX Lab (24B15CS214), B.Tech Semester-3
**Submitted to:** Dr. Arti Jain, Unix Lab Faculty
**Institute:** Jaypee Institute of Information Technology, Noida

## License

Released under the MIT License (see `LICENSE`). This project was created for academic purposes.
