Download Link: https://assignmentchef.com/product/solved-i220a-lab-13
<br>
The aim of this lab is to introduce to you to the C library functions used for doing input/output (I/O). After completing this lab, you should be familiar with the following topics:

<ul>

 <li>The use plain char’s as return values.</li>

 <li>Opening and closing</li>

 <li>Reading/writing blocks of bytes.</li>

</ul>

<h1>1.2        Background</h1>

Unlike other languages which were prevalent at the time C was rst implemented, the C language itself does not specify any I/O facilities. Instead, all I/O in C is relegated to library functions. C comes with I/O functions in its standard library; usually, these functions are declared in the standard header le &lt;stdio.h&gt;.

The C I/O functions will work irrespective of which operating system the function is running on. However, they will be implemented using the native I/O facilities; under Unix they will wrap Unix primitives like <a href="https://en.wikipedia.org/wiki/File_descriptor">le descriptors</a><a href="https://en.wikipedia.org/wiki/File_descriptor">,</a> read¬ () and write().

<h1>1.3        Starting Up</h1>

Use the startup directions from the earlier labs to create a work/lab13 directory and re up a terminal whose output you are logging using the script command. Make sure that your lab13 directory contains a copy of the les directory.

<h1>1.4            Standard Input Character Count</h1>

Change over to the stdin-char-count directory. Look at the stdin-char-count.c program as well as the Make le. The program simply outputs the encoding for EOF followed by the number of characters read from standard input. However, note that the Make le uses a special -f ag to ensure that a plain char type is unsigned; hence the declaration for char c in the program is equivalent to unsigned char c.

Compile and run the program on itself:

$ ./stdin-char-count &lt;stdin-char-count.c

The program should output the value of EOF as -1 but will then get stuck in an in nite loop. ^C out of it.

The problem is that we are assigning the output of getchar() to an unsigned char and then comparing it with a negative value; that comparison will always be false, resulting in the in nite loop.

Without changing the Make le, change the declaration for c so as to avoid the in nite loop. If you now run the program:

$ ./stdin-char-count &lt;stdin-char-count.c

it should output the number of characters in stdin-char-count.c. Compare your result with that from wc stdin-char-count.c.

<h1>1.5          File Character Count</h1>

Change over to the le-char-count directory. Look at the program in the lechar-count.c le. It outputs a count of the number of characters in the le speci ed by its single command-line argument.

The key function is <a href="http://man7.org/linux/man-pages/man3/fopen.3.html">fopen().</a> It is used to open the le with name speci ed by its rst argument for reading as speci ed by its second “r” mode argument. If the open succeeds, fopen() returns a non-NULL pointer to a FILE ADT. This FILE pointer can be used by other functions for performing operations on the le.

Compile the program using make. Run it on itself:

$ ./file-char-count file-char-count.c

Compare the result with that produced by wc.

Check to see if the program handles non-existent           les:

$ ./file-char-count xxx

It seems to run ok, but has a major bug. Can you spot the bug?

If you cannot spot the bug, try running the program using <a href="http://valgrind.org/">valgrind.</a>

$ valgrind ./file-char-count file-char-count.c

and you will see that the program is leaking memory. So it is allocating memory at some point which is never being released. Try to gure out where that is and x the bug; test using valgrind.

<h1>1.6        File Copy</h1>

Change over to the le-copy directory. Look at the program in the le-copy.c le. It copies the le speci ed by its rst command-line argument into the le

speci ed by its second command-line argument.

Again, the key function is <a href="http://man7.org/linux/man-pages/man3/fopen.3.html">fopen().</a> Not only is it being used to open the le with name speci ed by the rst command-line argument for reading, but it is also being used to open the le with name speci ed by the second command-line argument for writing. In the rst fopen(), the mode argument is speci ed as “r”, but the second fopen() has its mode argument speci ed as “w”.

Compile the program using make. Run it on itself and verify that it makes a correct copy:

$ ./file-copy                                                    #should output usage

$ ls -l file-copy.c t                                            #t should not be there

$ ./file-copy file-copy.c t

$ ls -l file-copy.c t #t be there with same char count $ cmp file-copy.c t  #should compare ok

Notice that t got created.

Now use your file-copy program to copy some other                le to t.

$ ./file-copy Makefile t

$ cmp Makefile t                                             #should compare ok

Set up the destination          le so that it cannot be written into:

$ chmod a-w t       #turn-off write perms for t $ ./file-copy file-copy.c t #should fail in the fopen(“w”)

$ cmp Makefile t                                         #should compare ok as t unchanged

The program would not have noticed the fact that the fopen() for writing failed if we had not checked the return value from the fopen(). If you look at the code for the function, notice that almost all library calls are checked for errors; that is absolutely necessary for writing good quality code which fails fast when encountering an error situation.

Unfortunately, the file-copy program does not do a complete job of error checking. If you look at the <a href="http://man7.org/linux/man-pages/man3/fgetc.3.html">man page</a> for fgetc(), the return value is documented as “EOF on end of le or error”. This means that the while-loop in the doCopy() function will terminate on either a real end-of- le or on error. Add code to doCopy() to have the program terminate when an error occurs on fgetc(). Hint: look at the <a href="http://man7.org/linux/man-pages/man3/fileno.3.html">man page</a> for ferror().

<h1>1.7          Appending To A File</h1>

Stay in the le-copy directory.

Using the “w”-mode argument to the second fopen() will clobber the contents of the le if it already exist. To avoid this, open the le for append by changing the mode argument from “w” to “a”. Compile and test.

$ rm -f t                                                              #ensure t not present

$ ./file-copy file-copy.c t #create empty t and append filecopy.c

