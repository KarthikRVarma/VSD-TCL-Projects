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


# Day 3 Module

## Description & Command line Demos

Converting the current clock & IO constraints in constraints.csv to an .sdc file format that is consumable by the synthesis tool.

--> Identify column numbers and indices for the clock, input and output port constraints, then looping through the elements in each section and mapping the value to the parameter. Further it will be converted to the tool command form.

--> For clock constriants its a smaller matrix and straightforward as seen in the csv.
```
#SDC constraints

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


set clock_early_rise_delay_start [lindex [lindex [ constraints search rect $clock_start_column $clock_start [expr {$constr_columns -1}] [expr {$input_ports_start -1}] early_rise_delay] 0 ] 0 ]
set clock_early_fall_delay_start [lindex [lindex [ constraints search rect $clock_start_column $clock_start [expr {$constr_columns -1}] [expr {$input_ports_start -1}] early_fall_delay] 0 ] 0 ]
set clock_late_rise_delay_start [lindex [lindex [ constraints search rect $clock_start_column $clock_start [expr {$constr_columns -1}] [expr {$input_ports_start -1}] late_rise_delay] 0 ] 0 ]
set clock_late_fall_delay_start [lindex [lindex [ constraints search rect $clock_start_column $clock_start [expr {$constr_columns -1}] [expr {$input_ports_start -1}] late_fall_delay] 0 ] 0 ]
set clock_early_rise_slew_start [lindex [lindex [ constraints search rect $clock_start_column $clock_start [expr {$constr_columns -1}] [expr {$input_ports_start -1}] early_rise_slew] 0 ] 0 ]
set clock_early_fall_slew_start [lindex [lindex [ constraints search rect $clock_start_column $clock_start [expr {$constr_columns -1}] [expr {$input_ports_start -1}] early_fall_slew] 0 ] 0 ]
set clock_late_rise_slew_start [lindex [lindex [ constraints search rect $clock_start_column $clock_start [expr {$constr_columns -1}] [expr {$input_ports_start -1}] late_rise_slew] 0 ] 0 ]
set clock_late_fall_slew_start [lindex [lindex [ constraints search rect $clock_start_column $clock_start [expr {$constr_columns -1}] [expr {$input_ports_start -1}] late_fall_slew] 0 ] 0 ]

set sdc_file [open $OutputDirectory/$DesignName.sdc "w"]
set i [expr {$clock_start+1}]
set end_of_ports [expr {$input_ports_start -1}]


puts "\n Info-SDC: Working on clock constraints..."



while {$i< $end_of_ports} {
        #puts "Working on clock [constraints get cell 0 $i ]"
puts -nonewline $sdc_file "\ncreate_clock -name [constraints get cell 0 $i ] -period [constraints get cell 1 $i ] -waveform \{0 [expr { [constraints get cell 1 $i ]*[constraints get cell 2 $i ]/100 }]\} \[get_ports [constraints get cell 0 $i ]\]"
        puts -nonewline $sdc_file "\nset_clock_latency -source -early -rise [constraints get cell $clock_early_rise_delay_start $i ] \[get_clocks [constraints get cell 0  $i]\]"
        puts -nonewline $sdc_file "\nset_clock_latency -source -early -fail [constraints get cell $clock_early_fall_delay_start $i ] \[get_clocks [constraints get cell 0  $i]\]"
        puts -nonewline $sdc_file "\nset_clock_latency -source -late -rise [constraints get cell $clock_late_rise_delay_start $i ] \[get_clocks [constraints get cell 0  $i]\]"
        puts -nonewline $sdc_file "\nset_clock_latency -source -late -fail [constraints get cell $clock_late_fall_delay_start $i ] \[get_clocks [constraints get cell 0  $i]\]"
        puts -nonewline $sdc_file "\nset_clock_transition   -rise -min [constraints get cell $clock_early_rise_slew_start $i ] \[get_clocks [constraints get cell 0  $i]\]"
        puts -nonewline $sdc_file "\nset_clock_transition   -fall -min [constraints get cell $clock_early_fall_slew_start $i ] \[get_clocks [constraints get cell 0  $i]\]"
        puts -nonewline $sdc_file "\nset_clock_transition   -rise -max [constraints get cell $clock_late_rise_slew_start $i ] \[get_clocks [constraints get cell 0  $i]\]"
        puts -nonewline $sdc_file "\nset_clock_transition   -fall -max [constraints get cell $clock_late_fall_slew_start $i ] \[get_clocks [constraints get cell 0  $i]\]"
        set i [expr {$i+1}]
}



set input_early_rise_delay_start [lindex [lindex [constraints search rect $clock_start_column $input_ports_start [expr {$constr_columns-1}] [expr {$output_ports_start-1}] early_rise_delay] 0] 0]
set input_early_fall_delay_start [lindex [lindex [constraints search rect $clock_start_column $input_ports_start [expr {$constr_columns-1}] [expr {$output_ports_start-1}] early_fall_delay] 0] 0]
set input_late_rise_delay_start [lindex [lindex [constraints search rect $clock_start_column $input_ports_start [expr {$constr_columns-1}] [expr {$output_ports_start-1}] late_rise_delay] 0] 0]
set input_late_fall_delay_start [lindex [lindex [constraints search rect $clock_start_column $input_ports_start [expr {$constr_columns-1}] [expr {$output_ports_start-1}] late_fall_delay] 0] 0]
set input_early_rise_slew_start [lindex [lindex [ constraints search rect $clock_start_column $input_ports_start [expr {$constr_columns -1}] [expr {$output_ports_start -1}] early_rise_slew] 0 ] 0 ]
set input_early_fall_slew_start [lindex [lindex [ constraints search rect $clock_start_column $input_ports_start [expr {$constr_columns -1}] [expr {$output_ports_start -1}] early_fall_slew] 0 ] 0 ]
set input_late_rise_slew_start [lindex [lindex [ constraints search rect $clock_start_column $input_ports_start [expr {$constr_columns -1}] [expr {$output_ports_start -1}] late_rise_slew] 0 ] 0 ]
set input_late_fall_slew_start [lindex [lindex [ constraints search rect $clock_start_column $input_ports_start [expr {$constr_columns -1}] [expr {$output_ports_start -1}] late_fall_slew] 0 ] 0 ]

puts "input_early_rise_delay = $input_early_rise_delay_start"
puts "input_early_fall_delay = $input_early_fall_delay_start"
puts "input_late_rise_delay = $input_late_rise_delay_start"
puts "input_late_fall_delay = $input_late_fall_delay_start"
puts "input_early_rise_slew_start = $input_early_rise_slew_start"
puts "input_early_fall_slew_start = $input_early_fall_slew_start"
puts "input_late_rise_slew_start = $input_late_rise_slew_start"
puts "input_late_fall_slew_start = $input_late_fall_slew_start"

```
<img width="1361" height="370" alt="image" src="https://github.com/user-attachments/assets/fcdea828-4591-4175-af5c-c32dcee0aae4" />

