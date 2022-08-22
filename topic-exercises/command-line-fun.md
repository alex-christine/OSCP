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

