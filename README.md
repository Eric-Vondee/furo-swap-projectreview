# Streaming money on streiods via bentobox(Furo swap)

## Contract Furostream inherits interface Furostream, ERC712(NFT), Boring ownable and Boring batchable. 

### Interface Furostream has 8 functions, 3 events and a struct: 
 ![ifurostream](https://user-images.githubusercontent.com/23663250/162505571-f51a13ce-ad27-49c3-a440-d8adaa1c7648.png)

### ERC721 is a standard for representing ownership of non-fungible tokens, where each token is unique. NFTs can represent ownership over digital or  physical assets.  The contract's NFT name is "Furo Steam" and the symbol "FURO" 
    `ERC721("Furo Stream", "FURO")`

### Contract BoringOwnable inherits state variables `owner/pendingOwner address` from contract BoringOwnableData.
   - `event OwnershipTransferred()`: emits an event OwnershipTransferred of previousOwner and newOwner.
   - `function transferOwnership()`: transfers ownership to address newOwner, if direc 
   - `function claimOwnership()`: transfers ownership from owner to pendingOwner, it also verifies if msg.sender(user interacting with the contract) is    pendingOwner before the transfer of ownership.
   - `modifier onlyOwner()`: modifier which verifies if msg.sender(user interacting with the contract) is the owner.

### Contract BoringBatchable inherits BaseBoringBatchable with functions:
  - `function _getRevertMsg(bytes memory _returnData)`: function to help extract useful message from a failed call
  - `function batch()`: 
  - `function permitToken()`: uses the EIP permit-712-signed approvals for gaseless offchain structured hashing and signing. Takes in arguments(token, from&to address, amount, deadline, v, r, s) where v,r,s is a siganture message from the message signer. 

                                                      ## FURO STREAM CONTRACT REVIEW


 ## State Variables
  - `wETH address`: wrapped ether contract address
  - `uint256 streamIds`: an unsigned integer to handle unique stream Identification 
  - `mapping of keyType(uint256) to struct stream`: mapping(uint256 => Stream) public streams
  - `Custom errors`:  it's a convenient, gas-efficient and clean way to explain to users why an operation failed through.

## constructor
The constructor takes in two arguments `interface _bentobox and wrapped ether(wETH) contract address` and writes the corresponding values to storage upon contract deployment.

![constructor](https://user-images.githubusercontent.com/23663250/162510496-e32d14b3-8f00-4542-b155-7edb4013c876.png)


## function setBentoBoxApproval()
function setBentoBoxApproval overrides its inheriting function in `interface IFurostream`, setMasterContractApproval contract and encodes the arguments passed.

![senBentoBox](https://user-images.githubusercontent.com/23663250/162517315-a0da151e-fe9b-44d5-bbb6-d304b3da6877.png)

## function createStream()
 -  Creates an active stream by passing the required arguments in the picture below and returns a streamId & deposited shares.
 -  It further validates if the starting time is less than the timestamp of the current block in seconds and reverts with the customError " InvalidStartTime()" if it fails.
 - DepositedShares is calculated by calling internal function _depositToken() with arguments indicated in the image below:
  
 ![_depositToken](https://user-images.githubusercontent.com/23663250/162510693-a32be784-3061-4d7b-bfd6-da83663e271b.png)
    
    - `token`: 
    - `address from`:  msg.sender(user interfacing with the contract).
    - `address to`: contract's address.
    - `amount`: token amount to be deposited 
    - `bool fromBentoBox`: boolean value
- _depositToken() is an internal function that firstly verifies if the `token address and balance of the contract` are true.
    - If true, it continues the process of making a low level call `bentoBox.deposit` to deposit value in ether and corresponding actions.
    - If false, makes a low level call either to `bentoBox.share` or `bentoBox.deposit`
- It further increments the streamId in storage, mints the streamd to recipient, initializes the `struct stream` and write to `mapping streams`. 
- Lastly emits `emit LogCreateStream` 

## function withdrawFromStream()
withdraws from an active stream by requiring the streamId, sharesTowithdraw, receiving address, bool value, taskData and returns the recipientbalance and the receiving address.
 
 ![withdrawfromstream](https://user-images.githubusercontent.com/23663250/162510806-1d77adb8-675e-40f2-b5c4-ee5b90d050d7.png)
 
    - Given the right streamId, it initially verifies the condition that `msg.sender(address interfacing with the contract)` is equal to the storage address of our recipient. 
    - Reads the properties of the stream from state, checks if the recipients balance is sufficent to facilitate the withdrawal and if otherwsie throws our error InValidWithdrawTooMuch();
        - ## inserts _transferToken()

        - `token`: address of the streamed token
        - `address from`: contract address 
        - `address to`: the withdraw address passed as an argument or recipients address which is gotten from the streamId.
        - `amount`: amount of tokens to be withdrawn
        - `bool toBentoBox`: boolean value
    - updates the recipient balance in state to guard against reentrancy before issuing a shares transfer to a recipient address.
    - emits the event LogWithdrawFromStream()

## function cancelStream()
    - This function cancels a current stream and withdraws the shares to recipients address 
    - emits LogCancelStream()

## function getStream()
    - Returns the properties of stream given the streamId
   ![getStream](https://user-images.githubusercontent.com/23663250/162510961-0cceee75-f37c-4e0d-9449-1088010eb76d.png)

## function streamBalanceOf()
  - Takes an argument of streamId, reads the properties of stream from storage and returns the sender's & recipient balance.
![streamBalanceOf](https://user-images.githubusercontent.com/23663250/162511002-3d290197-e1b6-4d08-919b-c0e670ca2d7e.png)

## function updateSender()
    - This function updates the sender's address in storage only if  `msg.sender(address interfacing/interacting with the contract)` is equal to address in storage.
![updateSender](https://user-images.githubusercontent.com/23663250/162511076-5fc3e8f8-9b39-4d1c-91b3-9f53330f9ae4.png)

## function updateStream()
    ![updateStream](https://user-images.githubusercontent.com/23663250/162511118-52f73de5-016d-41e8-9b71-7e79566f572e.png)

    - this function updates the streambalance 
    - gets the recipients current balance, transfers to the recipient and updates the withdrawnShares value in state (Could cause reentrancy)
    - then calls the _depositToken() function which transfers token from thereceipient before updating state. 
    - then finally updates the deposited shares and endTime


