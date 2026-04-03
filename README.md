This code is implementing a basic TFTP client through an RP2350 board
attached to a Wiznet W5500 SPI ethernet adapter

It requires a DHCP server which will deliver an IP address through a DHCP request
Lease time is not taken into account for the moment by the client. It shall be set
in a way the client has enough time to retrieve the TFTP transfer

A bootfile name shall be provided by the DHCP server

The client is then issuing TFTP request to the default Gateway (will be changed
later to a DHCP Option 66/67) 

The transfer happens and the code reports the beginning and end of the transfer

Sample TFTP server can be downloaded here: https://github.com/vejmarie/golang-tftp-example
Sample DHCP server can be downloaded here: https://github.com/coredhcp/coredhcp.git
And apply the Patches present into that tree

# To build 

Please use the tasks scheduler for rp2350. The default core scheduler is experiencing a race
condition which is creating a deadlock when the garbage collector is activated. This can
be easily triggered by running multiple transfer through that example. The heap is filled up
to the time memory is exhausted and the Garbage Collector is triggered. Unfortunately the
multicore implementation has a mutex issue, which is leading the use of a more multicore
friendly scheduler

For maximum performances please use the -opt=2 option
The code has also been updated for TFTP transfer block size of 1400 bytes (by default
TFTP is supporting 512 bytes). Please be sure that your TFTP server can adjust to this
requirements

../../../../build/tinygo flash -monitor -target pico2 -scheduler tasks -opt=2 ./main.go

# Some side effect

The code has been extensively written using chatHPE. This helped to write it faster, but
harder to debug. In the end I estimate to have been two times fater than by writing it 
myself

You can reach peak performances of a w5500 which is roughly 12mbps through a TFTP transfer.

