# Trusted Web Protocol: a decentralised data packet sending system
**Extract:** All of the data networks available today have serious flaws,vulnerabilities or inconveniences. We propose a solution that could, in asignificant (if not all) of the cases, replace these methods by eliminatingtheir flaws. The network is similar to the torrent protocol in manyways, but its users do not have to contribute to its operation at all. Tomake it worthwhile for some people to continue to maintain thenetwork, the network issues a cryptocurrency. The nodes that forwardthe data packets are rewarded for the amount of data they forward, andtheir work is verified by other nodes through a consensus process calledproof-of-success. Data packet headers and transactions through thenetwork would be stored in a blockchain and a procedure would beused to maintain the pseudo-randomness of the network selectionmechanism, thus minimizing the number of possible abuses.

## Introduction
Today, there are 3 common methods for sharing data packets over the Internet. Thesimplest is the peer-to-peer model, where two computers can communicate directlywithout a third party. This is undoubtedly the most secure solution, but unfortunately itis severely limited by the need for all parties to be online from start to finish of thedata transfer, and by the difficulty of implementing it with a large number of users.This problem is solved by the server-client model, which has enabled giants such asAmazon, Google and Microsoft to penetrate every part of our lives. While centralisedserver-based solutions are mostly subscription-based, the last decade has seen theproliferation of targeted advertising, which means that Facebook and Google nowknow more about us than our friends. Cyber attacks in recent years have shown thatour data is far from in good hands, partly due to human error and governmentregulations that require service providers to give intelligence and law enforcementagencies access to data. In order to use the future ubiquitous internet safely, we need atransparent network, independent of any centralised third party, which can at the sametime provide theand does not require any consent from the user side. In this paper, we propose asolution that sends data packets through a number of pseudo-randomly chosen nodesin exchange for a coin issued by a network, and the computers that oppose their workjustify their position by a certain amount of committed coins. The system can remain secure as long as the benign nodes have more coins than the malicious ones.

## Structure of the network
As shown in the figure below, the network is composed of 4 types of actors: endpoints
(i.e. users), relay servers, validators and leader nodes. The soul of the network is the
leader nodes, through which the connection data is transmitted to the nodes. The relay
servers are responsible for forwarding the data packets, while the validators, which
operate in groups, communicate to the leader nodes the success of the data packets in
reaching their destinations.

