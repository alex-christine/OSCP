# Topic Exercises

May find **Topic Exercises** inside each **Practice** section.&#x20;

Each exercise has three components:&#x20;

* [Question](topic-exercises.md#question)
* [Machine](topic-exercises.md#machine) (or a group of machines)
* [Flag](topic-exercises.md#undefined)&#x20;

The question asks you to perform a specific action or set of actions on the provided machine. Once you have successfully completed the objective, you will receive a flag in the form `OS{random-hash}`. You can then submit the flag into the Offsec Training Library (OTL), which will tell you if you have inserted the correct flag or not. The OTL will then save your progress, and track the number of correct submissions provided to date by yourself.

The way Topic Exercises are implemented allows us to use the same remote IP and port multiple times.

On the Topic Exercise VMs that require an SSH connection, course suggests issuing the the SSH command with a couple of extra options as follows:

`ssh -o "UserKnownHostsFile=/dev/null" -o "StrictHostKeyChecking=no" student@192.168.50.52 -p 2222`

The `UserKnownHostsFile=/dev/null` and `StrictHostKeyChecking=no` options have been added to prevent the **known-hosts** file on our local Kali machine from being corrupted.

## Question

Exercises usually (though not always) provide you with the exact objectives they must accomplish on the machine. The **question provides a machine name that you can start directly inside the OTL**. You can then connect to the machine through their Kali Linux VM and the course VPN pack.

## Machine

All of the Topic Exercise VMs are **contained in the student's own individual lab environment**. That is, you do not share these machines with other students, and therefore you will be able to start, stop and revert the machines as necessary.&#x20;

The Topic Exercise machines neither replace the PWK shared lab environment, nor do they replace the three dedicated client machines assigned to each student.

## Flag

Flags are often found as the contents of text files, but they can be hidden in a variety of locations. Flags always have a definite length, but are randomized on each revert of the exercise machine. Flags are always of the form `OS{d8e8fca2dc0f896fd7cb4cb0031ba249}`.

Because the flag is regenerated at machine boot **the flag of a compromised machine must be submitted before reverting**. If a student compromises, reverts, then submits the flag will be rejected as invalid.
