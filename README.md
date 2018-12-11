# Implementing a Reliable Transport Protocol (from J.F. Kurose)

## Guidelines

Project is divided into 2 parts, _Alternating-Bit-Protocol_ version and the second part for the _Go-Back-N_ version.

## Test cases

- No loss and no error scenario to demonstrate that the code works under no error case: <br>
  Enter the number of messages to simulate: 10 <br>
  Enter packet loss probability [enter 0.0 for no loss]: 0.0 <br>
  Enter packet corruption probability [0.0 for no corruption]: 0.0 <br>
  Enter average time between messages from sender's layer5 [ > 0.0]: 5.0 <br>
  Enter TRACE: 0 <br>

- 30 percent loss and no error scenario to demonstrate the code can recover the lossed packets: <br>
  Enter the number of messages to simulate: 50 <br>
  Enter packet loss probability [enter 0.0 for no loss]: 0.3 <br>
  Enter packet corruption probability [0.0 for no corruption]: 0.0 <br>
  Enter average time between messages from sender's layer5 [ > 0.0]: 10.0 <br>
  Enter TRACE: 0 <br>

- No loss and 30 percent corruption scenario to demonstrate the code can recover the erroneously received packets: <br>
  Enter the number of messages to simulate: 10 <br>
  Enter packet loss probability [enter 0.0 for no loss]: 0.0 <br>
  Enter packet corruption probability [0.0 for no corruption]: 0.3 <br>
  Enter average time between messages from sender's layer5 [ > 0.0]: 10.0 <br>
  Enter TRACE: 0 <br>

## Overview

The procedures we will write are for the sending entity (A) and the receiving entity (B). Only unidirectional transfer of data (from A to B) is required. Of course, the B side will have to send packets to A to acknowledge (positively or negatively) receipt of data. Our routines are to be implemented in the form of the procedures described below. These procedures will be called by (and will call) procedures that the text book author has written which emulate a network environment. The overall structure of the environment is shown in the attached figure:

