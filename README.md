# WordWrapper
Jeevan Vasanthan and Spandan Parikh
(jgv36)           &  (snp131)

The program we wrote, ww.c, contains eight methods: queue_init(struct Queue *q), enqueue(struct Queue *q, char *name), dequeue(struct Queue *q), readDir(void *arg), wrapFiles(void *arg), wrap(int width, int fd_input, fd_output), isFileOrDir(char *name), and main(int argc, char argv[]).

queue_init(struct Queue *q) is a void method that takes in a struct queue and initializes its values such as start, end, and numActiveThreads = 0. This method is used in the main method to initialize dirQueue and fileQueue.

enqueue(struct Queue *q, char *name) is a void method that has two paramertes, first a queue and pathname. It's purpose is add the pathname into the specified queue whether its file or a directory.

dequeue(struct Queue *q) only has a queue as a parameter and returns a char. It's purpose is to remove pathname from the specified queue and return it.

isFileOrDir(char *name) is a helper method that takes in the inputted file name. Utilizing S_ISDIR and S_ISREG to determine whether it's a file (returns 1), or a directory(returns 2), and returns 0 otherwise.

readDir(void *arg) is the directory worker method, it takes in a void *arg argument used for threading. It performs recursive directory traversal: traverses through directories and its subdirectories and utilizing isFileOrDir. Adding pathname to dirQueue within subdirectory if its a directory or adding it to fileQueue if its a file.

wrapFiles(void *arg) is the file worker method, it also takes in a void *arg argument used for threading. It performs multithreaded wrapping. Travesing through fileQueue and performing wrapping operations, such as determining if wrapping is allowed and utilizing the wrap method.

The main() method does some of the error checking, such as the width parameter being valid (> 0), if there is a file name/path inputted, the main method calls on to the isFileOrDir(). If it's determined to be a file, the file is opened and the wrap() is called to "normalize" file. If it's determined to be a directory, then it opens and changes to the inputed directory, compares the file name with "wrap." to determine if wrapping should be allowed. If it's allowed, the file is opened to read and the wrap() method is called to "normalize" the file. Then a new file is written into. If neither than it takes in the standard input and calls wrap. Additionally, the main method takes additional command line arguments such as numDirThreads and numWrapThreads which are initialized. numWrapThreads are used in wrapFiles, the file worker function to wrap files in the directory and any subdirectories and numDirThreads is used in readDirs, the directory worker, to traverse through directories. 

TEST CASES:
p3 stuff:
Test Case 0: Testing concurrent wrapping with directory traversal, -rN,M

	> p3 (working directory)
		> foo
			> bar.txt
			> baz
				> quux.txt
	
	Input: ./ww -r1,2 20 foo

	Output: program works as intended, foo now contains wrap.bar.txt and its subdirectory, baz, containes the normalized wrap.quux.txt
	> p3 (working directory)
		> foo
			> bar.txt
			> wrap.bar.txt
			> baz
				> quux.txt
				> wrap.quux.txt

Test Case 1: Testing  with directories containing subdirectories with no files, ensuring file threads are being terminated without even using it to wrap files

	> p3 (working directory)
		> foo
			> bar.txt
			> wrap.bar.txt
			> baz
				> quux.txt
				> wrap.quux.txt
		> foo2
			> fooTest
			> fooTest2

	Input: ./ww -r 20 foo2

	Output: works as intended, no erros thrown

Test Case 2: Testing with directories containing two subdirectories: one with no files and another with invalid files

	> p3 (working directory)
		> foo
			> bar.txt
			> wrap.bar.txt
			> baz
				> quux.txt
				> wrap.quux.txt
			> foo2
				> fooTest
				> fooTest2
					> .stuff.txt
					> wrap.stuff.txt

	Input: ./ww -r3 20 foo2

	Output: Works as intended; no errors regarding threads or file thrown

