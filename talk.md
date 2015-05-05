title: GTD at the command line
author:
  name: William Bert
  twitter: williamjohnbert
  github: sandinmyjoints
output: talk.html
style: style.css

--

# Getting Things Done at the Command Line

<link rel="stylesheet" href="//cdnjs.cloudflare.com/ajax/libs/highlight.js/8.5/styles/default.min.css">
<script src="//cdnjs.cloudflare.com/ajax/libs/highlight.js/8.5/highlight.min.js"></script>
<script>hljs.initHighlightingOnLoad();</script>

--

## Goal

- Share some shell knowledge I've found useful.

--

### Everything I'm going show here can be done other ways.

<p></p>

### There's nothing special about this way. I just find it convenient.

--

## Not a goal
- Compare different ways of doing things.

--

## If you've got a prompt, you've got a lot of power.

<http://aadrake.com/command-line-tools-can-be-235x-faster-than-your-hadoop-cluster.html>

--

## Humble ls

`ls` isn't very useful.

    $ ls
    data      hist.csv  talk.html

--

But we can improve it.

    $ alias ll='ls -lah'
    $ ll
    total 124232
    drwxr-xr-x  5 william  staff   170B May  4 16:17 ./
    drwxr-xr-x  8 william  staff   272B May  4 16:17 ../
    -rw-r--r--  1 william  staff     0B May  4 18:52 .hidden-file
    drwxr-xr-x  2 william  staff    68B May  4 16:17 data/
    -rw-r--r--  1 william  staff    61M May  4 16:18 hist.csv
    lrwxr-xr-x  1 william  staff    12B May  4 16:17 talk.html@ -> ../talk.html

- Long format.
- All files.
- Human readable sizes.
- If you take nothing else away from this talk...

--

Can use aliased `ll` with flags, too.

    $ ll -Sr
    total 124232
    -rw-r--r--  1 william  staff     0B May  4 18:52 .hidden-file
    lrwxr-xr-x  1 william  staff    12B May  4 16:17 talk.html@ -> ../talk.html
    drwxr-xr-x  2 william  staff    68B May  4 16:17 data/
    drwxr-xr-x  5 william  staff   170B May  4 16:17 ./
    drwxr-xr-x  8 william  staff   272B May  4 16:17 ../
    -rw-r--r--  1 william  staff    61M May  4 16:18 hist.csv

- Sorted by size.
- Reversed, so largest is last.

--

### More?

    $ man ls

- `man` uses `less`.
- Press `h` for help.

--

## Motivating example

- Historical query data

--

## Historical queries

<table>
<tr>
  <td>What are we working with?</td>

  <td>`$ file hist.csv`</td>
</tr>

<tr>
  <td>How big is it?</td>

  <td>`$ ll hist.csv`</td>
</tr>

<tr>
  <td>How many lines is that?</td>

  <td>`$ wc -l hist.csv`</td>
</tr>

<tr>
  <td>What does the data look like?</td>

  <td>`$ head hist.csv`</td>
</tr>

<tr>
  <td>Does the caret character appear in it?</td>

  <td>`$ cat hist.csv | grep '\^'`</td>
</tr>

<tr>
  <td>Replace every quote with a caret?</td>

  <td>`$ cat hist.csv | sed 's/"/^/g'`</td>
</tr>

</table>

--

<table>

<tr>
  <td>The millionth line?</td>

  <td>`$ cat hist.csv | head -1000000 | tail -1`</td>
</tr>

<tr>
  <td>Grab 10000 lines from the middle?</td>

  <td>`$ cat hist.csv | head -1350000 | tail -10000`</td>
</tr>

<tr><td>Grab 100000 random lines?</td>

  <td>
    <code>
    $ cat hist.csv | head -1350000 | tail -10000 >> somewhat-random.csv
    </code>
    <br />
    <code>
    $ cat hist.csv | head -1450000 | tail -10000 >> somewhat-random.csv
    </code>
    <br />
    <code>
    $ ^45^55
    </code>
    <br />
    <code>
    $ cat hist.csv | head -1550000 | tail -10000 >> somewhat-random.csv
    </code>
  </td>
  </tr>
  <tr>
    <td>See just the queries?</td>

    <td>`$ cut -d , -f 1 hist.csv | head`</td>
  </tr>

  <tr>
    <td>Sort by query? (~56 seconds)</td>

    <td>`$ sort hist.csv`</td>
  </tr>

  <tr>
    <td>See just the popularity?</td>

    <td>`$ cut -d , -f 2 hist.csv | head`</td>
  </tr>

  <tr>
    <td>Sort by popularity? (~2m18s)</td>

    <td>`$ cat hist.csv | sort -t , -k 2 -g`</td>
  </tr>
</table>


--

## This is what people mean when they talk about composable.
- Each program does one thing.
- Newline-separated text is the universal interface.

--

## Because the interface is simple
## and well defined,
## it's easy to plug yourself into it.

Example: upper.js

--

    #!/usr/bin/env node

    process.stdin.on('readable', function() {
      var data = process.stdin.read();

      while (data) {
        data = data.toString().toUpperCase();
        process.stdout.write(data);
        data = process.stdin.read();
      }
    });

    process.stdout.on('error', function(err) {
      if (err.code === 'EPIPE') {
        process.exit(0);
      }
    });