$ ./file-copy Makefile t                                    #append Makefile to t

So the destination le t should contain the contents of Makefile appended to the contents of file-copy.c. Verify using a text editor. You can also verify that the sum of the line counts of the two source les match the total number of lines in t:

$ wc -l file-copy.c Makefile $ wc -l t

<h1>1.8        Bu ering</h1>

When a function such as fputc() writes a character, the path taken by the data is as follows:

<ol>

 <li>Normally, fputc() does not do any I/O. Instead it is set up to write the character into a bu er in memory. This bu er is controlled by the stdio library and is in <a href="https://en.wikipedia.org/wiki/User_space">user space</a><a href="https://en.wikipedia.org/wiki/User_space">.</a></li>

 <li>When the bu er becomes full, the contents of the bu er is written from the user space bu er to a kernel space bu er using a OS call like <a href="http://man7.org/linux/man-pages/man2/write.2.html">write(). </a>Since this involves calling the OS, it can be quite slow compared to a normal memory write by one or two orders of magnitude.</li>

 <li>When the kernel bu er becomes full, it is actually written out to the le. This I/O is extremely slow compared to normal memory writes by several orders of magnitude.</li>

</ol>

When reading a character using fgetc() the data ows in the other direction: from the le to the kernel bu er and then into a stdio bu er using <a href="http://man7.org/linux/man-pages/man2/read.2.html">read()</a> and nally into the program using a function like fgetc().

Without root access, we cannot control (3). However, the stdio library does allow us to control the stdio bu er using <a href="http://man7.org/linux/man-pages/man3/setbuf.3.html">setvbuf()</a> and friends.

Change over to the no-bu er-copy directory and look at the no-bu er-copy.c le. The program uses an optional extra command-line argument: stdio bu ering is turned on i that extra argument is speci ed and equal to 1. Compile and measure the di erence when copying the gcc executable into the <a href="http://man7.org/linux/man-pages/man4/null.4.html">/dev/null</a> data sink:

$ time ./no-buffer-copy ‘which gcc‘ /dev/null 1

$ time ./no-buffer-copy ‘which gcc‘ /dev/null 0

You should see an appreciable di erence in performance.

<h1>1.9        Record I/O</h1>

Change over to the rec-io directory and look at the gen-rand-points.c. This program generates a number (given by its rst argument) of random 2-dimensional points with coordinates in [0, 1000], while writing them to a le (given by its second argument) in binary.

Build the program by typing make gen-rand-points and run it.

$ ./gen-rand-points 100 points.dat

It will print out the average magnitude of all the generated points.

Look at the generated points.dat le using a text editor. You should see that it looks like garbage as it contains the binary representation of the points.

Each point is written out to the le using <a href="http://man7.org/linux/man-pages/man3/fwrite.3p.html">fwrite().</a> Look at its documentation to understand what it does.

Given this binary dump of the points, it is possible to read back the points using <a href="http://man7.org/linux/man-pages/man3/fread.3p.html">fread().</a> Build the stat-points program and run it on the generated points in points.dat:

$ ./stat-points points.dat

It will print out statistics about the magnitude of the points. Note that the average should match the average printed out by the gen-rand-points program.

The stat-points program reads each point in to a dynamically grown array and sorts the array in order to determine the min, max and median. The code in dyn-array.h provides a speci cation for a dynamic array ADT and dyn-array.c provides its implementation.

These programs shows that using fread() and fwrite() it is possible to dump out binary data. However, the binary data has severe portability problems:

<ul>

 <li>If points.dat was written out on a little-endian system and read back on a big-endian system, the results would be garbage.</li>

 <li>If points.dat was read back in on a system using a di erent int size than the one on which it was written, then the results would be garbage.</li>

 <li>Even if points.dat was read and written on the same system, but the reading and writing programs were compiled using separate compilers, or the same compiler with di erent options, the result could be garbage (this is likely to be prevented because of ABI compatibility reasons).</li>

</ul>

Nevertheless, with gen-rand-points and stat-points running on the same system, they seem to work. Unfortunately, they contain a major bug.

By default, when a le is opened by the C library, it is assumed that it is a text le containing a sequence of lines. However, that depends on the de nition of lines which is system-dependent:

<ul>

 <li>The C libraries assume that a line is a maximum sequence of characters not containing a newline ’
’ character and terminated by a newline.</li>

 <li>Under Unix systems, a line is a maximum sequence of characters not containing a linefeed ’
’ character and terminated by a linefeed.</li>

 <li>Under Windows, a line is a maximum sequence of characters not containing the sequence carriage-return ’r’ line-feed ’
’ and terminated by the two character sequence carriage-return, line-feed.</li>

 <li>Under classic Mac-OS systems (not OS/X which is basically Unix), a line is a maximum sequence of characters not containing a carriage-return ’r’ character and terminated by a carriage-return.</li>

</ul>

When doing I/O on text les, the C library I/O routines translates between system line-endings and the C line-endings. On Unix, this translation is the identity function but is non-trivial on other systems. For example, the character sequence ’r’ ’
’ in a Windows le is read into a C program as a single character ’
’.

Hence doing I/O of binary data using text les is wrong. It will work ne under Unix, but will be incorrect under Windows. For example, the integer 10 (which is the Ascii code for 
 will be output as ’r’ followed by a ’
’.

The x for this is to open the les for binary I/O; this can be done by adding a b to the second mode string argument of fopen().

<ol>

 <li>Fix the modes in the gen-rand-points and stat-points program.</li>

 <li>Add an optional second le-name argument to stat-point. If that argument is provided, the statistics should be appended to that le instead of being written on standard output. Test and ensure that your modi ed program is valgrind-clean.</li>

</ol>