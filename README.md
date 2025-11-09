1. program
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
1$cbr0 set interval_ 0.005
$cbr0 attach-agent $udp0
set sink [new Agent/Null]
$ns attach-agent $n2 $sink
$ns connect $udp0 $sink
$ns at 0.2 "$cbr0 start"
$ns at 4.5 "$cbr0 stop"
$ns at 5.0 "finish"
$ns run
ns lab1.tcl
gedit program1.awk
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
The no. of packets received: 200