Test Case 3: Testing with directories with multiple subdirectories, two filled with regular files and one with nothing in it. Ensuring that the dirThreads are not set to 0 when the dirQueue is empty

	> p3 (working directory)
		> foo
			> bar.txt
			> wrap.bar.txt
			> baz
				> quux.txt
				> wrap.quux.txt
			> foo2
				> fooTest
				> fooTest2
					> fooTest
						> .stuff.txt
						> wrap.stuff.txt
					> fooTest2
			> foo3
				> foo3Test
					> stuff.txt
					> stuff2.txt
					> stuff3.txt
					> .stuff.txt
					> wrap.y.txt
				> foo3Test2
				> foo3Test3
					> text.txt
					> wrap.y.txt

	Input: ./ww -r5,5 20 foo3

	Output: works as intended, even when the dirqueue for subdirectory, foo3Test2 was empty. It still traversed to foo3Test3 and perfomred wrap operations. Ignoring invalid files and normalizing regular files

	> p3 (working directory)
		> foo
			> bar.txt
			> wrap.bar.txt
			> baz
				> quux.txt
				> wrap.quux.txt
			> foo2
				> fooTest
				> fooTest2
					> fooTest
						> .stuff.txt
						> wrap.stuff.txt
					> fooTest2
			> foo3
				> foo3Test
					> stuff.txt
					> stuff2.txt
					> stuff3.txt
					> .stuff.txt
					> wrap.y.txt
					> wrap.stuff,txt
					> wrap.stuff2.txt
					> wrap.stuff3.txt
				> foo3Test2
				> foo3Test3
					> text.txt
					> wrap.y.txt
					> wrap.text.txt

Test Case 4: Testing with a directory containing multiple subdirectories and those containing subdirectories with files of valid an invalid types.

	> p3 (working directory)
		> foo
			> bar.txt
			> wrap.bar.txt
			> baz
				> quux.txt
				> wrap.quux.txt
			> foo2
				> fooTest
					> .stuff.txt
					> book.txt
					> wrap.stuff.txt
					> fooSub
						> text.txt
					> fooSub2
						> test.txt
				> fooTest2
					> stuff.txt
					> fooSub3
						> test.txt
			> foo3
				> foo3Test
					> stuff.txt
					> stuff2.txt
					> stuff3.txt
					> .stuff.txt
					> wrap.y.txt
					> wrap.stuff,txt
					> wrap.stuff2.txt
					> wrap.stuff3.txt
				> foo3Test2
				> foo3Test3
					> text.txt
					> wrap.y.txt
					> wrap.text.txt
	Input: ./ww -r7,7 20 foo2

	Output: wrapped successfully without errors
	> p3 (working directory)
		> foo
			> bar.txt
			> wrap.bar.txt
			> baz
				> quux.txt
				> wrap.quux.txt
			> foo2
				> fooTest
					> .stuff.txt
					> book.txt
					> wrap.stuff.txt
					> wrap.book.txt
					> fooSub
						> text.txt
						> wrap.text.txt
					> fooSub2
						> test.txt
						> wrap.test.txt
				> fooTest2
					> stuff.txt
					> wrap.stuff.txt
					> fooSub3
						> test.txt
						> wrap.test.txt
			> foo3
				> foo3Test
					> stuff.txt
					> stuff2.txt
					> stuff3.txt
					> .stuff.txt
					> wrap.y.txt
					> wrap.stuff,txt
					> wrap.stuff2.txt
					> wrap.stuff3.txt
				> foo3Test2
				> foo3Test3
					> text.txt
					> wrap.y.txt
					> wrap.text.txt
Test Case 5: Tests with negative theads
	
	Input: ./ww -r10,-10 20 foo4

	> p3 (working directory)
	> foo
		> bar.txt
		> wrap.bar.txt
		> baz
			> quux.txt
			> wrap.quux.txt
		> foo2
			> fooTest
				> .stuff.txt
				> book.txt
				> wrap.stuff.txt
				> wrap.book.txt
				> fooSub
					> text.txt
					> wrap.text.txt
				> fooSub2
					> test.txt
					> wrap.test.txt
			> fooTest2
				> stuff.txt
				> wrap.stuff.txt
				> fooSub3
					> test.txt
					> wrap.test.txt
		> foo3
			> foo3Test
				> stuff.txt
				> stuff2.txt
				> stuff3.txt
				> .stuff.txt
				> wrap.y.txt
				> wrap.stuff,txt
				> wrap.stuff2.txt
				> wrap.stuff3.txt
			> foo3Test2
			> foo3Test3
				> text.txt
				> wrap.y.txt
				> wrap.text.txt
		> foo4
			> hello.txt

	Output: Does nothing since the second directory thread was negative.			