![structure of the emulated environment](https://user-images.githubusercontent.com/25902120/49824078-eccced00-fd89-11e8-92cf-6dd855620b37.jpg)

The unit of data passed between the upper layers and our protocols is a message, which is declared as:

```
struct msg {
  char data[20];
};
```

This declaration, and all other data structure and emulator routines, as well as stub routines (i.e., those you are to complete) are in the attached file, prog2.c, described later. Our sending entity will thus receive data in 20-byte chunks from layer5; our receiving entity should deliver 20-byte chunks of correctly received data to layer5 at the receiving side.

The unit of data passed between our routines and the network layer is the packet, which is declared as:

```
struct pkt {
  int seqnum;
  int acknum;
  int checksum;
  char payload[20];
};
```

Our routines will fill in the payload field from the message data passed down from layer5. The other packet fields will be used by our protocols to insure reliable delivery, as we've seen in class.

The routines we will write are detailed below. As noted above, such procedures in real-life would be part of the operating system, and would be called by other procedures in the operating system.

`A_output(message)` where message is a structure of type msg, containing data to be sent to the B-side. This routine will be called whenever the upper layer at the sending side (A) has a message to send. It is the job of our protocol to insure that the data in such a message is delivered in-order, and correctly, to the receiving side upper layer.

`A_input(packet)` where packet is a structure of type pkt. This routine will be called whenever a packet sent from the B-side (i.e., as a result of a tolayer3() being done by a B-side procedure) arrives at the A-side. packet is the (possibly corrupted) packet sent from the B-side.

`A_timerinterrupt()` This routine will be called when A's timer expires (thus generating a timer interrupt). We'll probably want to use this routine to control the retransmission of packets. See starttimer() and stoptimer() below for how the timer is started and stopped.

`A_init()` This routine will be called once, before any of our other A-side routines are called. It can be used to do any required initialization.

`B_input(packet)` where packet is a structure of type pkt. This routine will be called whenever a packet sent from the A-side (i.e., as a result of a tolayer3() being done by a A-side procedure) arrives at the B-side. packet is the (possibly corrupted) packet sent from the A-side.

`B_init()` This routine will be called once, before any of our other B-side routines are called. It can be used to do any required initialization.

## Software Interfaces

The procedures described above are the ones that we will write. The text book author has written the following routines which can be called by our routines:

`starttimer(calling_entity,increment)` where calling_entity is either 0 (for starting the A-side timer) or 1 (for starting the B side timer), and increment is a float value indicating the amount of time that will pass before the timer interrupts. A's timer should only be started (or stopped) by A-side routines, and similarly for the B-side timer. To give we an idea of the appropriate increment value to use: a packet sent into the network takes an average of 5 time units to arrive at the other side when there are no other messages in the medium.

`stoptimer(calling_entity)` where calling_entity is either 0 (for stopping the A-side timer) or 1 (for stopping the B side timer).

`tolayer3(calling_entity,packet)` where calling_entity is either 0 (for the A-side send) or 1 (for the B side send), and packet is a structure of type pkt. Calling this routine will cause the packet to be sent into the network, destined for the other entity.

`tolayer5(calling_entity,message)` where calling_entity is either 0 (for A-side delivery to layer 5) or 1 (for B-side delivery to layer 5), and message is a structure of type msg. With unidirectional data transfer, we would only be calling this with calling_entity equal to 1 (delivery to the B-side). Calling this routine will cause data to be passed up to layer 5.

## The simulated network environment

A call to procedure `tolayer3()` sends packets into the medium (i.e., into the network layer). Oour procedures `A_input()` and `B_input()` are called when a packet is to be delivered from the medium to our protocol layer.

The medium is capable of corrupting and losing packets. It will not reorder packets. When we compile our procedures and the author'sprocedures together and run the resulting program, we will be asked to specify values regarding the simulated network environment:

**Number of messages to simulate**. the author'semulator (and our routines) will stop as soon as this number of messages have been passed down from layer 5, regardless of whether or not all of the messages have been correctly delivered. Thus, we need not worry about undelivered or unACK'ed messages still in our sender when the emulator stops. Note that if we set this value to 1, our program will terminate immediately, before the message is delivered to the other side. Thus, this value should always be greater than 1.

**Loss**. We are asked to specify a packet loss probability. A value of 0.1 would mean that one in ten packets (on average) are lost.
Corruption. We are asked to specify a packet loss probability. A value of 0.2 would mean that one in five packets (on average) are corrupted. Note that the contents of payload, sequence, ack, or checksum fields can be corrupted. We checksum should thus include the data, sequence, and ack fields.

**Tracing**. Setting a tracing value of 1 or 2 will print out useful information about what is going on inside the emulation (e.g., what's happening to packets and timers). A tracing value of 0 will turn this off. A tracing value greater than 2 will display all sorts of odd messages that are for the author'sown emulator-debugging purposes. A tracing value of 2 may be helpful to we in debugging our code. We should keep in mind that real implementors do not have underlying networks that provide such nice information about what is going to happen to their packets!

**Average time between messages from sender's layer5**. We can set this value to any non-zero, positive value. Note that the smaller the value we choose, the faster packets will be be arriving to our sender.

## The Alternating-Bit-Protocol Version of this project

We are to write the procedures, `A_output()`, `A_input()`, `A_timerinterrupt()`, `A_init()`, `B_input()`, and `B_init()` which together will implement a stop-and-wait (i.e., the alternating bit protocol, which we referred to as rdt3.0 in the text) unidirectional transfer of data from the A-side to the B-side. Our protocol should use both ACK and NACK messages.

We should choose a very large value for the average time between messages from sender's layer5, so that our sender is never called while it still has an outstanding, unacknowledged message it is trying to send to the receiver. I'd suggest we choose a value of 1000. Wou should also perform a check in our sender to make sure that when `A_output()` is called, there is no message currently in transit. If there is, we can simply ignore (drop) the data being passed to the `A_output()` routine.

This project can be completed on any machine supporting C. It makes no use of UNIX features. (We can simply copy the prog2.c file to whatever machine and OS we choose).

We recommend that we should hand in a code listing, a design document, and sample output. For our sample output, our procedures might print out a message whenever an event occurs at our sender or receiver (a message/packet arrival, or a timer interrupt) as well as any action taken in response. We might want to hand in output for a run up to the point (approximately) when 10 messages have been ACK'ed correctly at the receiver, a loss probability of 0.1, and a corruption probability of 0.3, and a trace level of 2. We might want to annotate our printout with a colored pen showing how our protocol correctly recovered from packet loss and corruption.

## The Go-Back-N version of this project

We are to write the procedures, `A_output()`, `A_input()`, `A_timerinterrupt()`, `A_init()`, `B_input()`, and `B_init()` which together will implement a Go-Back-N unidirectional transfer of data from the A-side to the B-side, with a window size of 8. Our protocol should use both ACK and NACK messages. Consult the alternating-bit-protocol version of this project above for information about how to obtain the network emulator.

We would STRONGLY recommend that we first implement the easier project (Alternating Bit) and then extend our code to implement the harder project (Go-Back-N). Believe me - it will not be time wasted! However, some new considerations for our Go-Back-N code (which do not apply to the Alternating Bit protocol) are:

`A_output(message)`, where message is a structure of type msg, containing data to be sent to the B-side.
Our `A_output()` routine will now sometimes be called when there are outstanding, unacknowledged messages in the medium - implying that we will have to buffer multiple messages in our sender. Also, we'll also need buffering in our sender because of the nature of Go-Back-N: sometimes our sender will be called but it won't be able to send the new message because the new message falls outside of the window.

Rather than have we worry about buffering an arbitrary number of messages, it will be OK for we to have some finite, maximum number of buffers available at our sender (say for 50 messages) and have our sender simply abort (give up and exit) should all 50 buffers be in use at one point (Note: using the values given below, this should never happen!) In the "real-world", of course, one would have to come up with a more elegant solution to the finite buffer problem!

`A_timerinterrupt()` This routine will be called when A's timer expires (thus generating a timer interrupt). Remember that we've only got one timer, and may have many outstanding, unacknowledged packets in the medium, so we'll have to think a bit about how to use this single timer.

Consult the Alternating-bit-protocol version of this project above for a general description of what we might want to hand in. We might want to hand in output for a run that was long enough so that at least 20 messages were successfully transfered from sender to receiver (i.e., the sender receives ACK for these messages) transfers, a loss probability of 0.2, and a corruption probability of 0.2, and a trace level of 2, and a mean time between arrivals of 10. We might want to annotate parts of our printout with a colored pen showing how our protocol correctly recovered from packet loss and corruption.

## For extra credit

You can implement bidirectional transfer of messages. In this case, entities A and B operate as both a sender and receiver. You may also piggyback acknowledgments on data packets (or you can choose not to do so). To get the author'semulator to deliver messages from layer 5 to your B_output() routine, you will need to change the declared value of BIDIRECTIONAL from 0 to 1.

## Helpful Hints and the like

**Checksumming**. You can use whatever approach for checksumming you want. Remember that the sequence number and ack field can also be corrupted. We would suggest a TCP-like checksum, which consists of the sum of the (integer) sequence and ack field values, added to a character-by-character sum of the payload field of the packet (i.e., treat each character as if it were an 8 bit integer and just add them together).

**Note that any shared state** among your routines needs to be in the form of global variables. Note also that any information that your procedures need to save from one invocation to the next must also be a global (or static) variable. For example, your routines will need to keep a copy of a packet for possible retransmission. It would probably be a good idea for such a data structure to be a global variable in your code. Note, however, that if one of your global variables is used by your sender side, that variable should NOT be accessed by the receiving side entity, since in real life, communicating entities connected only by a communication channel can not share global variables.

**There is a float global variable** called time that you can access from within your code to help you out with your diagnostics msgs.

**START SIMPLE**. Set the probabilities of loss and corruption to zero and test out your routines. Better yet, design and implement your procedures for the case of no loss and no corruption, and get them working first. Then handle the case of one of these probabilities being non-zero, and then finally both being non-zero.

**Debugging**. We'd recommend that you set the tracing level to 2 and put LOTS of printf's in your code while your debugging your procedures.

**Random Numbers**. The emulator generates packet loss and errors using a random number generator. Our past experience is that random number generators can vary widely from one machine to another. You may need to modify the random number generation code in the emulator we have suplied you. Our emulation routines have a test to see if the random number generator on your machine will work with our code.
If you get an error message:
It is likely that random number generation on your machine is different from what this emulator expects. Please take a look at the routine `jimsrand()` in the emulator code. Sorry. then you'll know you'll need to look at how random numbers are generated in the routine `jimsrand()`; see the comments in that routine.
