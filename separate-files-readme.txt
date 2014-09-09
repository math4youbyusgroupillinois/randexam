This version of randexam has a new option
MULTIPLE_EXAM_FILES
This is initially false, so randexam proc-lib will still generate a single massive exam.tex (but now in a subdirectory 'print')

Setting this option to true will generate a single .tex file per exam

The rest of this readme discusses a couple of ways I generate pdfs and print all of these pdfs.
I have not yet created python code for these tasks.

I've used two ways to generate the pdfs:
1. A simple shell script
2. A Makefile (that is faster when you have 400 exams to convert)


1. Shell script:

#!/bin/bash
cd print
for i in *.tex; do /usr/texbin/pdflatex $i;done

2. Makefile:

The following Makefile is great because make can do jobs in parallel.
e.g. If you have 8CPU machine:

> cd print
> make -j8     # Run 8 jobs in parallel



SOURCES=$(wildcard *tex)
EXAM_PDF_FILES=$(SOURCES:.tex=.pdf)

all: $(EXAM_PDF_FILES)

#Remember! Makefiles require the commands to be indented with a tab not a space

%.pdf : %.tex
	rm -f $@ 
	pdflatex  $< 



----
Printing
So now you need a way to print all of these to a printer or two.
Enter CUPS. You'll need to learn how to set the printer-specific options to enable/disable stapling and double-sided.
This is what worked on my Mac that already had a printer queue name '_1228' defined to a printer in room 1228:
First, find out your printer queue and its option:

lpstat -p    # list printer queues
printer _1228 is idle.
printer _2203 is idle.

Get a big list of printer options for your printer of choice (-l is important):
lpoptions -p _1228 -l

Duplex/2-Sided Printing: None *DuplexNoTumble DuplexTumble
XRXStapling/Stapling: *None Front Dual Rear
... other options

Each line shows the short key, description and possible values. The default value is shown with an *
Now we can use this to set the output. Here's a script that works for me (on a Mac)

#!/bin/bash
#Typical duplex options for Siebel printers:  Duplex=DuplexNoTumble Duplex=None
EXAMPREFIX=cs241_02_cfork_exams
#Example: Print out the first  200 exams (#1 to #200):
for i in `seq 1 200` ; do 
  echo $i ;
  lpr -P _1228 -o  XRXStapling=Front -o Duplex=DuplexNoTumble ${EXAMPREFIX}_${i}.pdf
done





