# Introduction

Coming to robotics from a mechanical engineering background, I found myself
wanting to know more about computation, computer science and software
engineering.  The projects shown here summarize some of my self-learning
experiences.

# Projects

## Linux From Scratch (LFS)

In this project you download and compile all the separate pieces to
assemble your own bootable Linux distro. No coding is done in this project,
but you need to be able to follow instructions, know how to debug and know
a bit about UNIX. I managed to finish the project and registered on their
database [here](http://www.linuxfromscratch.org/cgi-bin/lfscounter.php). My
LFS id is 28548. 

## 6502 modification

The 6502 is praised for it's simplicity and remembered for the impact it
had on personal computing, so I decided to work though this tiny e-book I
found called [Easy 6502](https://skilldrick.github.io/easy6502/). After
reading the little book, I modified the snake game presented there to
reinforce what I had learnt.  The modifications make `snake`'s stomach
contents visible. The code can be found on my GitHub
[here](https://github.com/vecf/urWu8Snake) and the video below shows a
snippet of the gameplay.

<video width="320" height="240" alt="You Are What You Ate Snake Game"
controls> <source src="media/learning/urwu8.mp4" type="video/mp4">
</video>

## From NAND to Tetris

After playing with 6502, I was confident enough to try
[Nand2Tetris](https://www.nand2tetris.org/). The subtitle "Building a
Modern Computer From First Principles", is actually just the first half of
the course! In part 1, using only the
[NAND](https://en.wikipedia.org/wiki/NAND_gate) gate and
[DFF](https://en.wikipedia.org/wiki/Flip-flop_%28electronics%29)'s, you
build the chips to create the `Hack` computer: an educational
([MISC](https://en.wikipedia.org/wiki/Minimal_instruction_set_computer))
architecture. This is done in an educational hardware description language
provided with pre-written tests to make development incremental. By the end
of the course you write your own assembler for `Hack` in the programming
language of your choice. I did it in `Python`.

In part 2, you build a compiler targeting `Hack` for the `Jack`
object-oriented programming language which the authors created: I stuck
with `Python`. You also get to implement any project of your choice in the
`Jack` language to learn how it works.  Typically, students go for
`Tetris`, hence the name. I re-implemented `snake`. In the final project,
you implement the system calls provided by a bare-bones "operating system"
in `Jack`. 

The clip below shows the hardware emulator running my `snake` clone. When
loading snake without the system binaries, the emulator offers to provide
built-in versions. Then, when loading a version with snake and the system
binaries, the emulator immediately loads the program. 

Instead of the direction keys I used `vi` keys because I was the target
audience of the game. You set the speed and length of snake at the
beginning of the game to help debug during development. I implemented
playing the game at different sizes as a programming exercise, this is not
demonstrated in the video.  

<video width="320" height="240" alt="nand2tetris demo" controls> <source
src="media/learning/n2tdemo.mp4" type="video/mp4"> </video>

## MIT 6.004 - Computational Structures

I worked through the worksheets and labs of the 2017 version of this [MIT
ocw
course](https://ocw.mit.edu/courses/6-004-computation-structures-spring-2017/)
together with its [companion website](https://computationstructures.org).
Computational structures explores important topics that N2T skips: models
of computation, cost/performance trade-offs, virtual memory, caching, memory
hierarchy, interrupts, scheduling, pipelining and concurrency, compiler
optimizations. 

There are three major challenges in this course, completed using instructor
developed tools:

1. Programming a Turing machine to check if parentheses are balanced using
   the `TMSim` Turing machine simulator.
1. Build a minimal
   [RISC](https://en.wikipedia.org/wiki/Reduced_instruction_set_computer)
processor called 'Beta' using the `Jade` circuit simulator.
1. Modify a teaching operating system called `TinyOS` in the `BSIM` 'Beta'
   emulator. 

The companion website implements all these tools using browser
technologies. To see my answers to the labs in your browser: 

1. Right-click and select 'Save Link as ...' on my [saved\_state.json
file](media/learning/saved_state_vecf.json)
1. Navigate
   [here](https://computationstructures.org/exercises/fsm/lab.html), scroll
down and upload the file.  
1. You can then browse to any of the labs and my answers should display.


### Parenthesis Checking Turing Machine

The video below shows a Turing machine programmed to halt on a `1` if the
parenthesis on the input tape are balanced and `0` if they are not.  Doing it
with only two states gets you full marks. This was the most challenging
part of the [FSM](https://en.wikipedia.org/wiki/Finite-state_machine)
[lab](https://computationstructures.org/exercises/fsm/lab.html#). To see it
in action, scroll down and click 'Open `TMSim` in a new window', click
'TMSim assemble' and then 'Checkoff'.

<video width="320" height="240" alt="Turing Paren Checker"
controls> <source src="media/learning/TMSIM.mp4" type="video/mp4">
</video>

To do it in just two states, I developed the diagram shown below. The *f*
**state** represents 'found' and *l* is for 'looking'. The *o* **symbol** is
for 'open' and *s* is for 'start'. Symbols *z* and *y* are arbitrarily
named. The *l* **motion** means move the **tape** left, *r* is right and
*-* means don't move. This corresponds to the language of TMSim as seen in
the video.

![](media/learning/TuringStates.png)

### Build and Optimize the 'Beta' CPU

The final lab is [Optimizing the
Beta](https://computationstructures.org/exercises/dp/lab.html) that you
built in the previous labs.  Here a 'Benmark' of 20 is considered very
doable and 40 requires considerable insight and investment of time. Though
not considered a course requirement, I tried to pipeline my design but
failed. See the lab for more details.  My final working design is tested in
`/user/test_v7` and got a benchmark of 34.66.  You can also inspect the
`Beta` for this test looking at `/user/beta_6` and descending into its
sub-components.

![](media/learning/beta.png)

### Modify TinyOS

The [TinyOS
lab](https://computationstructures.org/exercises/tinyos/lab.html) provides
a minimal time-sharing OS written in `Beta` assembly. The provided code has
two user processes: one periodically prints a counter value and the other
takes user input and converts it to [Pig
Latin](https://en.wikipedia.org/wiki/Pig_Latin).  There are three design
problems in this lab:

1. Initially, the Pig Latin translator doesn't work. Once the kernel to
   user memory mapping is fixed, translations are printed at the console. 
1. Add a mouse device driver and print the coordinates of user clicks.
1. Fix interleaving text at the console due to concurrent access by the
   user processes by implementing locking primitives.

BSIM has a schematic view that shows the assembly code being executed on the
hardware and an editor mode that follows along line by line as the program
executes.  The video below showcases these features.

<video width="320" height="240" alt="BSIM Feature Demo"
controls> <source src="media/learning/BSIMfeatures.mp4" type="video/mp4">
</video>

The next video shows the end result of the modifications done in the TinyOS
lab.

<video width="320" height="240" alt="TinyOS Modifications"
controls> <source src="media/learning/TinyOS.mp4" type="video/mp4">
</video>

## Ongoing Studies

I also started studying operating systems with materials from
[here](https://pdos.csail.mit.edu/6.828/2021/xv6.html),
[here](https://pages.cs.wisc.edu/~remzi/OSTEP/) and [here](https://ops-class.org/), and plan to continue when I have more time.

# Conclusion

I learnt a lot doing these projects and feel that I have a solid
understanding of computation. It is not so much of mystery to me anymore
how my code gets executed! I am confident that these lessons will help me
to be a better software engineer.
