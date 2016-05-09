# babak

A lightweight solution for designing and enforcing simple backup policies.  It is
essentially just a thin wrapper around `rsync` originally intended for cross-HPC
backup purposes.

## Requirements

The only dependency for babak is perl5.  It is however highly recommended
to use babak in combination with `crontab`.  It is also assumed that you have properly
set up SSH keys between your computers and the hosts.  More details can be found below.

## Installation

```
$ git clone https://github.com/ndejay/babak.git babak
$ cd babak
$ sudo cp babak /usr/local/bin
$ mkdir ~/.babak
```

Or alternatively, install it wherever you want and simply prepend the `PATH`
environment variable with the path to the `babak` executable.

```
$ export PATH=/home/ndejay/bin
$ mkdir ~/.babak
```

## Usage

First, define a set of hosts in `~/.babak/hosts.tsv`.  These are the hosts you are
interested in transferring files to and from.

```
#HOST	USERNAME	HOSTNAME	PORT
raynor	jraynor	hostname1.lan	22
raynor2	jim.raynor	hostname2.net	9001
```

Second, in `~/.babak/manifest.tsv`, define the backup policy you require.  Each row
is a separate source-destination operation in which at least one of the hosts involved
must be the local machine.

In the simplest use case, if you wish to transfer all the files in your home directory
to a remote server, write the following:

```
#SOURCE_PATH	DESTINATION_PATH	INCLUDE_PATTERN	EXCLUDE_PATTERN	OPTIONS
local:/home/jim	raynor:/home/jraynor/backup	*		
```
Warning: Every field must be present for every line. (A tab must separate every field
even if a field is empty)

Then, when you call `babak`, a shell script will be created in `~/.babak/script.sh`
which you can then add to your `crontab`.

Each entry in your `manifest.tsv` file will be launched in a separate thread, and each
thread will have its own temporary log file `~/.babak/script.0.log` which will then
be concatenated `~/.babak/script.log` when all threads are done running.

```
$ babak
$ cat ~/.babak/script.sh
#!/usr/bin/env sh
rsync -azup --rsh 'ssh -p 22' \
  --include '*/' --include '*' \
  --exclude '' --prune-empty-dirs /home/jim jraynor@hostname1.lan:/home/jraynor/backup \
  \
  > /home/jim/.babak/script.0.log &
cat /home/jim/.babak/script.0.log > /home/jim/.babak/script.log
rm /home/jim/.babak/script.0.log
wait
```

Define the frequency at which you want these transfers to be completed by adding the
one of the following entries to `crontab -e`:

```
# To run the transfer five minutes after midnight, every day
5 0 * * * $HOME/.babak/script.sh
# To run the transfer once a week on a Friday night at 10pm
0 22 * * fri $HOME/.babak/script.sh
```

For more sophisticated uses of `crontab`, feel free to consult online documentation on
the matter.

It is also recommended that you *manually* run (outside of `crontab`) the script
produced by babak, just to make sure that you have properly set up your SSH keys and
whatnot.

## Advanced Features

It is worth mentioning that all lines beginning with the `#` symbol are
automatically ignored, so feel free to annotate your file definitions as you
please.  Also, these are tab-separated files.

### Pattern Matching

If you wish to include all but a given a set of patterns, set `INCLUDE_PATTERN`
to `*` and define `EXCLUDE_PATTERN`.  For example, if you want to transfer all
files except those starting with a `.` (dotfiles) from your home directory on the
local machine to a remote host, follow this example:

```
#SOURCE_PATH	DESTINATION_PATH	INCLUDE_PATTERN	EXCLUDE_PATTERN	OPTIONS
local:/home/jim	raynor:/home/jraynor/backup	*	*.*	
```

If you wish to exclude all but a given set of patterns, set `EXCLUDE_PATTERN`
to `*` and define `INCLUDE_PATTERN`.  For example, if you want to transfer
all the files that start with a `.`, try the following:

```
#SOURCE_PATH	DESTINATION_PATH	INCLUDE_PATTERN	EXCLUDE_PATTERN	OPTIONS
local:/home/jim	raynor:/home/jraynor/backup	.*	*	
```

### Additional Options

You can also pass any extra options to `rsync` using the `OPTIONS` column.
For instance, if you want all extraneous files to be deleted in the destination,
define your entry as follows:

```
#SOURCE_PATH	DESTINATION_PATH	INCLUDE_PATTERN	EXCLUDE_PATTERN	OPTIONS
local:/home/jim	raynor:/home/jraynor/backup	.*	*	--delete
```

These will be passed verbatim to the `rsync` call.  You may wish to consult the
`rsync` manpage for a more exhaustive list of options.

## License

The MIT License (MIT)

Copyright (c) 2016 Nicolas De Jay, Claudia Kleinman

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
