---
layout: post
title: "Smashing the Stack: Level 1"
date: 2013-01-31 20:25
comments: true
categories: challenges io sts
---

Welcome to level 1! This first stage simply weeds out the people that have _absolutely_ no idea what they're doing.
It's also a quick refresher on how to use gdb. In the past level 1 has been even simpler than it is now, but nevertheless
it is still elementary stuff.

The [Smashthestack](http://io.smashthestack.org:84/) website for IO shows you how to connect to level 1 and gives you the username
and password in order to ssh into the server.

```
[xtc@apollo ~]$ ssh io.smashthestack.org -l level1 -p 2224

level1@io.smashthestack.org's password: (level1)
```

After connecting you're shown an motd that tells you where the levels are stored and where to find the password to the next level
after you have completed each challenge.

Change your current working directory to `/levels/` and you can see all the levels that are in the IO set.

##Level 1

Starting off is simple. If you notice, some source code is provided for levels further down the road, but not level1.
Execute the level1 binary and it will tell you what it's looking for (a password as the first and only argument).
In order to find the password you're going to need to reverse-engineer the binary and extract the key. Let's start by spawning
a gdb session with the level1 binary loaded on the server.

```
level1@io:/levels$ ./level01 password
Fail.
level1@io:/levels$ gdb ./level01 -q
(gdb)
```

The `-q` option for gdb simply stands for 'quiet mode' so it doesn't print lengthy license information every launch.
Immediately afterwards I set the option `disassembly-flavor` to `intel`. This is primarily my own preference due to the readibility
of Intel's syntax compared to AT&T's. Choose whichever you like, but I will continue with Intel syntax for the remainder of the
challenges.

To start off debugging the first level binary in gdb, execute `disas main`. This statement does exactly what you'd think:
disassembles the main function.

```
(gdb) set disassembly-flavor intel
(gdb) disas main
...
0x080485b1 <main+27>:   mov    DWORD PTR [esp+0x4],eax
0x080485b5 <main+31>:   mov    DWORD PTR [esp],0x8048760
0x080485bc <main+38>:   call   0x80483b8 <printf@plt>
0x080485c1 <main+43>:   mov    DWORD PTR [ebp-0x4],0x0
0x080485c8 <main+50>:   jmp    0x8048618 <main+130>
0x080485ca <main+52>:   call   0x804852d <pass>
0x080485cf <main+57>:   mov    DWORD PTR [esp+0x8],0x64
0x080485d7 <main+65>:   mov    eax,DWORD PTR [ebp+0xc]
...
(gdb)
```

I've cut out the center of the output to demonstrate what this function does in terms of assembly code. I won't be explaining
everything; it's up to you to teach yourself basic assembly instructions (which are quite simple I might add).

Clearly you will notice right away that there's a call to a function called 'pass' which seems oddly suspicious.
Let's disassemble that function and peek inside to catch a glimpse of what it's hiding.

```
(gdb) disas pass
...
0x08048533 <pass+6>:    mov    DWORD PTR [ebp-0x4],0x8049140
0x0804853a <pass+13>:   mov    DWORD PTR ds:0x8049140,0x53
0x08048544 <pass+23>:   mov    DWORD PTR ds:0x8049144,0x65
0x0804854e <pass+33>:   mov    DWORD PTR ds:0x8049148,0x63
0x08048558 <pass+43>:   mov    DWORD PTR ds:0x804914c,0x72
0x08048562 <pass+53>:   mov    DWORD PTR ds:0x8049150,0x65
0x0804856c <pass+63>:   mov    DWORD PTR ds:0x8049154,0x74
0x08048576 <pass+73>:   mov    DWORD PTR ds:0x8049158,0x50
0x08048580 <pass+83>:   mov    DWORD PTR ds:0x804915c,0x57
0x0804858a <pass+93>:   mov    DWORD PTR ds:0x8049160,0x0
...
```

Clearly a string is being moved into the above memory addresses in the form of hexadecimal representations of ascii characters.

Let's break this apart to see what the string is. If this were a normal string I would be able to easily print it out by pointing
to the first memory address that it is held in. It appears that each character is placed at every fourth byte, so we will multiply
the amount of characters (8) by the padding reserved for each character (3 bytes), and then examine that collection of bytes and print
the chracters in each buffer.

```
(gdb) x/24s 0x8049140
0x8049140 <pw>:  "S"
0x8049142 <pw+2>:        ""
0x8049143 <pw+3>:        ""
0x8049144 <pw+4>:        "e"
0x8049146 <pw+6>:        ""
0x8049147 <pw+7>:        ""
0x8049148 <pw+8>:        "c"
0x804914a <pw+10>:       ""
0x804914b <pw+11>:       ""
0x804914c <pw+12>:       "r"
0x804914e <pw+14>:       ""
0x804914f <pw+15>:       ""
0x8049150 <pw+16>:       "e"
0x8049152 <pw+18>:       ""
0x8049153 <pw+19>:       ""
0x8049154 <pw+20>:       "t"
0x8049156 <pw+22>:       ""
0x8049157 <pw+23>:       ""
0x8049158 <pw+24>:       "P"
0x804915a <pw+26>:       ""
0x804915b <pw+27>:       ""
0x804915c <pw+28>:       "W"
0x804915e <pw+30>:       ""
0x804915f <pw+31>:       ""
```

Wow! That was simple. I bet that's the password we need. Let's plug it in to the first argument to verify.

```
(gdb) q
level1@io:/levels$ ./level01 SecretPW
Win!

You will find the ssh password for level2 in /home/level2/.pass
sh-4.1$
```

Winner! Congratulations, you've passed the first level. You may not realize what you're getting yourself into.
Now cat the password from level2's home directory and move on to the next stage.
