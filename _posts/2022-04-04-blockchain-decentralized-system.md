## ELI5: how does Blockchain allow a system to be decentralized?

### What are centralized and decentralized services?

When we talk about decentralization the first thing we correlate is the currency that has been in hype for years: Bitcoin. Bitcoin is a virtual, decentralized currency that is distributed and stored on the Blockchain - the technology that enables decentralization.

A decentralized system is one that does not have a central entity/authority acting as an intermediary. The Brazilian Real, for example, is a centralized currency: the Brazilian Mint acts as the central authority and intermediary; it defines how many banknotes and coins will be made. They are the ones in total control of the currency.

Bitcoin has a finite number of copies and all its transactions are public and unchangeable. Because the network is run by several users (the miners), this makes it decentralized.

The theory of a decentralized monetary system is simple, as you can see. We can simply think of a currency that has all its transactions public, has no authorities acting as intermediaries, and somehow cannot have its transaction history changed. Before the invention of the Blockchain (which came along with the invention of Bitcoin, by Satoshi Nakamoto) we never had the ability to implement such a system in a technical way, that is, implementing and popularizing it was always the problem.

### Why create a decentralized monetary system?

Bitcoin was created by Satoshi Nakamoto (a pseudonym, since nobody knows his real identity) who, we all believe, did not support the idea of banks having access to all our transactions. Knowing this, the fact that we have a monetary system where there are no bank interventions is a remarkable advantage. Consequently, we have a decrease in fraud and/or manipulation.

A decentralized system, however, is commonly used for crimes. Since there is no one acting as an intermediary, remaining anonymous within the network is extremely simple; this is why criminals demand cryptocurrencies as a form of payment in most cases.

As a consequence of the anonymity, money laundering with cryptocurrencies has been used many times.

### Decentralized system in practice

Imagine that you, me, and Joãozinho want to create a system similar to Bitcoin. All the monetary transactions we make with each other are stored on a sheet of paper. At the end of each month, we pay what we owe each other.

![alt text](/images/2022-04-04-blockchain-decentralized-system/transaction.png)

This is, in a nutshell, how a Blockchain works with cryptocurrencies: It stores all transactions in public form. If we were all honest, this system would work perfectly: nobody would change the value of the transactions, the sender or the receiver of the transactions. But in this paper scenario, nothing prevents someone from taking a pen and adding a few extra zeros to Joãozinho's transaction.

![alt text](/images/2022-04-04-blockchain-decentralized-system/transaction-02.png)

Two things prevent the data in a Blockchain from being altered: its architecture, which will be commented on, and mining.

### Cryptocurrency Mining
All transactions on a Blockchain need to be validated. This means that if Bob transfers 1 BTC to Alice, the amount will not automatically be transferred to his wallet, because first it needs to be validated. The act of validating a transaction is what we call mining.

Cryptocurrency mining is done by miners. Miners are nothing more than users who place their computing power (hardware) on the Blockchain network. The more computing power, the faster the transactions will be validated.

Validating a transaction is validating a purposefully complex mathematical calculation (in the case of the Proof of Work validation method - protocol used by Bitcoin). Miners solve mathematical calculations that purposefully require a lot of computing power. This process, depending on the Blockchain, can take seconds, minutes, or even hours. When this mathematical calculation is completed by a miner, a new block in the chain (the data of a transaction) will be added and the miner will be rewarded for his computational effort. Only at that point will a transaction be completed.

In other words, the role of miners is simply to put their computing power to solve mathematical calculations. When these calculations are solved, the block is validated and can be added to the chain.

The purpose of these calculations being purposely complex is directly linked to the architecture of the Blockchain that must first be understood.

### Technically speaking, how does a Blockchain allow a system to be decentralized?

Imagine the Blockchain as the name implies: a chain of blocks connected by a chain.

Its architecture consists of "blocks" that have the reference of the previous block (chain) - like an interconnected chain - and each block has a stored information (a monetary transaction, for example). Having a reference to the previous block is what makes the Blockchain immutable, i.e., almost impossible to have its information changed.

![alt text](/images/2022-04-04-blockchain-decentralized-system/blockchain.png)

Each block contains the following information:

- Amount: the amount of Bitcoins to be transferred;

- Sender: the hash of the block that is sending the Bitcoins (sender);

- Recipient: the hash of the block that is receiving the Bitcoins;

- Hash: the hash of the current block;

