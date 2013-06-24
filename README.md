packetcat
=========

Similar to netcat, allows you to script sending and receiving of packet based communications.

### Script Format ###
The script file contains the configuration and transmit data packetcat will use to communicate with a remote service. The following defines the format of the script file.

    ---
    T,192.168.1.200,4000,1000,250
    
    01 23 45 67 89
    
    A0 B1 C2 D3 F4 00 11 22
    33 44 55 66
    ---
    *
    
    00 01 02 03 04 05 06 07
    AA BB CC DD EE FF 01 23
    45 67
    
    00 01 02 03 04 05 06 07
    08 09 0A 0B 0C
    ---

#### Start ####    
All scripts start with a `---` line which starts a new connection (and ends the old one).

#### Configuration ####
The next line defines the configuration of the communication method. The configuration states the mechanism (TCP, UDP, Serial) and then a comma delineated list of parameters:

**TCP:** T,_IP or Host Name_, _Port_, _Receive Timeout_, _Rx to Tx Delay_

**UDP:** U, _IP or Host Name_, _Port_, _Receive Timeout_, _Rx to Tx Delay_

**Serial:** S, _Port Name_, _Receive Timeout_, _Rx to Tx Delay_

If the configuration should stay the same as previously configured (or as defined by command line arguments), a single asterisk `*` may be used. 

#### Communication ####
After the configuration line all lines are interpreted as:

**Transmit Data:** Hexadecimal bytes space separated. New lines (\n\r, CRLF, etc) are allowed to format the script and do not define the end of transmit data.

**Send:** An empty line should be used to identify when transmit data ends and when a send should occur. All data between the previous Send/Send and Close will be issued. In the case of TCP, the socket will remain opened after data is received.

**Send and Close:** A line containing `---` will operate the same as Send, only will close the Serial Port or TCP Socket once completed. The next line will be considered a configuration line.

**End of File:** Considered a Send and Close, only the application stops rather than reading a new configuration line.

### Output ###
The following defines the output generated when running a script.

    [T,192.168.1.200,4000,1000,250]
    >>
    01 23 45 67 89
    <<
    00 05
    
    >>
    A0 B1 C2 D3 F4 00 11 22
    33 44 55 66
    <<
    00 0C
    
    [T,192.168.1.200,4000,1000,250]
    >>
    00 01 02 03 04 05 06 07
    AA BB CC DD EE FF 01 23
    45 67
    !!
    Socket Closed
    
    >>
    00 01 02 03 04 05 06 07
    08 09 0A 0B 0C
    !!
    Receive Timeout

#### Start ####
When a session starts (or restarts) the configuration is printed in brackets. In cases where `*` is used in the script, the configuration used is still printed.

#### Communication ####

**Transmit Data:** Lines containing `>>` show the start of a transmit. The lines following contain the message sent and end in either an empty line, `<<` or `!!`.

**Received Data:** Lines containing `<<` show the start of a receive. The lines following contain the message received and end in either an empty line, `>>` or `!!`.

**Error Message:** Lines containing `!!` show the start of an error message. The lines following contain a description of the issue that occurred and end in either an empty line, `>>` or `<<`.

