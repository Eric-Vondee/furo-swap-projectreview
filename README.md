## Streaming money on streiods via bentobox

## contract Furostream inherits interface Furostream, ERC712(NFT), Boring ownable and Boring batchable. 

1. interface Furostream has 8 functions, 3 events and a struct: 
##inserts image
2. ERC721 is a standard for representing ownership of non-fungible tokens, that is, where each token is unique.
## NFTs can represent ownership over digital or physical assets. We considered a diverse universe of assets, and we know you will dream up many more:
- Physical property — houses, unique artwork
- Virtual collectables — unique pictures of kittens, collectable cards
- “Negative value” assets — loans, burdens and other responsibilities
## where NFT name is "Furo Steam" and the symbol is "FURO" ERC721("Furo Stream", "FURO")
3. contract BoringOwnable inherits state variables owner/pendingOwner address from contract BoringOwnableData.
-   event OwnershipTransferred(): emits an event OwnershipTransferred of previousOwner and newOwner.
-   function transferOwnership(): transfers ownership to address newOwner, if direc 
-   function claimOwnership(): transfers ownership from owner to pendingOwner, it also verifies if msg.sender(user interacting with the contract) is pendingOwner before the transfer of ownership.
-   modifier onlyOwner(): modifier which verifies if msg.sender(user interacting with the contract) is the owner.

4. contract BoringBatchable inherits BaseBoringBatchable with functions:
 - function _getRevertMsg(bytes memory _returnData): function to help extract useful message from a failed call
 - function batch(): 
 - function permitToken(): uses the EIP permit-712-signed approvals for gaseless offchain structured hashing and signing. Takes in arguments(token, from&to address, amount, deadline, v, r, s) where v,r,s is a siganture message from the message signer. 

 ## FURO STREAM CONTRACT AUDIT

 ## State Variables
 - `wETH address`
 - `uint256 streamIds`
-   `mapping of keyType(uint256) to struct stream`
- `Custom errors`:  is a convenient, gas-efficient and clean way to explain to users why an operation failed through.

## constructor
the constructor takes in two arguments(interface _bentobox and wrapped ether(wETH) contract address) and writes the corresponding values to storage. 
## inserts image

1. function setBentoBoxApproval: overrides its function in interface IFurostream, takes in the arguments depicted in the image below and calls setMasterContractApproval contract and encodes the arguments passed.
## inserts image
2. ## function createStream():
 * creates an active stream by passing the required arguments in the picture below and returns a streamId with deposited shares.
 - it further validates if the starting time is less than the timestamp of the current block in seconds and reverts with the customError "InvalidStartTime()" if it fails.
 - depositedShares is calculated by calling internal function _depositToken() which takes in arguments and are depicted below
    ## inserts _depositToken image
    - `token`: 
    - `address from`:  msg.sender(user interfacing with the contract).
    - `address to`: contract's address.
    - `amount`: token amount to be deposited 
    - `bool fromBentoBox`: boolean value
- _depositToken() first verifies the if the condition that checks token address and balance of the contract is greater than the amount, if this conditions holds it continues the process of making a low level call `bentoBox.deposit` and if it fails run the else condition.
- increments the streamId in state, mints the streamd to recipient, initialize the struct and save to mapping streams. 
- emit LogCreateStream 

3. ## function withdrawFromStream()
    * withdraws from an active stream by rrequiring the streamId, sharesTowithdraw, receiving address, bool value, taskData and returns the receiientbalance and the receiving address
    ## inserts withdrawFromStream(0) image
    - Given the right streamId, it initially verifies the condition that msg.sender(address interfacing with the contract) is equal to the address stored in our mapping and our recipient 
    - Reads the properties of the stream from state, checks if the recipients balance is sufficent to facilitate the withdrawal and if otherwsie throws our error InValidWithdrawTooMuch();
        - ## inserts _transferToken()
        - `token`: address of the streamed token
        - `address from`: contract address 
        - `address to`: the withdraw address passed as an argument or recipients address which is gotten from the streamId.
        - `amount`: amount of tokens to be withdrawn
        - `bool toBentoBox`: boolean value
    - updates the recipient balance in state to guard against reentrancy before issuing a shares transfer to a recipient address.
    - emits the event LogWithdrawFromStream()

4. ## function cancelStream()
    - This function cancels a current stream and withdraws the shares to recipients address 
    - emits LogCancelStream()

5. ## function getStream()
    - Returns the properties of stream given the streamId
    ## inserts getsream image 

6. ## function streamBalanceOf()
  - Takes an argument of streamId, reads the properties of stream from storage and returns the sender's & recipient balance.
    ## inserts stream baalnceOf

7. ## function updateSender()
    * This functions updates the sender's address in storage only if  msg.sender(address interfacing/interacting with the contract) is equal to the one in storage.
    ## inserts updateSender() image

8. ## function updateStream()
    ## add update stream imge
    * this function updates the streambalance 
    - gets the recipients current balance, transfers to the recipient and updates the withdrawnShares value in state (Could cause reentrancy)
    - then calls the _depositToken() function which transfers token from thereceipient before updating state. 
    - then finally updates the deposited shares and endTime


