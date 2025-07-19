# VSD-TCL-Projects
# Projects/Overview

# Implement TCL scripts for PD flow - constraints/report generation for openMSP430

## Index

## Day 1 - Introduction to TCL and VSDSYNTH Toolbox Usage
Basics Introduction/VSDSYNTHBOX tool usage

## Day 2 - Variable Creation and Processing Constraints from CSV
CSV to format[1] and SDC - Variable Creation
CSV to format[1] and SDC - Processing constraints, CSV

## Day 3 - Processing Clock and Input Constraints
Processing clock constraints
Processing input constraints

## Day 4 - Complete Scripting and Yosys Synthesis Introduction
Full script for donwload and conclusion
Introduction to Yosys synthesis tool usage
Hierarchy check and error handling script creation for Yosys

## Day 5 - Advanced Scripting Techniques and Quality of Results Generation
 Synthesis main file scripting and output file editing
 World of 'Procs'
 Interpret clock generation constraints
 Interpret IO delays and transition constraints
 Process bussed ports and configuration file creation for Opentimer
 Quality of results (QoR) generation algorithm



# Day 1 Module

## Description

Create shell script to link design_details.csv (*.v, .lib, clk/port constraints & o/p directories) with a tcl box to generate necessary constraints/reports

--> Vim shell script shabang -
```
#!/bin/tcsh -f
```
--> Logo for fun! & set current working directory -

```
echo " #    #    #    ######  ####### #    # ####  #    #  "
echo " #   #    # #   #     #    #    #    #  #    #   #   "
echo " #  #    #   #  #     #    #    #    #  #    #  #    "
echo " ###    #     # ######     #    ######  #    ###     "
echo " #  #   ####### #   #      #    #    #  #    #  #    "
echo " #   #  #     # #    #     #    #    #  #    #   #   "
echo " #    # #     # #     #    #    #    # ####  #    #  "
echo
echo ""
echo ""
echo "           VSD TCL Workshop Scripting - by Karthik "
echo "           Guidance - Kunal Ghosh "
echo ""
echo ""
echo ""

```
```
set myworkdir = 'pwd'
```

--> Implement a check for whether a valid csv is passed as an argument with this shell script , or if the user appends -help 

```
if ($#argv != 1) then
	   echo "Info : Please Provide the csv file"
	   exit 1
 endif
 if (! -f $argv[1] || $argv[1] == "-help") then
	if ($argv[1] == "-help") then
		echo USAGE: ./vsdsynth \<csv file\>
		echo
		echo 		where \<csv file\> consists of 2 columns
		echo
		echo           	\<Design Name\> is the name of top level module 
		echo
		echo            \<Output Directory\> is the name of Output Directory where you want to dump synthesis script, synthesized netlist and timing reports
		echo 
		echo             \<Early Libary Path\> is the file path of the early cell libary to be used for STA
		echo
                echo             \<Late Libary Path\> is the file path of the late cell libary to be used for STA
		echo
                echo             \<Constraints file\> is the file path of constraints to be used for STA
	else 
		echo "ERROR: Cannot find the CSV file $argv[1]. Exiting"
	endif
else
		tclsh vsdsynth.tcl  $argv[1]
endif
```

--> vsdsynth.tcl is our tcl box, to which first argument passed to shell script is provided

## Command line Demo

<img width="777" height="416" alt="image" src="https://github.com/user-attachments/assets/a16adca3-83ef-40c7-85e1-4c963fb87020" />

<img width="782" height="588" alt="image" src="https://github.com/user-attachments/assets/84abf15c-1d86-4539-aee3-5c89986e033c" />

<img width="732" height="80" alt="image" src="https://github.com/user-attachments/assets/1c068404-c76f-4def-b564-8344512cfc0a" />

# Day 2 Module

## Description

Converting design_details.csv to format1 and constraints.csv to sdc - basic variables processing

--> openMSP430_design_details.csv looks like this -

<img width="772" height="591" alt="image" src="https://github.com/user-attachments/assets/2194efe2-509c-4b6e-a2a0-84a8d991a324" />

--> openMSP430_design_constraints.csv looks like this -

<img width="1857" height="878" alt="image" src="https://github.com/user-attachments/assets/c00064c1-a90a-4021-a252-795c835115c1" />

--> creating variables for the directories/files in details.csv

