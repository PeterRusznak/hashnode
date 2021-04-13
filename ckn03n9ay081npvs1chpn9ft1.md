## Decentralized, censorship-resistant  Instagram-clone Tutorial (Ethereum, Hardhat, Ethers.js, IPFS, React )

## The Big Picture

We are at the tipping point of a new phase in the web’s evolution.
I believe that the [recent de-platforming of some prominent persons](https://blog.twitter.com/en_us/topics/company/2020/suspension.html)   will just excelerate the progress. The old, centralized way where data is mostly stored in centralized repositories will give way to something new: Web 3.0.
Decentralization is just one pillar of Web 3.0 (along with AI and other things) but it is where my focus currently lies. 
In this tutorial I show you how to make a  fully decentralized, IPFS - backed Instagram-clone, where users can upload and tip images. I took my inspiration from https://www.dappuniversity.com/ but my code below is everything but a simple copy-paste:

* Dapp University uses  [truffle](https://www.trufflesuite.com/)  and  [ganache](https://www.trufflesuite.com/ganache)  to compile and deploy.  I use  [Hardhat](https://hardhat.org/)  and the built-in Hardhat Network that I found more convenient. Its functionality focuses around Solidity debugging, featuring stack traces, `console.log()` and explicit error messages when transactions fail.
* He used the  [Web3.js](https://web3js.readthedocs.io/en/v1.3.4/)  library to interact with the deployed contract and I used  [Ethers.js](https://docs.ethers.io/v5/)  instead. This is a major difference as these two libraries are vastly different. I am not yet qualified to say which one is better but Ethers.js is smaller and newer so it is definitely something we should have an eye on.
* I also had to modify the way how he connects to IPFS (which did not work for me) but it is probably due to some problems with my versioning which I could not figure out.
* Instead of class-based React I used the newer, function based version. 

The full code is available on  [Github](https://github.com/PeterRusznak/instaclone).

## Software versions:

* OS: Ubuntu 20.04.2 LTS
* node: v14.15.4
* npm: 6.14.10

## Installing dependencies

As usual, execute the following command: npx create-react-app instaclone to create React frontend application with the name of instaclone. We'll use  [Hardhat](https://hardhat.org/) to compile and deploy our contract, so install into your project folder, (i.e. _instaclone_):

```script
npm install --save-dev hardhat
```
We also need some other dependecies:
```script
npm install --save-dev @nomiclabs/hardhat-waffle ethereum-waffle chai @nomiclabs/hardhat-ethers
```
and also:
```script
npm install ethers bootstrap identicon.js ipfs-api
```

When all the installation is done create your Hardhat project by running: `npx hardhat`. 

Select `Create an empty hardhat.config.js` with your keyboard and hit enter. When Hardhat is run, it searches for the closest `hardhat.config.js` file starting from the current working directory. This file normally lives in the root of your project and an empty `hardhat.config.js` is enough for Hardhat to work. The entirety of your setup is contained in this file.

Replace the 'hardhat.config.js` file content with the following:
```javascript
require("@nomiclabs/hardhat-waffle");

/**
 * @type import('hardhat/config').HardhatUserConfig
 */
module.exports = {
  solidity: "0.7.3",
};
```

## Creating the Smart Contracts

Start by creating a new directory called `contracts` and create a file inside the directory called `Instaclone.sol`. The language for smart contracts is  [solidity](https://solidity-by-example.org/).
I believe that anyone with some javascript or maybe java knowledge can decipher the meaning of a simple contract. I am not saying that solidity _is_ java or javascript but I do say that if you have some years of javascript experience the transition to solidity should not be super bumpy.

The file's content should be the following:

```
pragma solidity ^0.7.0;

contract Instaclone {
    string public name;
    uint256 public imageCount = 0;
    mapping(uint256 => Image) public images;

    struct Image {
        uint256 id;
        string hash;
        string description;
        uint256 tipAmount;
        address payable author;
    }

    event ImageCreated(
        uint256 id,
        string hash,
        string description,
        uint256 tipAmount,
        address payable author
    );

    event ImageTipped(
        uint256 id,
        string hash,
        string description,
        uint256 tipAmount,
        address payable author
    );

    constructor() public {
        name = "Instaclone";
    }

    function uploadImage(string memory _imgHash, string memory _description)
        public
    {
        // Make sure the image hash exists
        require(bytes(_imgHash).length > 0, "Must have HASH");
        // Make sure image description exists
        require(bytes(_description).length > 0, "Must have DESCRIPTION");
        // Make sure uploader address exists
        require(msg.sender != address(0), "Must have AUTHOR");

        // Increment image id
        imageCount++;

        // Add Image to the contract
        images[imageCount] = Image(
            imageCount,
            _imgHash,
            _description,
            0,
            msg.sender
        );
        // Trigger an event
        emit ImageCreated(imageCount, _imgHash, _description, 0, msg.sender);
    }

    function tipImageOwner(uint256 _id) public payable {
        // Make sure the id is valid
        require(_id > 0 && _id <= imageCount, "NOT EXISTING IMAGE");
        // Fetch the image
        Image memory _image = images[_id];
        // Fetch the author
        address payable _author = _image.author;
        // Pay the author by sending them Ether
        payable(address(_author)).transfer(msg.value);
        // Increment the tip amount
        _image.tipAmount = _image.tipAmount + msg.value;
        // Update the image
        images[_id] = _image;
        // Trigger an event
        emit ImageTipped(
            _id,
            _image.hash,
            _image.description,
            _image.tipAmount,
            _author
        );
    }
}
```
The contract has two interesting functions. The first one is `uploadImage()` Its name speaks for itself. However we won't upload the image itself to the smart contract, as it would be extremely expensive.
 [To store 1GB of data inside a smart contract on the Ethereum network would cost millions of dollars](https://medium.com/ipdb-blog/forever-isnt-free-the-cost-of-storage-on-a-blockchain-database-59003f63e01#:~:text=Storing%20and%20sending%207KB%20of,around%20%244%2C672%2C500%20at%20today%27s%20prices.). 
That's why we (the frontend) will upload the image to IPFS because IPFS returns a hash that uniquely identifies the uploaded image. Then we (frontend) can call the smart contract and upload and store only this hash. The `uploadImage()` function takes the IPFS-hash plus a string as description. The function validates that the uploaded data actually contains a hash and description (that's the job of the lines starting with`require`) and store the data in a `struct` which is similar to javascript objects. There is a class level variable called `imageCount` reflecting the number of uploaded images. We need to increment this variable. Finally the function emits an event to notify the calling frontend.

The second function is `tipImageOwner()`. It takes an argument which is the id of the image in question. It gets the relevant image from the struct which also contains the image's author's address. It is important to note that this function should be called only in a way that also sends some ether. We can get the exact amount of ether by `msg.value` (just like `msg.sender` reveals the caller's address). This amount then will be deducted from the caller's account and added to the author's account.
Finally the updates the struct and emits an event. 

## Compiling

Compilation is simple. Just run `npx hardhat compile`. The process creates a Json file based on the contract. 
![HardhatArtifacts.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1617288094746/5DQrAul21.png)

We need an _ABI_ and an address to interact with any smart contract. This Json file contains the _ABI_.

![ABI.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1617288454108/cMa5YPRNc.png)
That's why I will copy it to the `src` folder.

## Testing the smart contract

Writing tests when building smart contracts is of crucial importance, as your user's money is what's at stake. Smart contracts are immutable, there is no way to _modify_ them, we can only redeploy them. But that costs money. In this case we're going to use Hardhat Network, a local Ethereum network designed for development that is built-in and the default network in Hardhat. No need to setup anything to use it. In our tests we're going to use ethers.js to interact with the Ethereum contract we built.

Create a new directory called `test` inside our project root directory and create a new file called `InstacloneTest.js`.  The content is the following:

```javascript
const { expect } = require("chai");

describe('Instaclone contract', () => {
    let InstacloneContract
    let instaclone
    let owner
    let author
    let tipper

    before(async () => {
        InstacloneContract = await ethers.getContractFactory("Instaclone");
        [owner, author, tipper, ...addrs] = await ethers.getSigners();
        instaclone = await InstacloneContract.deploy();
    })

    describe('deployment', async () => {
        it('deploys successfully', async () => {
            const address = await instaclone.address
            expect(address).not.to.equal(0x0);
            expect(address).not.to.equal('');
            expect(address).not.to.equal(null);
            expect(address).not.to.equal(undefined);
        })

        it('has a name', async () => {
            const name = await instaclone.name()
            expect(name).to.equal("Instaclone");
        })
    })

    describe('images', async () => {
        let result, imageCount, ev, args
        const hash = 'QmZGQA92ri1jfzSu61JRaNQXYg1bLuM7p8YT83DzFA2KLH'

        before(async () => {
            result = await instaclone.connect(author).uploadImage(hash, 'Image description')
            ev = await result.wait()
            args = ev.events[0].args
            imageCount = await instaclone.imageCount()
        })

        //check event
        it('creates images', async () => {
            // SUCESS
            expect(imageCount.toNumber()).to.equal(1);
            expect(args.id.toNumber()).to.equal(imageCount.toNumber(), "id is correct");

            expect(args.hash).to.equal(hash, "hash OK");
            expect(args.description).to.equal('Image description', "Description OK");
            expect(args.author).to.equal(author.address, 'author is correct')

            // FAILURE: Image must have hash         
            await expect(
                instaclone.connect(author).uploadImage('', 'Image description')
            ).to.be.revertedWith("Must have HASH");

            // FAILURE: Image must have description
            await expect(
                instaclone.connect(author).uploadImage(hash, "")
            ).to.be.revertedWith("Must have DESCRIPTION");
        })

        //check from Struct
        it('lists images', async () => {
            const image = await instaclone.images(imageCount)
            expect(image.id.toNumber()).to.equal(imageCount.toNumber(), "id OK");
            expect(image.hash).to.equal(hash, 'Hash is correct')
            expect(image.tipAmount).to.equal(0, 'Amount is correct')
            expect(image.description).to.equal('Image description', 'description is correct')
            expect(image.author).to.equal(author.address, 'author is correct')
        })

        it('allows users to tip images', async () => {
            // Track the author balance before purchase
            let oldAuthorBalance = await ethers.provider.getBalance(author.address);
            let oldAuthorBalanceString = ethers.utils.formatEther(oldAuthorBalance)

            let weiValue = ethers.utils.parseEther('1')
            result = await instaclone.connect(tipper).tipImageOwner(imageCount, { value: weiValue });

            ev = await result.wait()
            args = ev.events[0].args
            imageCount = await instaclone.imageCount()

            expect(args.id.toNumber()).to.equal(imageCount.toNumber(), "id NEM OK");
            expect(args.hash).to.equal(hash, 'Hash is correct')
            expect(args.description).to.equal('Image description', 'Hash is correct')
            expect(args.tipAmount).to.equal(weiValue, 'Tip amount is correct')
            expect(args.author).to.equal(author.address, 'Tip amount is correct')

            // Check that author received funds
            let newAuthorBalance = await ethers.provider.getBalance(author.address);
            let newAuthorBalanceString = ethers.utils.formatEther(newAuthorBalance)
            console.log(newAuthorBalanceString)

            let tip = ethers.utils.parseEther('1')
            const expectedBalance = oldAuthorBalance.add(tip)
            expect(newAuthorBalanceString).to.equal(ethers.utils.formatEther(expectedBalance), "Author account INCREASED")
            // FAILURE: Tries to tip a image that does not exist
            await expect(
                instaclone.connect(tipper).tipImageOwner(9999, { value: weiValue })
            ).to.be.revertedWith("NOT EXISTING IMAGE");

        })
    })
})
```
Couple of thing here: it needs  [chai](https://www.chaijs.com/). Nested `describe`s are OK. 
This is how we deploy the contract and get the testing account with Ethers.js:
```javascript
 InstacloneContract = await ethers.getContractFactory("Instaclone");
 [owner, author, tipper, ...addrs] = await ethers.getSigners();
 instaclone = await InstacloneContract.deploy();
```

This is how we call a function without sending money:
```javascript
result = await instaclone.connect(author).uploadImage(hash, 'Image description')
ev = await result.wait()
args = ev.events[0].args
...
expect(args.description).to.equal('Image description', "Description OK");
```

If we do want to send money (Remember `msg.value` in the contract) we need to do like this:
```javascript
let weiValue = ethers.utils.parseEther('1')
result = await instaclone.connect(tipper).tipImageOwner(imageCount, { value: weiValue });
ev = await result.wait()
args = ev.events[0].args
...
 expect(args.tipAmount).to.equal(weiValue, 'Tip amount is correct')```

We can also test for failure (Remember `require` in the contract):
```javascript
await expect(
    instaclone.connect(tipper).tipImageOwner(9999, { value: weiValue })
).to.be.revertedWith("NOT EXISTING IMAGE");
```

Run `npx hardhat test` and expect similar results:

![test_results.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1617290622848/6zf_j1gDu.png)

## Deployment

Create a file `scripts/deploy.js` with the following content:
```javascript
async function main() {

    const [deployer] = await ethers.getSigners();

    console.log(
        "Deploying contracts with the account:",
        deployer.address
    );

    console.log("Account balance:", (await deployer.getBalance()).toString());

    const Instaclone = await ethers.getContractFactory("Instaclone");
    const clone = await Instaclone.deploy();

    console.log("Instaclone Contract address:", clone.address);
}

main()
    .then(() => process.exit(0))
    .catch(error => {
        console.error(error);
        process.exit(1);
    });
```

Open a new terminal and start Hardhat Network `npx hardhat node`. This gives you the available accounts.
![hardhat_network.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1617291181806/hjDXPJedA.png)

Import the first account's private key to Metamask. The Network Settings should be as below:

![HH_CHAIN_ID.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1617291494467/9wKRopaxp.png)

Open another terminal and deploy the contract:

``` 
npx hardhat run --network localhost scripts/deploy.js
```
The result logs the address of the owner of the contract (deployer's address), the owner's balance and the contract address. 

![deployed.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1617292078351/i4GUdbPDz.png)
Mark the contract's address because we'll need it later.

## Creating Frontend

The frontend consists two function-based React.js component, `App.js` and `Renderer.js`. 
In `App.js` there is a `useEffect` hook which starts the cycle. It runs only once when the component gets created. 
```javascript
 useEffect(() => {
    const load = async () => {
      await connectBlockChain();
    }
    load();
  }, []);```

UseEffect calls `connectBlockchain()` which, well connects to the smart contract. Note, this is the place where we need to specify the contract's address. (There might be a more elegant way to do it...)
```javascript
const connectBlockChain = async () => {
    ....
    await window.ethereum.enable();
    const provider = new ethers.providers.Web3Provider(window.ethereum);

    const signer = provider.getSigner();
    const contract_address = "0x5FbDB2315678afecb367f032d93F642f64180aa3" //local
    let deployedContract = new ethers.Contract(contract_address, Instaclone.abi, signer)
    ....
  }
```
As I mentioned earlier we upload the file to IPFS which returns a hash. We upload this hash to our smart contract. This is implemented as follows:
```javascript
 const uploadImage = async (description) => {
    //adding file to the IPFS
    ipfs.files.add(buffer, async (error, result) => {
       ...here comes logging
      //uploading hash to blockchain
      const tx = await contract.uploadImage(result[0].hash, description); 
      const receipt = await tx.wait();      
    })
  }
```
We send ether to the smart contract like this:
```javascript
const tipImageOwner = async (id, tipAmount) => {
    const tx = await contract.tipImageOwner(id, { value: tipAmount })
    const receipt = await tx.wait();  
}
```

Below is the full `App.js` file. 
```javascript
import React, { useEffect, useState } from 'react';
import Renderer from './Renderer.js';
import Instaclone from "./Instaclone.json"

const ipfsAPI = require('ipfs-api')
const ipfs = ipfsAPI('ipfs.infura.io', '5001', { protocol: 'https' })
const ethers = require('ethers');

function App() {

  const [contract, setContract] = useState();
  const [buffer, setBuffer] = useState();
  const [images, setImages] = useState([])
  const [loading, setLoading] = useState(false);

  const connectBlockChain = async () => {
    setLoading(true)
    await window.ethereum.enable();
    const provider = new ethers.providers.Web3Provider(window.ethereum);

    const signer = provider.getSigner();
    const contract_address = "0x5FbDB2315678afecb367f032d93F642f64180aa3" //local
    let deployedContract = new ethers.Contract(contract_address, Instaclone.abi, signer)
    setContract(deployedContract);

    await fetchImgages(deployedContract);
    setLoading(false)
  }

  // Load images
  const fetchImgages = async (contract) => {
    const imagesCount = await contract.imageCount()
    for (var i = 1; i <= imagesCount.toNumber(); i++) {
      const image = await contract.images(i);
      images.push(image);
    }
    images.sort((a, b) => b.tipAmount - a.tipAmount);
  }

  useEffect(() => {
    const load = async () => {
      await connectBlockChain();
    }
    load();
  }, []);

  const captureFile = event => {
    event.preventDefault()
    const file = event.target.files[0]
    const reader = new window.FileReader()
    reader.readAsArrayBuffer(file)
    reader.onloadend = () => {
      let buff = Buffer(reader.result)
      setBuffer(buff);
    }
  }

  const uploadImage = async (description) => {
    //adding file to the IPFS
    ipfs.files.add(buffer, async (error, result) => {
      console.log('Ipfs result = ', result)
      if (error) {
        console.error(error)
        return
      }
      //uploading hash to blockchain
      const tx = await contract.uploadImage(result[0].hash, description);
      console.log("tx = ", tx)
      const receipt = await tx.wait();
      console.log("receipt = ", receipt)
    })
  }

  const tipImageOwner = async (id, tipAmount) => {
    const tx = await contract.tipImageOwner(id, { value: tipAmount })
    console.log("tx = ", tx)
    const receipt = await tx.wait();
    console.log("receipt = ", receipt)
  }

  return (
    <div className="App">
      { loading
        ? <div id="loader" className="text-center mt-5"><p>Loading...</p></div>
        : <Renderer
          images={images}
          captureFile={captureFile}
          uploadImage={uploadImage}
          tipImageOwner={tipImageOwner}
        />
      }
    </div>
  );
}
export default App;
```
Rendering the page is the task of `Renderer.js`. It contains a form for file upload:
```
<form onSubmit={(event) => {
     event.preventDefault()
     const description = imageDescription.value
     uploadImage(description, captureFile)
}} >```
The form has access to the relevant functions (uploadImage, captureFile) in App.js.
It has access to the images, too and renders them iterating through them one by one with the help of IPFS/ Infura:
```javascript
{images.map((image, key) => {
....
<img src={`https://ipfs.infura.io/ipfs/${image.hash}`} 
```
The conversion between human-readable amounts like 0.01 and Ethereum-like values with many zeros takes also place here. Human-readable to BigNumber:
```javascript
let tipAmount = ethers.utils.parseEther('0.01')
 ```
and back:
```javascript
 ethers.utils.formatEther(image.tipAmount)
```
Below is the full code of `Renderer.js`
```
import React from 'react';
import Identicon from 'identicon.js';
const ethers = require('ethers');
const Renderer = ({ images, uploadImage, captureFile, tipImageOwner }) => {
    let imageDescription = "";
    return (
        <div className="container-fluid mt-5">
            <div className="row">
                <div className="content mr-auto ml-auto">

                    <h2>Share Image</h2>
                    <form onSubmit={(event) => {
                        event.preventDefault()
                        const description = imageDescription.value
                        uploadImage(description, captureFile)
                    }} >
                        <input type='file' accept=".jpg, .jpeg, .png, .bmp, .gif" onChange={captureFile} />
                        <div className="form-group mr-sm-2">
                            <br></br>
                            <input
                                id="imageDescription"
                                type="text"
                                ref={(input) => { imageDescription = input }}
                                className="form-control"
                                placeholder="Image description..."
                                required />
                        </div>
                        <button type="submit" className="btn btn-primary btn-block btn-lg">Upload!</button>
                    </form>
                    <p>&nbsp;</p>
                    {images.map((image, key) => {
                        return (
                            <div className="card mb-4" key={key} >
                                <div className="card-header">
                                    <img
                                        className='mr-2'
                                        width='30'
                                        height='30'
                                        src={`data:image/png;base64,${new Identicon(image.author, 30).toString()}`}
                                    />
                                    <small className="text-muted">{image.author}</small>
                                </div>
                                <ul id="imageList" className="list-group list-group-flush">
                                    <li className="list-group-item">
                                        <p className="text-center"><img src={`https://ipfs.infura.io/ipfs/${image.hash}`} style={{ maxWidth: '420px' }} /></p>
                                        <p>{image.description}</p>
                                    </li>

                                    <li key={key} className="list-group-item py-2">
                                        <small className="float-left mt-1 text-muted">
                                            TIPS: {ethers.utils.formatEther(image.tipAmount)} ETH
                                        </small>
                                        <button
                                            className="btn btn-link btn-sm float-right pt-0"
                                            name={image.id}
                                            onClick={(event) => {
                                                let tipAmount = ethers.utils.parseEther('0.01')
                                                tipImageOwner(event.target.name, tipAmount)
                                            }}
                                        >
                                            TIP 0.1 ETH
                                        </button>
                                    </li>
                                </ul>
                            </div>
                        )
                    })}
                </div>
            </div>
        </div >
    )
}
export default Renderer
```

That's it. Congratulation. You made it till the end.






  

























