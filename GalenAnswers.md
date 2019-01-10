# Questions

## HPC good citizenship

1. On the UCI cluster, the resource request "-pe openmp 64" refers to the number of processors requested.  Does that
   request mean that your commands will use multiple processors?
2. In general, how do you know how many processors, how much RAM, how many files would be required/needed/written by the
   jobs you are running on the cluster?
3. In order to be a "good citizen", you need to have some idea of much RAM your job requires.  In particular, you need
   to know the "peak" (i.e., maximum) RAM required at any point during execution.  Show an example of the shell command
   that you would use on a Linux machine to measure run time and peak ram usage of an arbitrary command, where the time/peak RAM values are written to a file.
4. What are the units of your answer for number 3?
5. What are the bash commands for the following operations:

    * Checking that a file exists
    * Checking that a file exists and is not empty

6. How would you use the commands from your answer to 5 to write a work flow for HPC that only runs a job if the
   expected output file is **not** present.

## Trickier questions

7. Would your answer to number 3 work on Apple's OS X operating system?  If no, do you have any idea why not? 

8. Most of the HPC nodes have 512Gb (gigabytes) of RAM. Let's say you have a job that will require **no more** than 24Gb
   of RAM.  How would you request resources so that you can run more than one job on a node **and** not cause nodes to
   crash?  Show an example of a skeleton HPC script as part of your answer.  **Knowing how to do this is super important
   and will save you loads of frustration and prevent you from taking out your colleagues' jobs on the cluster,
   preventing you from getting nasty emails from Harry!!!!!!!!!!!**
   
# Answers

## HPC good citizenship

1. Using "-pe openmp 64" does not mean that commands will use multiple processors, just that 64 cores will be reserved for your job. As far as I can tell, there's nothing to stop someone from submitting an extremely simple script with a single command and requesting 64 cores for it, but the command would still only run on one core.

2. One can find this information by running a smaller test dataset prefixed by the "/usr/bin/time -v" command. For example, running the commands 
```
touch timetest.txt
/usr/bin/time -v echo "This is me testing the time command" >> timetest.txt
```
gives the output:
```
        Command being timed: "echo This is me testing the time command"
        User time (seconds): 0.00
        System time (seconds): 0.00
        Percent of CPU this job got: 0%
        Elapsed (wall clock) time (h:mm:ss or m:ss): 0:00.00
        Average shared text size (kbytes): 0
        Average unshared data size (kbytes): 0
        Average stack size (kbytes): 0
        Average total size (kbytes): 0
        Maximum resident set size (kbytes): 1600
        Average resident set size (kbytes): 0
        Major (requiring I/O) page faults: 0
        Minor (reclaiming a frame) page faults: 72
        Voluntary context switches: 7
        Involuntary context switches: 1
        Swaps: 0
        File system inputs: 0
        File system outputs: 8
        Socket messages sent: 0
        Socket messages received: 0
        Signals delivered: 0
        Page size (bytes): 4096
        Exit status: 0
```
This output includes RAM needed (from which one can tell how many cores to request) and files read/written.

3. To alter the previous command so that it writes execution time and RAM useage to a file, one can use the following command instead of using the "-verbose" command:
```
/usr/bin/time -f "Time: %E, max RAM: %M" -o timetest.txt echo "This is me testing the time command" >> wctest
```
This command prints output to the file "timetest.txt" in the format "Time: x:xx.xx max RAM: xxxx"
For the command above, it worked out as "Time: 0:00.00, max RAM: 1600"

4. The units of elapsed time are h:mm:ss, and for RAM they are kbytes.

5. One can use the "test" command to see whether files exist using conditional execution or an if...fi statement:
```
touch whale
[ -e whale ] && echo "There she blows" || echo "Argh matey"
```
This should output "There she blows" since we just created the file "whale". The "-s" option of the test command will check that a file exists and isn't empty. If we change the "-e" option to "-s", the output will be "Argh matey" since we just created the (empty) file.

6. Here is an example of how an if...fi statement could be used to prevent a command from outputting and overwriting a file which already exists:
```
if [ ! -s outfilename.txt ]
then
command infilename.txt > outfilename.txt
else
echo "stopped: outfilename.txt already exists"
fi
```

## Trickier questions

7. "usr/bin/time -v/f/o" will not work on OS X, at least according to [this stack overflow post](https://stackoverflow.com/questions/32515381/mac-os-x-usr-bin-time-verbose-flag) because OS X uses BSD time which doesn't include the RAM measurement. According to an answer on the same post, it can be accomplished by installing GNU time manually. I don't have a computer with OS X, so I haven't been able to test whether my answers are correct.

8. We can ensure that our process won't try to use more RAM than is available and crash the node by ensuring that the node has the necessary amount of free RAM before the job is loaded on:
```
#$ -l mem_free=24GB
```
In addition, we have to limit the amount of RAM being used by each job. Since each core = 8 Gb, 3 [core ram equivalents](https://hpc.oit.uci.edu/BioClusterGE.pdf) are needed per job (24/8 = 3). So, we add the line:
```
-pe openmp 3
```
This line will ensure that it is impossible for so many jobs to be loaded onto the node that the node would crash. At most, 21 of these jobs could be loaded onto a single node, using up a total of 504 Gb of RAM at maximum.

Sample script:
```
#!/bin/bash
#$ -N myjobname
#$ -q bio,free64
#$ -m beas
#$ -M galentm@uci.edu   
#$ -pe openmp 3

some commands here
```