p2 stuff:
Test Case 0: Tests if the inputed width is valid (>0)

	./ww -30 test0.txt > test0_out
	Input: 
	This is very good text that has been put into a file for the purposes
	of being
	an
	example.
		
	That       is
		good.

	Output: Program should abort, due to an assert statement (in main). As width was not inputted properly. 
	ww: ww.c:312: main: Assertion `width > 0' failed.
	Aborted

Test Case 1: Tests with no more than one whitespace between words and without paragraph breaks.

	./ww 20 test1.txt > test1_out
	Input:
	This is very good text that has been put into a file for the purposes
	of being
	an
	example.
		
	That       is
		good.

	Output:
	This is very good
	text that has been
	put into a file for
	the purposes of
	being an example.

	That is good.

Test Case 2: Tests normalization, if two input files consisting of the same words in the
same paragraphs will produce identical output when wrapped to the same width.

	./ww 20 test1.txt > test1_out; from previous test
	./ww 20 test1_out > test2_out
	Input:
	test1_out; Output from the previous test

	Output: test2_out
	
	Input2: cmp test1_out test2_out; theoretically, both files should be the same. Thus, cmp should return nothing.

	Output: cmp prints nothing; program works as intended.

Test Case 3: Tests with an input with more than one whitespace between words and paragraph breaks with multiple blank lines.

    	./ww 20 test3.txt > test3_out
	Input:
	This is very good text that has been put into a file for the purposes
	of being
	an
	example.
		

	That       is
   			good.

	Output:
	This is very good
	text that has been
	put into a file for
	the purposes of
	being an example.

	That is good.

Test Case 4: Tests with an input with no blank lines between paragraphs.

   	./ww 20 test4.txt > test4_out
	Input:
	This is very good text that has been put into a file for the purposes
	of being
	an
	example.

	Output:
	This is very good
	text that has been
	put into a file for
	the purposes of
	being an example.

Test Case 5: Tests with an input with characters equal to width size.

    ./ww 20 test5.txt > test5_out
    Input:
    ttttt ttttt ttttt tt

    Output:
    ttttt ttttt ttttt tt

Test Case 6: Tests with an input with multiple words > width size

    ./ww 20 test6.txt > test6_out
    Input:
    Good evening, this is a particularly interesting project 
    called WordWrapperinSystemProgramming
    and it is goind well thus far. Created by
    SpandanParikhandJeevanVasanthan. 
    Have a good day.

    Output:
	Good evening, this
	is a particularly
	interesting project
	calledWordWrapperinSystemProgramming
	and it is goind well
	thus far. Created bySpandanParikhandJeevanVasanthan.
	Have a good day.

Test Case 7: Tests with an input that has whitespace characters at the beginning of a new line

    ./ww 20 test7.txt > test7_out
    Input:
	This is very good text that has been put into a file for the purposes
		of being
			an
		example.

    Output:
	This is very good
	text that has been
	put into a file for
	the purposes of
	being an example

Test Case 8: Tests with an input of a different width length

    ./ww 30 test8.txt > test8_out
    Input:
    This is very good text that has been put into a file for the purposes
    of being
    an
    example.
		
    That       is
       	good.

    Output:
    This is very good text that
    has been put into a file for
    the purposes of being an
    example.

    That is good.

Test Case 9: Testing with an inputted directory, test_dir, containing files t1.txt, t2.txt
.dontrun.txt wrap.yrun.txt; Testing if wrap run on authorized files and does nothing to files
starting with "." and/or "wrap."

	> p2 (working directory)
		> src
			> test_dir
				> t1.txt
				> t2.txt
				> .dontrun.txt
				> wrap.yrun.txt
			> Makefile
			> book.txt
			> ww.c
			> ww (exec file)
			> README.md (README file)
			> test1.txt
			> test3.txt
			> test4.txt
			> test5.txt
			> test6.txt
			> test7.txt
			> test8.txt

	Input: ./ww 30 test_dir

	Output: Program works as intended, test_dir now contains additional files: wrap.t1.txt and t2.txt

	> p2 (working directory)
		> test_dir
			> t1.txt
			> t2.txt
			> .dontrun.txt
			> wrap.t1.txt
			> wrap.t2.txt
			> wrap.yrun.txt

Test 10: Testing with a large buffer

	./ww 80 book.txt > test10_out
	Input: 
	Buses 

	Running throughout the system is a collection of electrical conduits called buses that carry bytes of information back and forth between the components. Buses
	are typically designed to transfer fixed-sized chunks of bytes known as words. The number of bytes in a word (the word size) is a fundamental system parameter 
	that varies across systems. For example, Intel Pentium systems have a word size of 4 bytes, while serverclass systems such as Intel Itaniums and Sun SPARCS    
	have word sizes of 8 bytes. Smaller systems that are used as embedded controllers in automobiles and factories can have word sizes of 1 or 2 bytes. For
	simplicity, we will assume a word size of 4 bytes, and we will assume that buses transfer only one word at a time. 

	I/O devices 

	Input/output (I/O) devices are the system’s connection to the external world. Our example system has four I/O devices: a keyboard and mouse for user input, a
	display for user output, and a disk drive (or simply disk) for long-term storage of data and programs. Initially, the executable hello program resides on the
	disk. Each I/O device is connected to the I/O bus by either a controller or an adapter. The distinction between the two is mainly one of packaging. Controllers
	are chip sets in the device itself or on the system’s main printed circuit board (often called the motherboard). An adapter is a card that plugs into a slot on
	the motherboard. Regardless, the purpose of each is to transfer information back and forth between the I/O bus and an I/O device. Chapter 6 has more to say
	about how I/O devices such as disks work. And in Chapter 12, you will learn how to use the Unix I/O interface to access devices from your application programs.
	We focus on an interesting class of devices known as networks, but the techniques generalize to other kinds of devices as well.

	Output: 
	Buses

	Running throughout the system is a collection of electrical conduits called
	buses that carry bytes of information back and forth between the components.
	Buses are typically designed to transfer fixed-sized chunks of bytes known as
	words. The number of bytes in a word (the word size) is a fundamental system
	parameter that varies across systems. For example, Intel Pentium systems have a
	word size of 4 bytes, while serverclass systems such as Intel Itaniums and Sun
	SPARCS have word sizes of 8 bytes. Smaller systems that are used as embedded
	controllers in automobiles and factories can have word sizes of 1 or 2 bytes.
	For simplicity, we will assume a word size of 4 bytes, and we will assume that
	buses transfer only one word at a time.

	I/O devices

	Input/output (I/O) devices are the system’s connection to the external world.
	Our example system has four I/O devices: a keyboard and mouse for user input, a
	display for user output, and a disk drive (or simply disk) for long-term storage
	of data and programs. Initially, the executable hello program resides on the
	disk. Each I/O device is connected to the I/O bus by either a controller or an
	adapter. The distinction between the two is mainly one of packaging. Controllers
	are chip sets in the device itself or on the system’s main printed circuit
	board (often called the motherboard). An adapter is a card that plugs into a
	slot on the motherboard. Regardless, the purpose of each is to transfer
	information back and forth between the I/O bus and an I/O device. Chapter 6 has
	more to say about how I/O devices such as disks work. And in Chapter 12, you
	will learn how to use the Unix I/O interface to access devices from your
	application programs. We focus on an interesting class of devices known as
	networks, but the techniques generalize to other kinds of devices as well.
	
Test 11: Tests if program ignores a subdirectory

	Input: 
	> p2 (working directory)
		> src
			> test_dir
				> testDir2
					> testFile.txt
				> t1.txt
				> t2.txt
				> .dontrun.txt
				> wrap.yrun.txt
			> Makefile
			> book.txt
			> ww.c
			> ww (exec file)
			> README.md (README file)
			> test1.txt
			> test3.txt
			> test4.txt
			> test5.txt
			> test6.txt
			> test7.txt
			> test8.txt

	Input: ./ww 30 test_dir

	Output: Program works as intended, test_dir now contains additional files: wrap.t1.txt and t2.txt and it also ignores testDir2 along 
	with .dontrun.txt and wrap.yrun.txt

	> p2 (working directory)
		> test_dir
			> testDir2
				> testFile.txt
			> t1.txt
			> t2.txt
			> .dontrun.txt
			> wrap.t1.txt
			> wrap.t2.txt
			> wrap.yrun.txt
			
Test 12: Tests Standard input and output.
	
	Input: ./ww 20
	This is very good text that has been put into a file for the purposes of bein an example.
	
	Output:
	This is very good
	text that has been
	put into a file for
	the purposes of bein
	an example.