--> The commands dumped in sdc file are shown -
/home/vsduser/vsdsynth/outdir_openMSP430/openMSP430.sdc
<img width="862" height="402" alt="image" src="https://github.com/user-attachments/assets/113a5089-6bb2-46d6-b94e-f6b18ae5f1e7" />

--> For input and ouput port constraints - one additional step would be to categorize bits and bussed ports, and provide a * wildcard for the bussed ports to be consumable for the tool

<img width="751" height="257" alt="image" src="https://github.com/user-attachments/assets/632cc774-edad-4c2f-9e1e-8cb5f1044470" />


```
set related_clock [lindex [lindex [constraints search rect $clock_start_column $input_ports_start [expr {$constr_columns-1}] [expr {$output_ports_start-1}]  clocks] 0 ] 0]



set i [expr {$input_ports_start+1}]
set end_of_ports [expr {$output_ports_start-1}]
puts "\nInfo-SDC: Working on IO constraints....."
puts "\nInfo-SDC: Categorizing input ports as bits and bussed"

while { $i < $end_of_ports } {

set netlist [glob -dir $NetlistDirectory *.v]
set tmp_file [open /tmp/1 w]

foreach f $netlist {
        set fd [open $f]
                #puts "reading file $f"
        while {[gets $fd line] != -1} {

                set pattern1 " [constraints get cell 0 $i];"
            if {[regexp -all -- $pattern1 $line]} {
                        #puts "\npattern1 \"$pattern1\" found and matching line in verilog file \"$f\" is \"$line\""
                                set pattern2 [lindex [split $line ";"] 0]
                        #puts "\ncreating pattern2 by splitting pattern1 using semi-colon as delimiter => \"$pattern2\""
                                if {[regexp -all {input} [lindex [split $pattern2 "\S+"] 0]]} {
                        #puts "\nout of all patterns, \"$pattern2\" has matching string \"input\". So preserving this line and ignoring others"
                                set s1 "[lindex [split $pattern2 "\S+"] 0] [lindex [split $pattern2 "\S+"] 1] [lindex [split $pattern2 "\S+"] 2]"
                                #puts "\nprinting first 3 elements of pattern as \"$s1\" using space as delimiter"
                                puts -nonewline $tmp_file "\n[regsub -all {\s+} $s1 " "]"
                                #puts "\nreplace multiple spaces in s1 by space and reformat as \"[regsub -all {\s+} $s1 " "]\""
                                }

                }
        }
close $fd
}
close $tmp_file


set tmp_file [open /tmp/1 r]
set tmp2_file [open /tmp/2 w]
#puts "reading [read $tmp_file]"
#puts "splitting /tmp/1 file as  [split [read $tmp_file] \n]] ]"
#puts "sorting /tmp/1 file as [lsort -unique [split [read $tmp_file] \n]]"
puts -nonewline $tmp2_file "[join [lsort -unique [split [read $tmp_file] \n]] \n]"
close $tmp_file
close $tmp2_file
set tmp2_file [open /tmp/2 r]

set count [llength [read $tmp2_file]]
#puts "Count is $count"
if {$count > 2} {
    set inp_ports [concat [constraints get cell 0 $i]*]
        #puts "\n Bussed"
} else {

    set inp_ports [constraints get cell 0 $i]
        #puts "\n Not Bussed"
}



puts -nonewline $sdc_file "\nset_input_delay -clock  \[get_clocks [constraints get cell $related_clock $i]\] -min -rise -source_latency_included [constraints get cell $input_early_rise_delay_start $i] \[get_ports $inp_ports\]"
        puts -nonewline $sdc_file "\nset_input_delay -clock \[get_clocks [constraints get cell $related_clock $i]\] -min -fall -source_latency_included  [constraints get cell $input_early_fall_delay_start $i] \[get_ports $inp_ports\]"
        puts -nonewline $sdc_file "\nset_input_delay -clock  \[get_clocks [constraints get cell $related_clock $i]\] -max -rise -source_latency_included  [constraints get cell $input_late_rise_delay_start $i] \[get_ports $inp_ports\]"
        puts -nonewline $sdc_file "\nset_input_delay -clock  \[get_clocks [constraints get cell $related_clock $i]\] -max -fall -source_latency_included  [constraints get cell $input_late_fall_delay_start $i] \[get_ports $inp_ports\]"

        puts -nonewline $sdc_file "\nset_input_transition -clock  \[get_clocks [constraints get cell $related_clock $i]\] -min -rise -source_latency_included [constraints get cell $input_early_rise_slew_start $i] \[get_ports $inp_ports\]"
        puts -nonewline $sdc_file "\nset_input_transition -clock \[get_clocks [constraints get cell $related_clock $i]\] -min -fall -source_latency_included  [constraints get cell $input_early_fall_slew_start $i] \[get_ports $inp_ports\]"
        puts -nonewline $sdc_file "\nset_input_transition -clock  \[get_clocks [constraints get cell $related_clock $i]\] -max -rise -source_latency_included  [constraints get cell $input_late_rise_slew_start $i] \[get_ports $inp_ports\]"
        puts -nonewline $sdc_file "\nset_input_transition -clock  \[get_clocks [constraints get cell $related_clock $i]\] -max -fall -source_latency_included  [constraints get cell $input_late_fall_slew_start $i] \[get_ports $inp_ports\]"

set i [expr {$i+1}]
}
close $tmp2_file

```