- Previous hash: the hash of the previous block.

The first block (Block 1) is called the genesis block. As you can see, it has one less property: the previous hash. This is because this is the first record in the Blockchain, i.e., there is no previous block to store the hash. It is important to remember that the hash of each block is defined by the information it contains. If a piece of information changes, the hash of the block will automatically change as well.

The picture gives an example of a Blockchain with three transactions recorded. The first transaction was 10 Bitcoins for address 345, the second was 1 Bitcoin for address 678, and the last transaction was 20 Bitcoins for address 91011.

Now imagine that someone modifies the properties of Block 1. Instead of 10 Bitcoins sent to hash 345, 20 Bitcoins are reported in the record. Unlike the sheet of paper example, this would cause problems due to the previous hash property that all blocks - with the exception of the genesis block - have.

Since one value of Block 1 (amount) has changed (from 10 to 20), the hash of the block will automatically change (from 123 - its original value). When the hash of this block changes, the previous hash property of the next block will also change. If this property changes, the original hash of this block will also change, and so on, causing a domino effect throughout the Blockchain. Automatically, the value of all blocks in the chain will change, and consequently, will have to be validated again.

### Role of the miners

Since all the blocks in the chain will change if a single value is modified, the entire chain will have to be validated again. As already mentioned, mining a single block in a chain can be a very time consuming task.

By default, the blockchain that will be considered true will be the one with the most blocks validated. Because of this, you would not be able to change the values of the records by yourself because you would need to validate all the blocks in the Blockchain again. Doing this successfully means having more computing power than all the miners in the entire world - something that is unlikely. In 2022, there are about 1,000,000 Bitcoin miners around the world. They all work non-stop to validate new blocks on the network. Since they all work around the clock validating blocks in a single chain, being able to change records and validate more blocks than they do is unlikely because you are at a computational power disadvantage.

The only possible way to successfully change the values of a blockchain is to validate more blocks than all 1,000,000 miners combined - which is physically almost impossible, since you would need to have a computational power as high as all the computational power of all the miners working in the chain.

In other words, the more miners working to validate blocks in the chain, the more computational power is required for a single person to change the records.

This is a very interesting approach because successfully changing records depends on something physical (hardware), not digital.

### The only way to change the records of a Blockchain - 51% Attack

51% Attack is a type of attack where one person or a group of people have more computing power (51% +) than the power of all the miners together working on the chain. In this scenario, the individual could change one value in the chain and validate the entire chain again. Since he has a higher computational power than all the other miners combined, he could continue validating the chain and consequently validate more blocks; which would make the chain the largest in the blockchain. If this happened, it would be considered the true chain even though it contains changed data.

This means that you could only successfully alter the records of a blockchain if you mastered at least 51% of the computing power working on the chain.

### Energy consumption for transaction validation

In May 2021, Elon Musk - one of today's biggest cryptocurrency influencers - tweeted about Tesla, his electric vehicle company, no longer accepting Bitcoin as a form of payment.

![alt text](/images/2022-04-04-blockchain-decentralized-system/dk13xl5ejry61.png)

According to him, the rapid growth in the use of fossil fuels for mining the cryptocurrency worries him.

Bitcoin started to have a lot of visibility a few years ago, which made many people enter the mining market. Consequently, the demand for power and GPUs have increased dramatically, which directly influences their markets.

![alt text](/images/2022-04-04-blockchain-decentralized-system/15843.jpeg)

It is estimated that to validate a single Bitcoin transaction, about 2100 kWh of energy is used, which is roughly what an average American household consumes in 75 days.

All the rising energy prices have caused miners to opt for cheaper alternatives for mining, such as the fossil fuel that was commented on by Elon Musk. Because of the competition, miners are looking for cheaper options, but they are the most polluting, thus directly affecting the environment.

As a solution to this problem, new validation mechanisms are already being developed and even used, as is the case of Proof-of-Stake, a mechanism that will be used by the Ethereum network and that does not depend on computing power for validations.

### References

- https://pt.wikipedia.org/wiki/Blockchain
- https://ethereum.org/en/defi/
- https://ethereum.org/en/developers/docs/consensus-mechanisms/pos/
- https://bitcoin.org/bitcoin.pdf
- https://ethereum.org/669c9e2e2027310b6b3cdce6e1c52962/Ethereum_White_Paper_-_Buterin_2014.pdf
