---
description: Answers to Command Line Fun module
---

# Command Line Fun

## Text Searching and Manipulation

### VM1

Extract the 13th field from the file located on the Kali VM #1 in the _challenge_ folder in order to solve this problem challenge. Try to get the complete flag with a single one-liner (i.e. with no extra processing needed to submit).

```bash
$ cut -d "," -f 13 <file.csv> | paste -s -d '' | grep -o 'OS{\S*}'
```

### VM2

There's a pile of assorted flags in the VM #2 _/challenge_ folder. It's nice to have a lot of flags, but the right one is the shortest one. _Sort_ the pile in order to find it.

```bash
$ head values_and_flags.txt 
{{0bcecb6cb0862261510c0ed512dc694120f}},5976962477
{{061f5ce2f68ffe824f53e51c38bc01a3918}},6293041180
{{067dc63b79a8ad6f0e981a2c9c982bb37c0}},3438641438
{{02a61a34c7ae1b9b6b230897b86f1105130}},2012387166
{{0eb899500af42815da7b23deabd09f68969}},2582722867
{{09177e065f68bd634c90ea86af72d5ff35}},3533648260
{{071fd2988b0fdd68296c8eeac00a9f73dbe}},1864381346
{{01bc195b9a3e606c3bb7ebde39ee86b7ef}},1224724138
{{071109aa5817214ad329935e861577b6179}},7111034910
{{0ce3b6c78e1dd7a9f15112dc6b1da8676d2}},5352073331

$ awk '{ print length, $0 }' values_and_flags.txt | sort -k1n | head -n 1 | cut -d" " -f2 | grep -o 'OS{\S*}'
OS{FLAG}
```

## Comparing Files

### VM1

There are two access-logs, `access-logA` and `access-logB`. Spot the differences (and ONLY the differences) _in order of appearance_ in their respective files to get this flag.

```bash
$ diff --suppress-common-lines access-logA.txt access-logB.txt | awk '/</ || />/' | grep -o '[A-Za-z0-9{}]' | paste -s -d ''
OS{FLAG}
```

## Managing Processes

### VM1

We need your help to complete some dirty jobs. These jobs are available on the VM #1 within the _/challenge_ folder. Follow the instructions given by the _dirty-jobs_ program to learn how to complete these jobs and get the flag.

```bash
$ ./dirty-jobs 
So you want to help me complete some dirty jobs; Awesome!
Go ahead and start by running each of the four dirty jobs available in ./jobs.
Make sure you do NOT complete these jobs (i.e. do NOT end these processes).
These jobs (processes) need to be visible in your process list for me to verify that you are working the job.

Once you have one active session of each job (4 jobs total), press any key to continue: 

^Z
[1]+  Stopped                 ./dirty-jobs

```

Started all jobs and suspended them with `Ctrl + Z`.

```bash
$ jobs
[1]   Stopped                 ./dirty-jobs  (wd: /challenge)
[2]   Stopped                 ./avian-vomitologist
[4]   Stopped                 ./garbage-collector
[5]-  Stopped                 ./pig-farmer
[6]+  Stopped                 ./sewer-inspector
```

Jumped back into `./dirty-jobs`.

{% code overflow="wrap" %}
```bash
$ fg %1                                                                                                           
./dirty-jobs    (wd: /challenge)

You went pig-farming. Yuck.
You collected some garbage. Gross.
You inspected a sewer. Disgusting.
You handled some avian vomit. I am starting to gag.


Ugh. My stomach does not feel so good. Please hurry up and finish those jobs before I lose my lunch.
At this time, pause this process and finish (i.e. end) all dirty jobs processes.
In fact, end every other process (EXCEPT your shell of course - unless you want to start over).

NOTE: Your process list should only list three processes (bash, dirty-jobs, and your process list command) once ready.
The process list command is listed because it is running at the time of executing the ps (ps inception),
but it does not exist after completion of the command. In other words, do not worry about the ps.

NOTE2: The default kill (just `kill <pid>`) will not work on the dirty job processes.
These processes do not respond to SIGTERM. Make sure you send SIGKILL if you are trying to use `kill`.

Once all other jobs are complete (i.e. ended), press any key to continue: 

^Z
[1]+  Stopped                 ./dirty-jobs  (wd: /challenge)
(wd now: /challenge/jobs)

```
{% endcode %}

Killed all processes using a SIGKILL signal (`kill -9 <PID>`).

```bash
$ fg %1                                                                                                           
./dirty-jobs    (wd: /challenge)

Great job. Here is your flag:
OS{2793b0bd43cfd4cb2d59513f26a841cf}

Press any key to continue...
```

## File and Command Monitoring

### VM1

```bash
$ cd /challenge && watch -n1 -e ./watchmen 
```

After awhile the output froze on a screen with the `OS{FLAG}`

## Downloading Files

### VM1

Exercise was simply to start the VM and then download an HTML file from it via `curl`. The HTML page contained `OS{FLAG}`
