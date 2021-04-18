## Best resources to start with Blockchain

## How to start with Blockchain?

I usually write tutorials here but this post is going to be different: I am going to share my personal opinion about the best resources to learn blockchain. I do this because I got a comment under one of my earlier post asking: 

_what are the best resources for beginners to start with blockchain?_

Blockchain is a broad term. Let's focus on Ethereum first.

## Ethereum

If you want to learn the concept you can start with Austin Griffith's  [playlist](https://www.youtube.com/playlist?list=PLJz1HruEnenCXH7KW7wBCEBnBLOVkiqIi). Pro tip: start from the second video as the first one might look intimidating (i.e. sounds like gibberish to beginners).
Once you understand the very basics like hashing, gas, addresses, contracts, etc... you can move to  [another great video of him](https://www.youtube.com/watch?v=0vAKP3Y-BLs), where he explains  his project, [scaffold-eth](https://github.com/austintgriffith/scaffold-eth) to a couple of guys. Scaffold-eth is definitely worth experimenting with. Be careful though as running the built-in blockchain will freeze a weaker computer after a while.

[Dappuniversity](https://www.dappuniversity.com/) is another great place to start. It is absolutely very beginner-friendly. On his [youtube channel](https://www.youtube.com/channel/UCY0xL8V6NzzFcwzHCgB8orQ)  he meticulously explains every details as no prior blockchain  experience is assumed. He builds dapps (decentralized applications) step-by-step with frontend. The frontend he uses is the class-based, nowadays slightly outdated version of React, though. I did not buy his paid courses so I can't judge them but the free material on his channel is definitely a smooth start for beginners.

On Ethereum the language for smart contracts is [solidity](https://docs.soliditylang.org/). At the first glance it resembles javascript. You can learn solidity on [this interactive site](https://cryptozombies.io/). I learned solidity by going through this interactive tutorial. I even built an oracle. But whenever I need a 
reference or an example https://solidity-by-example.org/ is my first place to visit.

As for tools I can recommend Hardhat. They have a cool, up-to-date [tutorial](https://hardhat.org/tutorial/). Other developers prefer Truffle. They also have great [tutorials](https://www.trufflesuite.com/tutorials).

As you progress on your blockchain-journey sooner or later you'll bump into the [oracle
problem](https://blog.chain.link/what-is-the-blockchain-oracle-problem/). You will realize that smart contracts can not access data from the outside world through `http request` as almost any other application can. Instead, smart contracts pull data through something called an [oracle](https://medium.com/@teexofficial/what-are-oracles-smart-contracts-the-oracle-problem-911f16821b53). This will lead you to chain.link. [Here](https://docs.chain.link/docs?_ga=2.29853180.2106070822.1618745119-351727.1615024782) is the best place to start.

Finally I would like to mention another interesting youtube channel, [EatTheBlocks](https://www.youtube.com/channel/UCZM8XQjNOyG2ElPpEUtNasA), run by a French guy. This channel has plenty of videos which look quite promising but I did not really have the time to watch. Tell me your opinion in the comments if you watched them.

## Beyond Ethereum

### Hyperledger

As blockchains are moving more and more to the direction of being the dominant form of digital agreement I expect more and more companies to set up their own personalized and private blockchains. 
[Hyperledger](https://www.hyperledger.org/learn) itself is not a blockchain, nor a cryptocurrency. It is a software, a project within the Linux foundation to create one's own personalized, private blockchain service.The good news is that you don't have to learn a new language as it is available in `java`, `javascript` and `go`. The best place to start is [here](
https://hyperledger-fabric.readthedocs.io/en/release-2.2/getting_started.html). They are also present on [youtube](https://www.youtube.com/channel/UC7_X0WkMtkWzaVUKF-PRBNQ). 

### Internet Computer
They are the next step of the blockchain evolution. [Dfinity's Internet Computer](https://dfinity.org/) is a fundamentally different way of persisting data. They want to create a public network that can be used as a **complete replacement for today’s legacy IT stack**, including Big Tech’s cloud services, and legacy infrastructure software such as file systems, web servers, middleware, and databases.
 It is basically an extension of internet that allows developers to build and deploy software directly on the internet without databases or third party cloud providers(AWS, etc...). With their blockchain-based architecture they want to completely re-define how the internet works and their vision is to “reboot” the internet in a way that destroys the ability to create virtual monopolies like Facebook, LinkedIn, Instagram and WhatsApp. Despite the whole thing is based on blockchain their users will not necessarily know it. They don't have to install Metamask or any other wallet. They can access the application the same way as any other website (i.e. there is no gas fee). The only bad news about Internet Computer is that you need to learn [yet another language](https://sdk.dfinity.org/docs/language-guide/motoko.html). Does it worth the effort? Probably yes, because even if they can only hold half of their [promises](https://medium.com/dfinity/announcing-internet-computer-mainnet-and-a-20-year-roadmap-790e56cbe04a) they will eventually grow huge. The best place to start is [here](https://sdk.dfinity.org/docs/developers-guide/tutorials-intro.html).

### IPFS

The [InterPlanetary File System (IPFS)](https://ipfs.io/) is not a blockchain. It is a protocol and peer-to-peer network for storing and sharing data in a distributed file system. But it is perfectly [works together with Ethereum](https://medium.com/@austin_48503/tl-dr-scaffold-eth-ipfs-20fa35b11c35) to store large amount of data when storing it otherwise would be either cost-prohibitive or not decentralized.  
I also wrote [this](https://fullstackwithpr.hashnode.dev/scribble-on-canvas-and-sell-it-as-nft-react-truffle-ipfs-opensea) and [this](https://fullstackwithpr.hashnode.dev/decentralized-censorship-resistant-instagram-clone-tutorial-ethereum-hardhat-ethersjs-ipfs-react) tutorials which stored data on IPFS.









 