--

## Limitations / Suggestions
- Clearly, not everything is newline separated text!
- (But many things are or can be.)
- Be careful with destructive changes. Better to write to a new file.
- Iterate on your commands.

--

## Motivating problem

- Start/stop MySQL, when running multiple instances.

## How to start

    $ /usr/local/opt/mysql/bin/mysqld --bind-address=127.0.0.1 \
      --port=3311 --general-log-file=./mysql_data/mysql.log \
      --datadir=./mysql_data --socket=/tmp/mysql.my-database.3311.sock &

## How to stop?

- Search through running processes for mysqld, find the one using port 3311,
  note its process number, stop that process.

--

## Processes

- `ps`: shows information about running processes.
    - `e` for every process.
    - `f` for full data about each process.
- `kill`: sends a signal to a process.
- Exit codes: `0` means success. Anything else means error.
- Control operators: `;`, `&&`, `||`
- Sub-shells: `$(command)` lets you do something with the output of `command`.
- Variables: `OUTPUT=$(command)` saves the output of `command`.

--

## Shell scripts
### Pros
- Highly portable.
- No runtime needed.
- Can use familiar commands.

### Cons
- Shell is not a great language. Easy to make mistakes.
- Unfamiliar syntax for some things.

--

## My rules of thumb
- Shell scripts are best when you keep them short and simple.
- Learn certain patterns and reuse them.
- Somtimes just one line saved to a script is helpful.
    - Example: `db_tunnel_*` scripts
- Beyond one screenful of code, consider using something else.

--

    #!/bin/bash

    MYSQLD_BIN=/usr/local/opt/mysql/bin/mysqld

    PORT=$(./_mysql_port)

    if [ -z "$PORT" ]; then
        echo "Could not detect port. Bailing.";
        exit 1
    fi

    echo Checking for mysqld running on port $PORT...

    pid=$(./_mysql_pids) || { echo "Error checking for mysql pids. Bailing."; exit 1; }

    if [ -z "$pid" ]; then
        $MYSQLD_BIN \
            --bind-address=127.0.0.1 \
            --port=$PORT \
            --general-log-file=./mysql_data/mysql.log \
            --datadir=./mysql_data \
            --socket=/tmp/mysql.my-database.$PORT.sock \
            2>/dev/null 1>/dev/null &
        echo "MySQL started."
    else
        echo mysqld is already running on port $PORT with pid $pid.
    fi

--

    #!/bin/bash

    PORT=$(./_mysql_port)

    if [ -z "$PORT" ]; then
        echo "Could not detect port. Bailing.";
        exit 1
    fi

    echo Checking for mysqld running on port $PORT...

    pid=$(./_mysql_pids) || { echo "Error checking for mysql pids. Bailing."; exit 1; }

    if [ -z "$pid" ]; then
        echo mysqld was not running on port $PORT.
    else
        kill $pid 1>/dev/null && echo Stopping mysqld running on port $PORT with pid $pid.
    fi

--

    #!/bin/bash

    PORT=$(./_mysql_port)

    if [ -z "$PORT" ]; then
        echo "Could not detect port. Bailing.";
        exit 1
    fi

    ps -e | grep -v grep | grep -E "mysqld .*--port=$PORT" | cut -d ' ' -f 1

--

## Motivating problem

Every day, router errors spike for a few minutes, causing alarms to go off.

If you can reproduce the problem, you can fix it.

--

## Log files
- Newline-separated text files: the universal interface.
- Where they are: depends on project.

--

## Useful commands
- `$ ll -tr`
    - Time-sorted.
    - Reversed, so most recently modified is last.
- `$ tail -f file.log`
- `$ tail -f file.log | grep -E '"error":'`

--

## Bash functions

- Example: `greplog`

--

## Limitations
- Only one machine at a time.
- Loggly is much better at aggregation.

## But...
- This is realtime.
- We don't send everything (info, router) to Loggly.

--

## Finding and acting on files

- `ls` again.
- `find`: find files.
- `xargs`: act on multiple files.

--

## curl and jq

- Only things I'm going to show you that aren't part of POSIX.
- (POSIX is a standard for OSes.)
- Example: heredoc, xargs, curl, jq

--

## OS X, Coreutils

- OS X is a BSD. Not a Linux.
- OS X tools are sometimes older, lack some features.
- Coreutils: set of tools installed on most Linuxes.
- `brew install coreutils`
- Example: grep vs. ggrep

--

## Resources

### Tutorials
- http://linuxcommand.org/index.php
- http://tldp.org/LDP/Bash-Beginners-Guide/html/
- http://tldp.org/HOWTO/Bash-Prog-Intro-HOWTO.html

### Reference
- http://www.bashoneliners.com/
- http://linoxide.com/guide/linux-command-shelf.html
- http://clippy.in/b/YJLM9W
- http://tldp.org/LDP/abs/html/
    - http://tldp.org/LDP/abs/html/refcards.html
- https://gist.github.com/jorin-vogel/2e43ffa981a97bc17259 -- collection of solutions to a csv problem including many
  using bash one-liners.
