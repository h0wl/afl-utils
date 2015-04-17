# afl-utils

Some utilities to automate crash sample processing and analysis for crashes
found with [american-fuzzy-lop (afl)](http://lcamtuf.coredump.cx/afl/).

### Dependencies

* Python3 (with sqlite3)
* [Exploitable](https://github.com/rc0r/exploitable) (for script execution support)

### Problems / Bugs

* These tools are slow!
* `avl_vcrash` might miss *some* invalid crash samples. Identification of real crashes is
  hard and needs improvements!
* `avl_vcrash` identifies *some* crash samples as invalid that are considered valid by
  `afl-fuzz` when run with option `-C`.
* Tool outputs might get cluttered if core dumps/kernel crash messages are displayed on
  your terminal (see `man core(5)`; workaround anybody?).
* ~~gdb+exploitable script execution will be interrupted when using samples that do not lead
  to actual crashes. `afl_collect` will print the files name causing the trouble (for manual
  removal).~~ Fixed by using a patched `exploitable.py` that handles `NoThreadRunningError`
  (see [Exploitable](https://github.com/rc0r/exploitable)). **Be sure to use the patched
  version of `exploitable.py`!**
* The more advanced features like gdb+exploitable script generation and execution as well as
  crash sample verification *probably will* fail for targets that don't read their input from
  files (`afl-fuzz` invoked without `-f <filename>`) but from `stdin`. I didn't look into this
  yet.

### Feature Ideas / ToDo

- [x] collect crash samples from `crashes.*` sub directories 
- [ ] "`stdin`-support" (see Problems/Bugs); We do get crash samples for "`stdin`"-mode, right?!
- [x] submit classification data into some sort of database
    - [x] basic sqlite3 database support added
    - [ ] want more db connectors? Drop me a line!
- [ ] auto clean-up of uninteresting crashes
    - [x] by exploitable classification
    - [ ] through some means of deduplicating crash samples (might be clever to incorporate this into
          the crash collection step;
          [some ideas](https://groups.google.com/forum/#!topic/afl-users/b5v3mY_hy30))
- [x] afl_multicore: wrapper script that starts multiple afl-instances for parallel fuzzing on multiple cores
    - [ ] tmux/screen mode
    - [ ] afl_multicore_exit/kill for quitting/killing all jobs at once
    - [ ] afl_multicore_watch for checking fuzzer_stats?
- [ ] afl_resume: wrapper script that resumes multiple afl-instances at once

### The Tools

#### afl\_collect

`afl_collect` basically copies all crash sample files from an afl synchronisation directory
(used by multiple afl instances when run in parallel) into a single location providing
easy access for further crash analysis. Beyond that `afl_collect` has some more advanced
features like invalid crash sample removing as well as generating and executing `gdb` scripts
that make use of [Exploitable](https://github.com/jfoote/exploitable). The purpose of these
scripts is to automate crash sample classification (see screenshot below).  

Usage:  

![afl_collect_usage](https://raw.githubusercontent.com/rc0r/afl-utils/master/.scrots/afl_collect_usage.png)

Sample output:

![afl_collect_sample](https://raw.githubusercontent.com/rc0r/afl-utils/master/.scrots/afl_collect_sample.png)


#### afl\_multicore

`afl_multicore` starts several parallel fuzzing jobs, that are run in the background (using `nohup`), so
afl's fancy interface is gone. Fuzzer outputs (`stdout` and `stderr`) will be redirected to `/dev/null`.
Use `--verbose` to see the outputs (`nohup.out` might also contain some useful info).
If you want to check the fuzzers' progress see `fuzzer_stats` in the respective fuzzer directory in
the synchronisation dir (`sync_dir/SESSION###/fuzzer_stats`)!
I might be adding some `tmux` or `screen` mode allowing to run the fuzzer instances in `tmux`/`screen`
with a separate window for each instance. This would bring back the nice interface and would be handy
in debugging faulty invocations of `afl-fuzz`.

Usage:  

![afl_multicore_usage](https://raw.githubusercontent.com/rc0r/afl-utils/master/.scrots/afl_multicore_usage.png)

Sample output:

![afl_multicore_sample](https://raw.githubusercontent.com/rc0r/afl-utils/master/.scrots/afl_multicore_sample.png)


#### afl\_vcrash

afl\_vcrash verifies that afl-fuzz crash samples lead to crashes in the target binary and
optionally removes these samples automatically.

Usage:

![afl_vcrash_usage](https://raw.githubusercontent.com/rc0r/afl-utils/master/.scrots/afl_vcrash_usage.png)
  
### Changelog

Release | Description
:-------:|----
0.10a | Initial release, just collect crash sample files
0.11a | Crash sample file list creation added, afl_vcrash added
0.12a | gdb+exploitable script generation added
0.13a | Auto-cleanup of invalid crash samples added
0.14a | gdb+exploitable script execution and output parsing added for easy crash classification
0.15a | Code refactoring, minor bug fixes
0.16a | Minor bug fix for gdb+exploitable script generation
0.17a | Basic SQLite3 database support added
0.18a | Fixed gdb+exploitable script interruptions that occur on graceful exits of the target binary
0.19a | Added auto-cleanup feature for samples leading to uninteresting crashes
0.20a | Sample collection from all `crashes*` sub directories added, minor bug fix for sample cleanup
0.21a | Initial version of afl_multicore added