![twp_flowchart](https://github.com/Project-Axolotl/whitepaper/assets/63842039/6bae4ff0-cd7d-4397-84b2-0e20aada66ba)

## The data packages
Binary data packets of arbitrary size are delivered to the receiver in multiple chunks.This has two advantages. Firstly, nodes cannot be forced to delete the data that passesthrough them, so if the cryptographic algorithm used for encryption were to be crackedor a malicious party were to gain a significant quantum advantage - it would take 8hours to crack a 2048-bit RSA key with 20 million quantum bits - the contents of allmessages ever sent could potentially be leaked. In addition, breaking messages intomultiple chunks helps distribute the load across nodes, thereby reducing networklatency. Since encryption with RSA is severely limited both in time and in maximumsize, we propose a solution, already used in several places, referred to as hybridencryption, where the data itself is encrypted using a symmetric method (e.g., AES)and then RSA public-private key-based encryption is applied on the significantlyshorter key. It is important that the fragments in a stream encryption method are notcontiguous bits, as meaningful details can be extracted from them if the key is known.Instead, it makes sense to extract bits based on a pattern known to both parties, thusavoiding this problem. A further problem is that if for some reason any chunk of themessage fails to reach its destination, decoding will be impossible, so it is important toallow for redundancy by routing individual chunks to multiple nodes.

## Proof-of-success
To reward relay servers fairly, we must assume that relay servers are malicious, i.e.
they want to minimize the amount of data transferred and maximize the reward. In
order to check whether the data packet actually reaches the receiver, we assign a set of
n servers to each packet, who then decide on a consensus basis whether the delivery of
the data packet was successful. Only the IP addresses of the validators and the receiver
are transmitted to the relay server, so the relay server does not know which one is the
validator and which one is the receiver. However, it would still be sufficient to
forward the data packet to n/2+1 IP addresses, which in turn does not ensure that the
receiver will receive the message. These odds can be overridden by randomly
allocating n/2 more votes among the validators, and since the relay server is not aware
which validators have more votes, it is forced to forward the data packet to all IP
addresses. In addition, we can further reduce the susceptibility to fraud by crediting the
rewards collected by the relay servers only after a certain amount of data has been
transmitted and if they try to cheat the system, they lose the locked amount.

## Blockchain
The process, first described by Satoshi Nakamoto in his Bitcoin white paper, is
nowadays mainly used for cryptocurrencies. The idea is to sort the data into blocks
(e.g. incoming transactions in the last 3 minutes), apply a hash function to this block
and hash the next block with the result. The result is that if we change even one bit in
any element of the chain, we get a completely different result at the end of the chain.
Since today's average CPU can perform thousands of hashes, it is easy to iterate
through the c h a i n and check its correctness. In the case of our network, we store not
only transactions, but also data packet headers, which include the sender, receiver,
number of bursts, and packet length. Thanks to blockchain technology, this data can be
maintained securely without being altered, thus guaranteeing the correctness of the
reward.

## Selection procedure and Proof-of-Stake
The system can only maintain reliable, regular operation without the involvement of
third parties if an easily definable, traceable and universally definable procedure is
used to select each node. The basis of the procedure is a hash function (in our case, we
propose SHA-256). The goal of the procedure is to select a pseudorandom but
unambiguous node from a certain data set, i.e. a node with the same parameters and
always the same result. For the selection of leader nodes, only the hash of the previous
block is input, but for the selection of relay servers and validators, the header of the
given dataset is included, since their selection is for a single dataset, not for the
duration of a whole block. From the hash obtained from this data set, we hash off a
fraction of the size of the largest integer that can be formed, which is at least a few
orders of magnitude larger than the expected total number of nodes in the network. Let
this size be 32 bits for the time being. For each node reporting for that position, we
allocate an index in sequence from zero, and take the modulus of the 32-bit number
subtracted from the beginning of the hash and the number of nodes reporting for that
position, which will be the index of the node thus selected. The method can be used to
select multiple nodes that are pseudorandomly but uniquely selected, for example an
entire validator group, by appending indices from zero to the original data set before
hashing. The Python code of the procedure looks like this (names: `n1`, `n2`, `s1`, `s2` =
`number of nodes needed`, `total number of nodes`, `last block hash`, `message header`):
```py
def chooseNodes(n1, n2, s1, s2 = ''):
  chooseNodeIndexes = [];
  for i in range(n1):
    choosenNodeIndexes.append(int(SHA256(s1+s2+i)[0:32]) % n2);
  return choosenNodeIndexes;
```
Relay servers cannot harm the network, but the leader nodes and the valiadictershave a direct impact on the blockchain, so trust is essential. For manycryptocurrencies, trust is provided by a process called Proof-of-Work, i.e. a randomnumber is placed in the block, which causes the hash of the block to start with a givennumber of 0 bits. Whoever finds this random number first gets the reward and iscredited by the network. This means that the individual with the greatestcomputational power dominates the network, which means that in the near future,individuals with quantum dominance could easily take control of the network. It is alsoextremely energy and time intensive, making it difficult to sustain in the long term. Inour case, the network requires a certain amount of coins to be deposited as a token oftrust. Since validators and leader nodes work in groups, the consensus result of thegroups will be permanent on the blockchain, while those who voted against themajority will lose their bound coins, and those who voted with the majority willreceive interest on the coins. The only modification to the procedure described aboveis that an individual can be added to the lists more than once, so the more coinssomeone has, the more likely they are to be selected and the more rewards they canearn. This means that the person with more than half of the coins, which is virtuallyimpossible to obtain, can gain complete control of the network.

## Conclusion
In this document we have proposed a communication protocol that could be used to
replace today's solutions without the need for centralised servers. Relay servers would
be responsible for transmitting data packets, for which they would be rewarded with
digital coins. Their work would be verified by a team of validators and lead nodes
would record the results on a blockchain and coordinate the transactions that pass
through the network. The validators and lead nodes would tie up coins, which they
could lose if they voted against the majority and earn interest if they voted for the
majority. For this reason, as long as more than half of the coins in the network are in
the hands of benign nodes and validators, the safe operation of the system is
guaranteed.
