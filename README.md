# . Implement three nodes point –to-point network with duplex links between
them. Set the queue size, vary the bandwidth, and find the number of packets 
dropped.
Commands
gedit lab1.tcl
set ns [new Simulator]
set nf [open lab1.nam w]
$ns namtrace-all $nf
set nd [open lab1.tr w]
$ns trace-all $nd
proc finish {} {
global ns nf nd
$ns flush-trace
close $nf
close $nd
exec nam lab1.nam &
exit 0
}
set n0 [$ns node]
set n1 [$ns node]
set n2 [$ns node]
$ns duplex-link $n0 $n1 1Mb 10ms DropTail
$ns duplex-link $n1 $n2 512kb 10ms DropTail
$ns queue-limit $n1 $n2 10
set udp0 [new Agent/UDP]
$ns attach-agent $n0 $udp0
set cbr0 [new Application/Traffic/CBR]
$cbr0 set packetSize_ 500
2
$cbr0 set interval_ 0.005
$cbr0 attach-agent $udp0
set sink [new Agent/Null]
$ns attach-agent $n2 $sink
$ns connect $udp0 $sink
$ns at 0.2 "$cbr0 start"
$ns at 4.5 "$cbr0 stop"
$ns at 5.0 "finish"
$ns run
ns lab1.tcl
BEGIN{
dcount=0;
rcount=0;
}{
event=$1;
if(event=="d")
{
dcount++;
}
if(event=="r")
{
rcount++;
}
}
END{
printf("The no. of packets dropped: %d\n",dcount);
printf("The no. of packets reveived: %d\n",rcount);
}
awk -f program1.awk lab1.tr
OUTPUT:
The no. of packets dropped: 10
The no. of packets received: 200.       
2 . Implement transmission of ping messages/trace route over a network 
topology consisting of 6 nodes and find the number of packets dropped due to 
congestion.
gedit prg2.tcl
#Create Simulator 
set ns [new Simulator] 
#Use colors to differentiate the traffic 
$ns color 1 Blue 
$ns color 2 Red 
#Open trace and NAM trace file 
set ntrace [open prg2.tr w] 
$ns trace-all $ntrace 
set namfile [open prg2.nam w] 
$ns namtrace-all $namfile 
#Finish Procedure 
proc Finish {} { 
global ns ntrace namfile 
#Dump all trace data and close the file 
$ns flush-trace 
close $ntrace 
close $namfile 
#Execute the nam animation file 
exec nam prg2.nam & 
5
#Find the number of ping packets dropped 
puts "The number of ping packets dropped are " 
exec grep "^d" prg2.tr | cut .d " " .f s | grep .c "ping" & 
exit 0 
} 
#Create six nodes 
for {set i 0} {$i < 6} {incr i} { 
set n($i) [$ns node] 
} 
#Connect the nodes 
for {set j 0} {$j < 5} {incr j} { 
$ns duplex-link $n($j) $n([expr ($j+1)]) 0.1Mb 10ms DropTail         Define the recv function for the class 'Agent/Ping' 
Agent/Ping instproc recv {from rtt} { 
$self instvar node_ 
puts "node [$node_ id] received ping answer from $from with round trip time $rtt 
ns" 
} 
#Create two ping agents and attach them to n(0) and n(5) 
set p0 [new Agent/Ping] 
$p0 set class_ 1 
$ns attach-agent $n(0) $p0 
6
set p1 [new Agent/Ping] 
set class_ 1 
$ns attach-agent $n(5) $p1 
$ns connect $p0 $p1 
#Set queue size and monitor the queue 
#Queue size is set to 2 to observe the drop in ping packets 
$ns queue-limit $n(2) $n(3) 2 
$ns duplex-link-op $n(2) $n(3) queuePos 0.5 
#Create Congestion 
#Generate a Huge CBR traffic between n(2) and n(4) 
set tcp0 [new Agent/TCP] 
$tcp0 set class_ 2 
$ns attach-agent $n(2) $tcp0 
set sink0 [new Agent/TCPSink] 
$ns attach-agent $n(4) $sink0 
$ns connect $tcp0 $sink0 
#Apply CBR traffic over TCP 
set cbr0 [new Application/Traffic/CBR] 
$cbr0 set packetSize_ 500 
$cbr0 set rate_ 1Mb 
$cbr0 attach-agent $tcp0             #Schedule events 
$ns at 0.2 "$p0 send" 
7
$ns at 0.4 "$p1 send" 
$ns at 0.4 "$cbr0 start" 
$ns at 0.8 "$p0 send" 
$ns at 1.0 "$p1 send" 
$ns at 1.2 "$cbr0 stop" 
$ns at 1.4 "$p0 send" 
$ns at 1.6 "$p1 send" 
$ns at 1.8 "Finish" 
$ns run
ns prg2.tcl
OUTPUT
  3. Implement an Ethernet LAN using n nodes and set multiple traffic nodes 
and plot congestion window for different source / destination.
gedit prog3.tcl
# Create Simulator
set ns [new Simulator]
# Use colors to differentiate the traffic
$ns color 1 Blue
$ns color 2 Red
# Open trace and NAM trace file
set ntrace [open prog5.tr w]
$ns trace-all $ntrace
set namfile [open prog5.nam w]
$ns namtrace-all $namfile
# Create flat files to create congestion graph windows
set winFile0 [open WinFile0 w]
set winFile1 [open WinFile1 w]
# Finish Procedure
proc Finish {} {
 # Dump all trace data and close the files
 global ns ntrace namfile
 $ns flush-trace
 close $ntrace
 close $namfile
 # Execute the NAM animation file
 exec nam prog5.nam &
 # Plot the Congestion Window graph using xgraph
 exec xgraph WinFile0 WinFile1 &
 exit 0
}
# Plot Window Procedure
proc PlotWindow {tcpSource file} {
 global ns
10
 set time 0.1
 set now [$ns now]
 set cwnd [$tcpSource set cwnd_]
 puts $file "$now $cwnd"
 $ns at [expr $now+$time] "PlotWindow $tcpSource $file"
}
# Create 6 nodes
for {set i 0} {$i < 6} {incr i} {
 set n($i) [$ns node]
}
# Create duplex links between the nodes
$ns duplex-link $n(0) $n(2) 2Mb 10ms DropTail
$ns duplex-link $n(1) $n(2) 2Mb 10ms DropTail
$ns duplex-link $n(2) $n(3) 0.6Mb 100ms DropTail
# Nodes n(3), n(4), and n(5) are considered in a LAN
set lan [$ns newLan "$n(3) $n(4) $n(5)" 0.5Mb 40ms LL Queue/DropTail MAC/802_3 Channel]
# Orientation to the nodes
$ns duplex-link-op $n(0) $n(2) orient right-down
$ns duplex-link-op $n(1) $n(2) orient right-up
$ns duplex-link-op $n(2) $n(3) orient right
# Setup queue between n(2) and n(3) and monitor the queue
$ns queue-limit $n(2) $n(3) 20
$ns duplex-link-op $n(2) $n(3) queuePos 0.5
# Set error model on link n(2) to n(3)
set loss_module [new ErrorModel]
$loss_module ranvar [new RandomVariable/Uniform]
$loss_module drop-target [new Agent/Null]
$ns lossmodel $loss_module $n(2) $n(3)
# Set up the TCP connection between n(0) and n(4)
set tcp0 [new Agent/TCP/Newreno]
$tcp0 set fid_ 1
11
$tcp0 set window_ 8000
$tcp0 set packetSize_ 552
$ns attach-agent $n(0) $tcp0
set sink0 [new Agent/TCPSink/DelAck]
$ns attach-agent $n(4) $sink0
$ns connect $tcp0 $sink0
# Apply FTP Application over TCP
set ftp0 [new Application/FTP]
$ftp0 attach-agent $tcp0
$ftp0 set type_ FTP
# Set up another TCP connection between n(5) and n(1)
set tcp1 [new Agent/TCP/Newreno]
$tcp1 set fid_ 2
$tcp1 set window_ 8000
$tcp1 set packetSize_ 552
$ns attach-agent $n(5) $tcp1
set sink1 [new Agent/TCPSink/DelAck]
$ns attach-agent $n(1) $sink1
$ns connect $tcp1 $sink1
# Apply FTP application over TCP     set ftp1 [new Application/FTP]
$ftp1 attach-agent $tcp1
$ftp1 set type_ FTP
# Schedule Events
$ns at 0.1 "$ftp0 start"
$ns at 0.1 "PlotWindow $tcp0 $winFile0"
$ns at 0.5 "$ftp1 start"
$ns at 0.5 "PlotWindow $tcp1 $winFile1"
$ns at 25.0 "$ftp0 stop"
$ns at 25.1 "$ftp1 stop"
$ns at 25.2 "Finish"
# Run the simulation
$ns run
12
ns prog3.tcl
OUTPUT
 4. Develop a program for error detecting code using CRC-CCITT (16- bits).
nano CRC4.java
import java.util.Scanner;
import java.io.*;
public class CRC4 {
 public static void main(String args[]) {
 Scanner sc = new Scanner(System.in);
 //Input Data Stream
 System.out.print("Enter message bits: ");
 String message = sc.nextLine();
 System.out.print("Enter generator: ");
 String generator = sc.nextLine();
int data[] = new int[message.length() + generator.length() - 1];
int divisor[] = new int[generator.length()];
for(int i=0;i<message.length();i++)
data[i] = Integer.parseInt(message.charAt(i)+"");
for(int i=0;i<generator.length();i++)
divisor[i] = Integer.parseInt(generator.charAt(i)+"");
//Calculation of CRC
for(int i=0;i<message.length();i++)
{
if(data[i]==1)
for(int j=0;j<divisor.length;j++)
data[i+j] ^= divisor[j];
}
//Display CRC
System.out.print("The checksum code is: ");
for(int i=0;i<message.length();i++)
data[i] = Integer.parseInt(message.charAt(i)+"");
for(int i=0;i<data.length;i++) 
 System.out.print(data[i]);
System.out.println();
//Check for input CRC code
14
System.out.print("Enter checksum code: ");
message = sc.nextLine();
System.out.print("Enter generator: ");
generator = sc.nextLine();
data = new int[message.length() + generator.length() - 1];
divisor = new int[generator.length()];
for(int i=0;i<message.length();i++)
data[i] = Integer.parseInt(message.charAt(i)+"");
for(int i=0;i<generator.length();i++)
divisor[i] = Integer.parseInt(generator.charAt(i)+"");
//Calculation of remainder
for(int i=0;i<message.length();i++) {
if(data[i]==1)
for(int j=0;j<divisor.length;j++)
data[i+j] ^= divisor[j];
}
//Display validity of data
boolean valid = true;
for(int i=0;i<data.length;i++)
if(data[i]==1){
valid = false;
break;
}
if(valid==true) 
System.out.println("Data stream is valid");
else 
System.out.println("Data stream is invalid. CRC error occurred.");
}
}
Press Ctrl + O (to write out).
Press Enter to confirm the file name.
Press Ctrl + X to exit nano.
 javac CRC4.java
 java CRC4 
15
OUTPUT