This involves converting the csv to matrix object m, and further into an array - this makes it easy to access elements in TCL through lindex command

```

    set filename [lindex $argv 0]
    package require csv
    package require struct::matrix
    struct::matrix m
    set f [open $filename]
    csv::read2matrix $f m , auto
    close $f
    set columns [m columns]
    #m add columns $columns
    m link my_arr
    set rows [m rows]

```

--> Further for details.csv we remove the space in first column and denote them as directory names, to which the second column value is passed

```
        set i 0
        while {$i < $rows} {
                 puts "\nInfo: Setting $my_arr(0,$i) as '$my_arr(1,$i)'"
                 if {$i == 0} {
                         set [string map {" " ""} $my_arr(0,$i)] $my_arr(1,$i)
                 } else {
                         set [string map {" " ""} $my_arr(0,$i)] [file normalize $my_arr(1,$i)]
                 }
                  set i [expr {$i+1}]
        }
        puts "\nInfo: Below are the list of the initial variables and their values. User can use these variables for further debugging purposes."
        puts "DesignName = $DesignName"
        puts "OutputDirectory = $OutputDirectory"
        puts "NetlistDirectory = $NetlistDirectory"
        puts "EarlyLibraryPath = $EarlyLibraryPath"
        puts "LateLibraryPath = $LateLibraryPath"
        puts "ConstraintsFile = $ConstraintsFile"
```

--> Create a check for directory/file exists. For directory - if it doesnt exist - use mkdir and create

```
if { [file isdirectory $OutputDirectory]} {
	puts "\nInfo : Output directory exists and found in path $OutputDirectory "
} else {
	puts "\n Info : Cannot find the output directory $OutputDirectory. Creating $OutputDirectory"
	file mkdir $OutputDirectory
}
if { [file isdirectory $NetlistDirectory]} {
	puts "\nInfo : Netlist directory exists and found in path $NetlistDirectory "
} else {
	puts "\n Info : Cannot find the Netlist directory $NetlistDirectory. Creating $NetlistDirectory"
	file mkdir $NetlistDirectory
}
if { [file exists $EarlyLibraryPath]} {
	puts "\nInfo : Early cell library file exists and found in path $EarlyLibraryPath "
} else {
	puts "\n Info : Cannot find the early cell library file in path $EarlyLibraryPath. Exiting..."
	exit
}
if { [file exists $LateLibraryPath]} {
	puts "\nInfo : Late cell library file exists and found in path $LateLibraryPath "
} else {
	puts "\n Info : Cannot find the Late cell library file in path $LateLibraryPath. Exiting..."
	exit
}
if { [file exists $ConstraintsFile]} {
	puts "\nInfo : Constraints file exists and found in path $ConstraintsFile "
} else {
	puts "\n Info : Cannot find the Constraints file in path $ConstraintsFile. Exiting..."
	exit
}
```
--> For constraints.csv file, as we saw there are three sections - clock, input port and output port constraints. The same methos is followed for readability using tcl - convert csv to matrix to array & use lindex. Main point is to find the starting rows/columns for these sections for efficient processing 

```
puts "\n Info: Dumping SDC constraints for $DesignName"
::struct::matrix constraints
set chan [open $ConstraintsFile]
csv::read2matrix $chan constraints , auto
close $chan
set constr_rows [constraints rows]
puts " number of rows in constraints file = $constr_rows "
set constr_columns [constraints columns]
puts " number of columns in constraints file = $constr_columns "


set clock_start [lindex [lindex [constraints search all CLOCKS] 0] 1]
set clock_start_column [lindex [lindex [constraints search all CLOCKS] 0] 0]
puts "clock_start =  $clock_start"
puts "clock_start_column = $clock_start_column"

set input_ports_start [lindex [lindex [ constraints search all INPUTS] 0] 1]
puts "input_ports_start =$input_ports_start"
set output_ports_start [lindex [lindex [ constraints search all OUTPUTS] 0] 1]
puts "output_ports_start = $output_ports_start "
```

## Command line Demo

Executing the current tcl script 

<img width="866" height="282" alt="image" src="https://github.com/user-attachments/assets/50e4a387-1ef9-43ea-b21d-ef522ed4e2ff" />

<img width="1912" height="922" alt="image" src="https://github.com/user-attachments/assets/414d8d46-ed27-4693-95a9-faab99de915e" />















