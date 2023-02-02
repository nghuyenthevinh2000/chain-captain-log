# How distribution?

"One tree cannot amount to anything, three of them makes a mountain"

A distributed system needs strong coordination to achieve the result like one unified system. There are many goals that a distributed system can try to achieve, but **this research will focus on fault - tolerance**.

Learning from: https://disco.ethz.ch/courses/hs21/distsys/

# Distributed system problems
1. message passing: n nodes send message to each others
    * message loss: message is not guaranteed that it will arrive safely at the receiver.
    * variable message delay: messages might experience different transmission times (can lead to timeout)
2. state replication: all nodes execute a (potentially infinite) sequence of commands c1, c2, c3, . . . , in the same order so as to achieve same state.
    * centralized serializer: a serializer is required to send commands in correct order. It will become a centralized failure point in a distributed system. How to distribute serializer?

The rest of this research will go deeper into these two problems
1. message passing
2. distributed state replication
    * [paxos: a solution for distributed state replication](paxos.md)