--> Commands dumped in sdc file are as follows -

<img width="1223" height="482" alt="image" src="https://github.com/user-attachments/assets/808410c9-f43c-4a22-b866-2e3e203953de" />


# Day 4 Module

## Description & Command line Demos

Started with introduction to yosys tool, example with a simple memory file with output netlist diagrams were explained.

Hierarchy check for the verilog files. Making sure all the modules and submodules are valid and instantiated properly in the RTL files.

--> Commands for the tool to do the hier check for files in verilog directory is done by dumping in a hier.ys file in output directory. We also have error handling mechanism as we are executing a shell command in the tcl script. The error_flag is obtained by using catch{exec{{}}.

Code block -

```
#Hierarchy Check Script


puts "\n Info: Creating hierarchy check script to be used by Yosys"
set data "read_liberty -lib -ignore_miss_dir -setattr blackbox ${LateLibraryPath}"
#puts "data is \"$data\""
set filename "$DesignName.hier.ys"
#puts "\nfilename is \"$filename\""
set fileId [open $OutputDirectory/$filename "w"]
#puts "open \"$OutputDirectory/$filename"\ in write mode"
puts -nonewline $fileId $data
#puts "netlist is \"$netlist\""
set netlist [glob -dir $NetlistDirectory *.v]
foreach f $netlist {
        set data $f
        #puts "data is \"$f\""
        puts -nonewline $fileId "\n read_verilog $f"
}
puts -nonewline $fileId "\nhierarchy -check"
close $fileId



set my_error [catch { exec yosys -s $OutputDirectory/$DesignName.hier.ys >& $OutputDirectory/$DesignName.hierarchy_check.log} msg]
puts "Error flag is \"$my_error\""
if { $my_error } {
        set filename "$OutputDirectory/$DesignName.hierarchy_check.log"
        puts "log file name is \"$filename\" "
        set pattern "referenced in module"
        #puts "pattern is $pattern"
        set count 0
        set fid [open $filename r]
        while {[gets $fid line] != -1} {
                # -- used to say end of command options. everything after this is args
                incr count [regexp -all -- $pattern $line]
                if {[regexp -all -- $pattern $line]} {
                        puts "\nError: module [lindex $line 2] is not part of design $DesignName. Please correct RTL in the path '$NetlistDirectory'"
                        puts "\nInfo: Hierarchy check FAIL"
                }
        }
        close $fid
} else {
        puts "\nInfo: Hierarchy check PASS"
}


```

--> Execution in command line 

<img width="1315" height="441" alt="image" src="https://github.com/user-attachments/assets/6db42b03-a056-4a63-8227-bcee8ec4369a" />

--> We could also insert an undefined module to check the working of catch command.

--> hier.ys file looks like this

<img width="870" height="587" alt="image" src="https://github.com/user-attachments/assets/b6dc06a8-50bf-4ac9-a2a4-9c9a435d496e" />



# Day 5 Module

## Description & Command line Demos

Completing the synthesis required scripts, deep dive into different procs, and converting formatt[1] and sdcs to format[2] which is consumable for OpenTimer tool to generate the timing & QoR finally.

-->Below script is a design_name.ys file that is passed to yosys writing all the necessary constraints and netlist required to start the process. It also has a similar error handling case with the error/snth details available in a separate log, similar to hierarchy checks.

```
puts "\nInfo: Creating main synthesis script to be used for yosys"
set data "read_liberty -lib -ignore_miss_dir -setattr blackbox ${LateLibraryPath}"
set filename "$DesignName.ys"
#puts "\nfilename is \"$filename\""
set fileId [open $OutputDirectory/$filename "w"]
#puts "open \"$OutputDirectory/$filename\" in write mode"
puts -nonewline $fileId $data
#puts "netlist is \"$netlist\""
set netlist [glob -dir $NetlistDirectory *.v]
foreach f $netlist {
	set data $f
	#puts "data is \"$f\""
	puts -nonewline $fileId "\n read_verilog $f"
}

puts -nonewline $fileId "\nhierarchy -top $DesignName"
puts -nonewline $fileId "\nsynth -top $DesignName"
puts -nonewline $fileId "\nsplitnets -ports -format __\ndfflibmap -liberty ${LateLibraryPath}\nopt"
puts -nonewline $fileId "\nabc -liberty ${LateLibraryPath}"
puts -nonewline $fileId "\nflatten"
puts -nonewline $fileId "\nclean -purge\niopadmap -outpad BUFX2 A:Y -bits\nopt \nclean"
puts -nonewline $fileId "\nwrite_verilog $OutputDirectory/$DesignName.synth.v"
close $fileId
puts "\nInfo: Synthesis script created and can be accesed from the path $OutputDirectory/$filename"
puts "\nInfo: Running Synthesis...."

if {[catch { exec yosys -s $OutputDirectory/$DesignName.ys >& $OutputDirectory/$DesignName.synthesis.log} msg]} {
	puts "\nError: Syntesis failed due to errors. Please refer to log $OutputDirectory/$DesignName.synthesis.log for errors"
	exit
} else {
	puts "\nInfo: Synthesis finished sucessfully"
}

puts "\nInfo: Please refert olog $OutputDirectory/$DesignName.synthesis.log"
```

-->  Command line execution for succesful synthesis is shown -

<img width="1127" height="198" alt="image" src="https://github.com/user-attachments/assets/6ded84d4-c01b-496e-898d-730b7b1241bb" />

--> The output files are provided in main

--> Further we remove * for the bussed ports in synth netlist to be passed to OpenTimer. We get a synth.v o/p netlist.

```
set fileId [open /tmp/1 "w"]
puts -nonewline $fileId [exec grep -v -w "*" $OutputDirectory/$DesignName.synth.v]
close $fileId

set output [open $OutputDirectory/$DesignName.final.synth.v "w"]

set filename "/tmp/1"
set fid [open $filename r]
	while {[gets $fid line] != -1} {
	puts -nonewline $output [string map {"\\" ""} $line]
	puts -nonewline $output "\n"
}

close $fid
close $output

puts "\nInfo: Please find the synthesized netlist for $DesignName at below path. You can use this netlist for STA or PNR"
puts "\n$OutputDirectory/$DesignName.final.synth.v"

```

"Info: Please find the synthesized netlist for openMSP430 at below path. You can use this netlist for STA or PNR

/home/vsduser/vsdsynth/outdir_openMSP430/openMSP430.final.synth.v"

<img width="1280" height="882" alt="image" src="https://github.com/user-attachments/assets/216259f7-21ae-4b18-adbb-d98f00c8ec5f" />


--> PROCS - procedures in TCL - equivalent to functions in other prog languages
We are dealing with 5 procs to help with proceeding to timing analysis for OpenTimer

```
puts "\nInfo: Timing Analysis Started ... "
puts "\nInfo: Initializing number of threads, libraries, sdc, verilog netlist path..."

source /home/vsduser/vsdsynth/procs/reopenStdout.proc
source /home/vsduser/vsdsynth/procs/set_num_threads.proc
reopenStdout $OutputDirectory/$DesignName.conf
set_multi_cpu_usage -localCpu 4

source /home/vsduser/vsdsynth/procs/read_lib.proc
read_lib -early /home/vsduser/vsdsynth/osu018_stdcells.lib
read_lib -late /home/vsduser/vsdsynth/osu018_stdcells.lib

source /home/vsduser/vsdsynth/procs/read_verilog.proc
read_verilog $OutputDirectory/$DesignName.final.synth.v

source /home/vsduser/vsdsynth/procs/read_sdc.proc
read_sdc $OutputDirectory/$DesignName.sdc
reopenStdout /dev/tty
```
1. reopenStdout: Redirects the standard output (stdout) of the OpenTimer session to a specified log file.
2. set_multi_cpu_usage: Configures OpenTimer to use <num_threads> CPU cores for parallel timing analysis to speed up processing.
3. read_verilog: Parses and loads the Verilog netlist file to build the design’s gate-level structural representation.
4. read_lib: Reads a Liberty timing library file that defines cell timing, power, and functionality for analysis.
5. read_sdc: Loads Synopsys Design Constraints (SDC) to apply clock definitions, timing exceptions, and design constraints.
   --> in our case, read_sdc will remove the [] brackets in .sdc file to make it intelligible for the OpenTimer tool to consume.
```
proc read_sdc {arg1} {
set sdc_dirname [file dirname $arg1]
set sdc_filename [lindex [split [file tail $arg1] .] 0 ]
set sdc [open $arg1 r]
set tmp_file [open /tmp/1 "w"] 
puts -nonewline $tmp_file [string map {"\[" "" "\]" " "} [read $sdc]]     
close $tmp_file
}
```   
   --> Further processing is done on the create clock constraints to convert to format[2]
     
```
set tmp_file [open /tmp/1 r]
set timing_file [open /tmp/3 w]
set lines [split [read $tmp_file] "\n"]
set find_clocks [lsearch -all -inline $lines "create_clock*"]
foreach elem $find_clocks {
	set clock_port_name [lindex $elem [expr {[lsearch $elem "get_ports"]+1}]]
	puts "clock_port_name is \"$clock_port_name\" "
	set clock_period [lindex $elem [expr {[lsearch $elem "-period"]+1}]]
	puts "clock_period is \"$clock_period\" "
	set duty_cycle [expr {100 - [expr {[lindex [lindex $elem [expr {[lsearch $elem "-waveform"]+1}]] 1]*100/$clock_period}]}]
	puts "duty_cycle is \"$duty_cycle\" "
	puts $timing_file "clock $clock_port_name $clock_period $duty_cycle"
	}
close $tmp_file
```

--> Further processing is done on the set_clock_latency, set_clock_transition, set_input_delay, set_input_transition,  set_output_delay constraints. Example snippet is as follows for set_clock_latency

```
set find_keyword [lsearch -all -inline $lines "set_clock_latency*"]
set tmp2_file [open /tmp/2 w]
set new_port_name ""
foreach elem $find_keyword {
        set port_name [lindex $elem [expr {[lsearch $elem "get_clocks"]+1}]]
	if {![string match $new_port_name $port_name]} {
        	set new_port_name $port_name
        	set delays_list [lsearch -all -inline $find_keyword [join [list "*" " " $port_name " " "*"] ""]]
        	set delay_value ""
        	foreach new_elem $delays_list {
        		set port_index [lsearch $new_elem "get_clocks"]
        		lappend delay_value [lindex $new_elem [expr {$port_index-1}]]
        	}
		puts -nonewline $tmp2_file "\nat $port_name $delay_value"
	}
}

close $tmp2_file
```

--> Below snippet is used to process bussed ports to format[2] for the open timer tool (* to _number_)

```
set ot_timing_file [open $sdc_dirname/$sdc_filename.timing w]
set timing_file [open /tmp/3 r]
while {[gets $timing_file line] != -1} {
        if {[regexp -all -- {\*} $line]} {
                set bussed [lindex [lindex [split $line "*"] 0] 1]
                set final_synth_netlist [open $sdc_dirname/$sdc_filename.final.synth.v r]
                while {[gets $final_synth_netlist line2] != -1 } {
                        if {[regexp -all -- $bussed $line2] && [regexp -all -- {input} $line2] && ![string match "" $line]} {
                        puts -nonewline $ot_timing_file "\n[lindex [lindex [split $line "*"] 0 ] 0 ] [lindex [lindex [split $line2 ";"] 0 ] 1 ] [lindex [split $line "*"] 1 ]"
                        } elseif {[regexp -all -- $bussed $line2] && [regexp -all -- {output} $line2] && ![string match "" $line]} {
                        puts -nonewline $ot_timing_file "\n[lindex [lindex [split $line "*"] 0 ] 0 ] [lindex [lindex [split $line2 ";"] 0 ] 1 ] [lindex [split $line "*"] 1 ]"
                        }
                }
        } else {
        puts -nonewline $ot_timing_file  "\n$line"
        }
}

close $timing_file
puts "set_timing_fpath $sdc_dirname/$sdc_filename.timing"
}

```

--> Below code snippet is used to create the spef file for parsitics information required for timing calcs in the tool

```
if {$enable_prelayout_timing == 1} {
	puts "\nInfo: enable prelayout_timing is $enable_prelayout_timing. Enabling zero-wire load parasitics"
	set spef_file [open $OutputDirectory/$DesignName.spef w]
	puts $spef_file "*SPEF \"IEEE 1481-1998\""
	puts $spef_file "*DESIGN \"$DesignName\""
	puts $spef_file "*DATE \"Sun Jun 11 11:59:00 2023\""
	puts $spef_file "*VENDOR \"VLSI System Design\""
	puts $spef_file "*PROGRAM \"TCL Workshop\""
	puts $spef_file "*DATE \"0.0\""
	puts $spef_file "*DESIGN FLOW \"NETLIST_TYPE_VERILOG\""
	puts $spef_file "*DIVIDER /"
	puts $spef_file "*DELIMITER : "
	puts $spef_file "*BUS_DELIMITER [ ]"
	puts $spef_file "*T_UNIT 1 PS"
	puts $spef_file "*C_UNIT 1 FF"
	puts $spef_file "*R_UNIT 1 KOHM"
	puts $spef_file "*L_UNIT 1 UH"
}
close $spef_file
```

--> .conf file is created which would compile all the commands to be passed as input to OpenTimer -
```
set conf_file [open $OutputDirectory/$DesignName.conf a]
puts $conf_file "set_spef_fpath $OutputDirectory/$DesignName.spef"
puts $conf_file "init_timer"
puts $conf_file "report_timer"
puts $conf_file "report_wns"
puts $conf_file "report_tns"
puts $conf_file "report_worst_paths -numPaths 10000 " 
close $conf_file
```

--> Few demo screenshots for the above snippets

<img width="1213" height="201" alt="image" src="https://github.com/user-attachments/assets/29f38a9f-52e6-448a-abe7-52931e029afe" />

<img width="706" height="518" alt="image" src="https://github.com/user-attachments/assets/68225afe-627d-4bc9-8173-a218721e7fb1" />

<img width="1207" height="552" alt="image" src="https://github.com/user-attachments/assets/80e11fd3-00c3-4cea-8e0f-8a112b960c21" />

<img width="1337" height="955" alt="image" src="https://github.com/user-attachments/assets/0d15f225-d671-4ad5-821a-6ad436aeb2d5" />

--> Quality of Results generation

--> First we find the runtime for STA analysis by the tool. Results are available in the .results file

```
set time_elapsed_in_us [time {exec /home/vsduser/OpenTimer-1.0.5/bin/OpenTimer < $OutputDirectory/$DesignName.conf >& $OutputDirectory/$DesignName.results} ]
set time_elapsed_in_sec "[expr {[lindex $time_elapsed_in_us 0]/100000}]sec"
puts "\nInfo: STA finished in $time_elapsed_in_sec seconds"
puts "\nInfo: Refer to $OutputDirectory/$DesignName.results for warings and errors"
```

<img width="1132" height="401" alt="image" src="https://github.com/user-attachments/assets/1b5d725d-fba2-4bbf-8878-392ef16e44a5" />


--> Script to extract worst output vios and setup/hold vios and their counts, gate instances -
```
#-------------------- Finding worst outptut violation --------------------------#
set worst_RAT_slack "-"
set report_file [open $OutputDirectory/$DesignName.results r]
set pattern {RAT}
while {[gets $report_file line] != -1} {
	if {[regexp $pattern $line]} {
		set worst_RAT_slack "[expr {[lindex $line 3]/1000}]ns"
		break
	} else {
		continue
	}
}
close $report_file
#--------------------------- Finding number of outptut violation ------------------------#
set report_file [open $OutputDirectory/$DesignName.results r]
set count 0
while {[gets $report_file line] != -1} {
	incr count [regexp -all -- $pattern $line]
}
set Number_output_violations $count
close $report_file

#--------------------------- Finding worst setup violation -------------------------#
set worst_negative_setup_slack "-"
set report_file [open $OutputDirectory/$DesignName.results r] 
set pattern {Setup}
while {[gets $report_file line] != -1} {
	if {[regexp $pattern $line]} {
		set worst_negative_setup_slack "[expr {[lindex $line 3]/1000}]ns"
		break
	} else {
		continue
	}
}
close $report_file


#-------------------------find number of setup violations--------------------------------#
set report_file [open $OutputDirectory/$DesignName.results r]
set count 0
while {[gets $report_file line] != -1} {
	incr count [regexp -all -- $pattern $line]
}
set Number_of_setup_violations $count
close $report_file

#-------------------------find worst hold violation--------------------------------#
set worst_negative_hold_slack "-"
set report_file [open $OutputDirectory/$DesignName.results r] 
set pattern {Hold}
while {[gets $report_file line] != -1} {
	if {[regexp $pattern $line]} {
		set worst_negative_hold_slack "[expr {[lindex $line 3]/1000}]ns"
		break
	} else {
		continue
	}
}
close $report_file

#-------------------------find number of hold violations--------------------------------#
set report_file [open $OutputDirectory/$DesignName.results r]
set count 0
while {[gets $report_file line] != -1} {
	incr count [regexp -all -- $pattern $line]
}
set Number_of_hold_violations $count
close $report_file

#-------------------------find number of instances--------------------------------#

set pattern {Num of gates}
set report_file [open $OutputDirectory/$DesignName.results r] 
while {[gets $report_file line] != -1} {
	if {[regexp $pattern $line]} {
		set Instance_count "[lindex [join $line " "] 4 ]"
		break
	} else {
		continue
	}
}
close $report_file


puts "DesignName is \{$DesignName\}"
puts "time_elapsed_in_sec is \{$time_elapsed_in_sec\}"
puts "Instance_count is \{$Instance_count\}"
puts "worst_negative_setup_slack is \{$worst_negative_setup_slack\}"
puts "Number_of_setup_violations is \{$Number_of_setup_violations\}"
puts "worst_negative_hold_slack is \{$worst_negative_hold_slack\}"
puts "Number_of_hold_violations is \{$Number_of_hold_violations\}"
puts "worst_RAT_slack is \{$worst_RAT_slack\}"
puts "Number_output_violations is \{$Number_output_violations\}"
```

--> Finally we format this to put every important parameter in a neat table like format -

```
puts "\n"
puts "						****PRELAYOUT TIMING RESULTS**** 					"
set formatStr "%15s %15s %15s %15s %15s %15s %15s %15s %15s"

puts [format $formatStr "----------" "-------" "--------------" "---------" "---------" "--------" "--------" "-------" "-------"]
puts [format $formatStr "DesignName" "Runtime" "Instance Count" "WNS Setup" "FEP Setup" "WNS Hold" "FEP Hold" "WNS RAT" "FEP RAT"]
puts [format $formatStr "----------" "-------" "--------------" "---------" "---------" "--------" "--------" "-------" "-------"]
foreach design_name $DesignName runtime $time_elapsed_in_sec instance_count $Instance_count wns_setup $worst_negative_setup_slack fep_setup $Number_of_setup_violations wns_hold $worst_negative_hold_slack fep_hold $Number_of_hold_violations wns_rat $worst_RAT_slack fep_rat $Number_output_violations {
	puts [format $formatStr $design_name $runtime $instance_count $wns_setup $fep_setup $wns_hold $fep_hold $wns_rat $fep_rat]
}

puts [format $formatStr "----------" "-------" "--------------" "---------" "---------" "--------" "--------" "-------" "-------"]
puts "\n"
```


<img width="1542" height="441" alt="image" src="https://github.com/user-attachments/assets/ecb7e91f-74f2-4344-a413-c5239d4e4cc9" />














