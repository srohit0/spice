# NAME
spice - interface calls for parsing spice netlst.

# SYNOPSIS
use spice ;
spiceInit ( $file ) ;
   #returns 0 if initilization is successful and >0 otherwise.
   #check $spice::error in case of failure.

# DESCRIPTION
## CALLING spice.pm ROUTINES
This package only cares about m, R, C, X, and subckts.
spice-decks are ignored.
 
This priliminary version of spice supports following subroutine
calls-

```
@subckt = getTopSubckts ( ) ;
   returns a list of subckts, top in the hierarchy.
 
@subcktList = getSubcktList ( ) ;
   returns a list of subckts present in the netlist.
 
$subcktDefn = getSubckt ( $subckt ) ;
   returns a string containing the definition of subckt.
 
getResistors ( $subckt ) ;
   returns a hash containing the name and value of Resistors.
 
getCapacitors ( $subckt ) ;
   returns a hash containing the name and value of capacitors.
 
getTransistors ( $subckt ) ;
   returns a hash of transitor names and their types. i.e. n or p.
 
getInstances ( $subckt ) ;
   returns a hash of instantion name and their subckt names.
```

## ERROR HANDLING
All these subroutines return a value less than 0 upon failure.
 
you may want to check following strings for the reason-
```
$spice::error
$spice::warn
 ```
You may want to set $spice::verbose to 1, for a detailed message on
the standard output.

# APPLICATION
This release o Extracting the hierarchy. o Traversing the netlist. o Creating html document for an easy naviagation of netlist. o any other appication, which requires netlist specific information.

# Future Release 
Current version does not support more than one netlist open at a time. Future version, (Object Oriented Design) shall support this feature. It can help developing the following applications- 
- Comparing two spice netlists. 
- extracting the netlist difference.

Here is the code of traversing the netlist-

```
use spice ;
$spice::verbose = 0 ;

my $spiceFile ;
my $subckt ;

if ( $#ARGV < 0 ) {
   while (1) {
      print "Enter spice file:" ;
      $spiceFile = <STDIN>; chop $spiceFile ;
      last if ( length $spiceFile )
      }
   print "Enter subckt name(default-top):" ;
   $subckt = <STDIN>; chop  $subckt ;
   }
else {
   $spiceFile = $ARGV[0] ;
   $subckt = $ARGV[1] ;
   }

my $init ;
$init = spiceInit ( $spiceFile ) ;
   if ( $init == "-1" ) {
     print "$spice::error\n" ;
     exit 0 ;
     }

my @subckts ;
($subckt) ? ( @subckts = ( $subckt) ) : ( @subckts = getTopSubckts() ) ;

foreach $subckt ( @subckts ) {
   traverseHier ( $subckt, 0 ) ;
   }

sub traverseHier ( ) {
   my ( $node, $count ) = @_ ;
   my $i ;
   for ($i=0; $i<$count; $i++ ) { print "\t" ; }
   $count = $count + 1;
   print "$count.$node\n" ;
   my %subckts = getInstances ($node) ;
   my @children = values %subckts ;
   @children = removeDup ( @children ) ;
   undef %subckts ;
   if ( $#children < 0 ) { return ; }
   my $child ;
   foreach $child ( @children ) {
      traverseHier ( $child, $count ) ;
      }
   return ;
   }

sub removeDup {
   my ( @list ) = @_ ;
   my $part ;
   my %hash = ( ) ;
   foreach $part ( @list ) {
      $part =~ s/\s+//g ;
      $hash{$part} = 1 if ( length ( $part ) ) ;
      }
   @list = keys %hash ;
   return @list ;
   }

exit 0 ;
```

# INTERNALS
The netlist is parsed in two simple phases. First phase requires
opening a temporary file, which is deleted soon after creating
the data-structures. This was done to manage unusually large
netlist, to reduce on run time memory requirements. (With earlier
implementation, where I do not write any tmp files, I came across
a testcase, which took huge memory to build the internal data
structures.)
 
You may have problems running the scripts using this package in the
directories, where you do not have write permission. Please set
$spice::tmpFile variable to a file name with absolute path, where
you have write permissions.
 
Top level instances are stored in a hypothetical subcircuit called
B<top>. If you need to change this subcircuit name, please set the
variable $spice::topSubckt to the name you desire.

# LIMITATION
As mentioned earlier, this package only cares about the circuit
elements like Resistors, Capacitors and Transistors along with
subcircuit deinition and instances. Everything else is ignored.
 
user can open only one file at a time. Future version shall
support opening more tha one netlist at a time.


##################################################################
###    Copyright (c) 2000 Rohit Sharma. All rights reserved.
###    This program is free software; you can redistribute it and/or
###    modify it under the same terms as Perl itself.
##################################################################

#############
### Author      : Rohit Sharma
### Date        : 21 August, 2000.
### Description : SPICE netlist interface
#############
