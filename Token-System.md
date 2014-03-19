## Token System on SAFE Network  ##
Version 1.2

Last Updated 19 March 2014

#### 1.	Introduction
The SAFE network [ref Network] utilises a mathematically complete, peer-to-peer Public Key Infrastructure (PKI) authorisation on an autonomous network [ref Autonomous], secured key-value storage and reliable Kademlia based routing [ref Routing]. The network is designed to be decentralised and has the ability to get rid of Domain Name System (DNS). The PKI solution deployed within the SAFE network validates a user’s identity with mathematical certainty.

Bitcoin [ref BitCoin] has proved the ability of crypto currencies to disrupt the status quo.  Bitcoin proposed and executed a very innovative idea, coupled with a well-considered system design based on the block-chain and proof-of-work concepts. In essence, Bitcoin is a partially-decentralised (due to the use of the block chain) digital currency on a centralised network. MaidSafe propose a token based economic system on the SAFE network. In effect, a decentralised digital currency system on decentralised network. 
 
#### 2.	Trusted Group
With the SAFE network, it can be assumed that the majority of a close (XOR distance as defined by Kademlia) group of nodes is trustable. While not impossible to generate a majority of malicious nodes in a close group around a particular target, it should be considered computationally unfeasible.

On the SAFE network, the following rules ensure a trusted group:

* It is hard to have a vault with a particular address (the address of a new vault will be defined by the network using the hash of the vault’s credentials). In addition, each time a vault is switched off and then rejoins the network, it will be assigned a new address. Further more, the node will not be considered as full functional vault till complete the verification period.
* When a request is emitted from a group, all group member’s signatures will be attached. On the receiver side, not only a routing level find closest verification will be carried out to verify the senders are really closest nodes to the target, but also the public keys for the nodes will be downloaded from the network to validate signatures.
* The exchange of a vault between different users is not allowed. A vault is allowed to be not linked to an account (unowned) and then linked to an account later (becomes owned). However once linked, that vault cannot be detached.

Furthermore, there is the RUDP [ref RUDP] layer which encrypts communications between nodes to prevent the message content from being secretly modified by a third party. This ensures the request reflects the real intention of the sender.

Once the majority of the nodes inside the group have sent out a consistent request, it shall be considered valid.

Once the majority of the nodes have decided to perform the same action, the action shall be considered valid.

The Trusted Group feature delivered by the SAFE network ensures the system is secure as long as the majority of the nodes are honest and it is computationally / economically expensive to form a malicious node.

#### 3.	Transaction Managers / Structured Data Version
On the SAFE network, vaults assume various personas or roles [ref Persona], depending on the requests they receive.  For example, the DataManager persona is responsible for managing the integrity and availability of a given piece of data on the network.  A separate persona, TransactionManager, is proposed to handle all the token-related transactions.  A TransactionManager group will be a trusted group of nodes which are closest to any given transaction identity. The TransactionManager is responsible for the business logical to complete a transaction.

StructuredDataVersions (SDV) is a data structure which allows the creation and handling of a version tree.  The entire version tree is addressed by a randomly-chosen ID. This data type is managed by the VersionHandler Vault persona.  A VersionHandler group is the trusted group of nodes closest to the SDV ID.  Each entry (version) in the tree will have the format <version_number, immutable data / data_name>. One of the use of this data structure is to store versions of directories on a Virtual Filesystem offered by Maidsafe-Drive.

Based on the token types, Transaction Manager persona will be creating variation of SDV types. SDV related to tokens and transactions can only be created and updated by the TransactionManager group around it. Random id associated with the version tree can be directly used as the token name or can be related to the transaction id. With different payload inside SDV, it can be used for various purposes; as a transaction, as a token itself, as a wallet.

As transaction Manager for a token are group of Vaults close to the token name, the same group will also act as Version handler for the SDV tree which Transaction Manager will be handling (or can be separated by introducing an additional layer of Hash(token name)).

Transaction Manager’s functionality can be further detailed by taking an example of a Token creation and Token exchange mechanism. If a new token is mined, a SDV tree with name same as token identity, will created by Transaction Manager. 

The first version (root of tree) will be <0 (id with all bits zero), Hash(name of token holder)> (using Hash of holder name to prevent leaking user info). As only the TransactionManager can update the versions, they "own" SDV in the network sense, but the current tip-of-tree inside represents the token holder (creditor). Transaction Manager only authorise the owner of the token to do a transaction.

So, say for token "nnn" the current token holder is "abc" and he wants to give the token to "xyz". abc sends a valid request : "TokenTransferRequest abc to xyz for token nnn" to the TransactionManagers at "nnn". T.M. get tip-of-tree for SDV "nnn" and check it says something like <999, hash(abc)>.  If hash(abc) is not the last owner ot the token, it doesn't have the right to transfer the token and the request fails. If hash(abc) is listed as the last version of the tree, transaction will progress and the new version will be created by T.M. and will be stored on VersionHandler as <1000, hash(xyz)> and xyz is now the token holder(owner). Anyone can get these versions to validate the transaction including abc and xyz themselves.

In this example, the version names represent actual MAID names (or MPID names or whatever). We could of course do the more complicated thing and have abc and xyz represent ImmutableData chunks which the TransactionManagers can create/encrypt/store/get. By doing this, the ImmutableData part becomes the payload part, and SDV can be used as transactions or wallets or anything else.

#### 4.	Transfer Mechanism
Transfer mechanism is defined as: ‘allowing a transaction (transfer an amount from A's wallet to B's wallet) between two user's persona groups to be completed’. The transaction shall be open and read only to public (allowing upper layer third party broker app to validate there is transaction happening/completed).

The SAFE wallet is defined as : the place holding the credits (and the credit change history) for an account.

The procedure of a transfer (user_A transfers credit to user_B) can be illustrated as :

1.	User A make a function call : user_A.Transfer(user_B, amount, wallet)
2.	When the MaidManager group of user A receives a request, they :
    
    i)    debit the amount from user A's wallet
    
    ii)    send a request to the TransactionManager
    
    iii)    send a notification to the upper layer APP
3.	When the TransactionManager group receives a notification, they :

    i)    send a notification to user B's persona

    ii)    create an internally transaction
4.	When user B's MaidManager group receives a valid notification, they :

    i)    send an acknowledgement to the TransactionManager group

    ii)    credit user B's wallet with the amount

![Third Party Transaction Validator](https://raw.github.com/maidsafe/Whitepapers/master/resources/third_party_transaction_validator.png)

#### 5.	Proof Of Resource
On the SAFE network, a user contributes to network by running a vault, which will handle requests and store data for others. The following parameters are used to measure a vault and a user account :

* stored_space : total size of chunks that have been stored to that vault by network

* lost_data : As data is stored on a node it may switch off or be otherwise unavailable, we consider that data lost after the network decides the node has lost it. This is a critically important measure and in no way means the network has lost data. This is a common practice for a node on the network.

* healthy_space (h.s.) :  h.s. = stored_spacecur - lost_dataprev    ------   ①

* available_space : the storage space a vault claimed can contribute to network

* data_cost : The same user storing the same chunk will only be charged once (at the rate of 4x chunk size for the first time, to cover the initial cost for the network to keep 4 duplicated copies).  Other users will also be charged for putting the chunk, but at the rate of 1x chunk size.  The charge will be capped at 50 users.  (The subsequent users paying for the chunk covers the case where the initial storer disappears from the network)

* used_space : total data_cost of all chunks that user put to network

The Proof Of Resource (P.O.R) is derived from healthy_space (which is a kind of QoS measurement)

* P.O.R will be updated when healthy_space updated in bi-direction

P.O.R =  healthy_space       ------   ②

This ensures P.O.R becomes huge negative when user transfer out P.O.R and then switch off the vault

* P.O.R will be checked when a user tries to PUT data.
(used_space + data_cost ) < P.O.R      ------   ③

A user’s Initial allowance will be granted by setting used_space to negative claimed available_space. If any cheating is detected, the used_space will be changed to reflect that.

This will also cover for situations when a user’s P.O.R drops low by allowing the user to mutate his free allowance data.

* P.O.R is transferable among users

* P.O.R shall be an int in a unit directly derived from size unit KB (MB ...)

* MaidAccount will be the wallet to the P.O.R, and MaidManager group will be responsible for 
the updating.

* P.O.R shall be considered as a standard unit being recognized by all nodes across the SAFE  network.

| User A |   | | User B |  |  | 
| ---------------|:-----------------:|:-----------------:|:-----------------:|:-----------------:|:-----------------:|
| Action | (used_space, stored_space, P.O.R)  | Allowance | Action | (used_space, stored_space, P.O.R)  | Allowance | 
|Create Account	| (0, 0, 0) | 0 |Create Account	| (0, 0, 0) | 0 |
| Picked by Network To store 50 data	| (0, 50, 50) | 50 | Never picked up or don’t have a Vault | (0, 0, 0) | 0 |
|Store 20 data to network	| (20, 50, 50) | 30 | Nothing Done | (0, 0, 0) | 0 |
|Sell 20 P.O.R to User B	| (20, 50, 30) | 10 | Buy 20 P.O.R from User A | (0, 0, 20) | 20 |
|stored_space decreased by 10. Q.O.S drop or stored chunk removed by network | (20, 40, 20) | 0 | Store 10 data and get picked to store 10 | (10, 10, 30) | 20 |

#### 6.	Economic system with two types of token
P.O.R is proposed in order to facilitate the exchange of storage space on the SAFE network. However, as it doesn't have a predictable cap number, it may not be able to be considered as a genuine token based virtual currency. To provide a more robust currency, MaidSafe proposes to have a token system that is totally independent of P.O.R, called safecoin. Safecoin will have a predicatable cap and will be injected into network using the storage space related mining procedure.

A bridge (converting rate) between P.O.R and safecoin can be established by the market solely. With third party upper layer broker applications, it will be possible to use safecoin to buy P.O.R or vice versa (user_A give safecoin to user_B in exchange for user_B's P.O.R). We expect the per unit value of P.O.R keeps decreasing, meanwhile the per unit value of safecoin keeps rising. i.e. one safecoin could buy more and more P.O.R. Safecoin is also be kept in maid account wallet, which can only be  updated by the MaidManager group.

The value of one safecoin represents will be recognized by all peers across the network and outside the network. If the economic system works as intended, safecoin will become a ‘virtual currency’ with the SAFE network being used to complete all transactions. Meanwhile P.O.R will solely be used for exchanging space allowance among users.

A projection of P.O.R. is estimated as :

| Network Wide Total Data | Copies | Traffic | De-Duplication Factor | P.O.R | Nodes | Data Put per Node | P.O.R per Node |
| ---------------|:-----------------:|:-----------------:|:-----------------:|:-----------------:|:-----------------:|:-----------------:|:-----------------:|
| 1000 | 4 | 1 | 1 | 5000 | 100 | 10 | 50|
| 3800 | 4 | 0.9 | 0.9 | 17100 | 200 | 19 | 85.5 |
| 10800 | 4 | 0.8 | 0.8 | 43200 | 400 | 27 | 108 |
| 27200 | 4 | 0.7 | 0.7 | 95200 | 800 | 34 | 119 |
| 64000 | 4 | 0.6 | 0.6 | 192000 | 1600 | 40 | 120 |
| 144000 | 4 | 0.5 | 0.5 | 360000 | 3200 | 45 | 112.5 |
| 313600 | 4 | 0.4 | 0.4 | 627200 | 6400 | 49 | 98 |
| 665600 | 4 | 0.3 | 0.3 | 998400 | 12800 | 52 | 78 |
| 1382400 | 4 | 0.2 | 0.2 | 1382400 | 25600 | 54 | 54 |


To prevent a potential shortfall of P.O.R, it is proposed that safecoin can be converted into P.O.R directly (network level). However, to maintain the cap number of safecoin predictable, the network will not allow P.O.R to be converted back into safecoin.

The conversion rate (safecoin => P.O.R ) can be established as :

conversion rate (P.O.R / Token) = PT / NT  ≈ (PT<sub>local</sub> * (AS / AD<sub>local</sub> ) ) / NT    ------   ④

where : PT is the current total of P.O.R across the network

NT is the cap number of token

PT<sub>local</sub> is the current average P.O.R across the local group

AS is the total address space

AD<sub>local</sub> is the average address distance among the local group                      

given : both data and node_id are evenly distributed

Once a safecoin has been converted into P.O.R, that token will be released, i.e. the token will be marked as not occupied, allowing other user to re-mine it.

#### 7.	Safecoin General
Safecoin is capped at 232 (4.3 billion). Unlike P.O.R which is just an integer number held in maid_account, each safecoin is represented by an SDV, holding a list of owner history. The data structure of such an SDV can be illustrated as : <diagram safecoin SDV structure>

| Field | Length | Format |
| ---------------|:-----------------:|:-----------------:|
|name | 64 Bytes | See below |
| metadata_field |  | |
| version n-1 | 72 Bytes | [version_num, hash(prev_owner)] |
|version n | 72 Bytes | [version_num, hash(cur_owner)] |

The name of an safecoin is 64 bytes long to allow it to be a network-addressable object.  However, the name has a particular format:

[ 32 bits: Token ID    |   224 bits: ID padding   |   x bits (x <= 248): Subdivision bits   |   248 - x: Random   |   8 bits: Value of x ]

The initial part (Token ID) inherently limits the total number of tokens available to 232 since each token must have a unique ID.

The second part (ID padding) must be predictable (e.g. it could just be all ‘0’s, or it could be the ID concatenated 7 times).  Its purpose is to force all subdivisions of a given coin token the same trusted group of vaults to eliminate the need for network traffic when handling such subdivisions.

The third part defines the subdivision name.  For example, if x == 1 (regardless of whether the value of that bit is 0 or 1) then that token represents a half of the original token.

The fourth part is random padding.

The fifth part indicates the level of subdivision of the original token, i.e. it contains the value of x.

This format allows us to split the original token into 2248 parts if required.  The splitting process will only allow the token (or subdivided token) to be bisected, so e.g. quartering a token would need to be done in 2 steps.  When splitting a token, only the name changes; all other parts are copied to the new subdivisions.  The split results in 2 SDVs, each representing a half of the original SDV. This procedure is further illustrated in the following diagram.

![Diagram Split SDV](https://raw.github.com/maidsafe/Whitepapers/master/resources/split_sdv_diagram.png)


#### 8.	Mining safecoin
Every mining interval, the PmidManager (P.M) group around a vault will perform the mining for that vault. P.M will generate a Random Attempt Target (R.A.T) based on the following calculation :

R.A.T = Hash( (merkle_tree_root + msg_id) XOR R.A.T prev )    ------   ⑤

where : merkle_tree_root is generated from all the chunks stored on that vault

msg_id is the agreed random id among the P.M group.

The R.A.T will then be sent to TransactionManager (T.M) as a SDV creation request, claiming the ownership of that SDV on behalf of that vault.

T.M will try to create an SDV with that name, and the last owner shall be the hash(maid_name).

If that SDV already existed, T.M. will then try to update all that SDV's sub-dividents which are not taken yet.

The MaidManager group of the maid_name will be notified when T.M. created or updated SDV successfully, otherwise the request is muted.

The mining interval allowed for a vault is determined by its contribution to the network. The interval is calculated as :
  
when healthy_space is greater than 4GB : mining_interval = 25 - min(24, healthy_space / 4GB) (hour)      ------   ⑥

i.e. the quickest mining speed is one attempt per hour, and the slowest is one attempt per day (24 hours)

Continuous connection time of vault is used, i.e. when vault switched off / disconnected, the interval timer will reset and stopped, generating no coins for the offline vault.

Following is the estimation of MaidSafe network size, the average mining attempts and the total attempts along time : (table network projection)

| Number of Nodes | Attempts Per day Per node | Total Attempts along time (years) |  |  |  |  | 
| ---------------|:-----------------:|:-----------------:|:-----------------:|:-----------------:|:-----------------:|:-----------------:|
| |  | 1 | 2 | 5 | 10 | 20 |
| 10000 | 6 | 22 million |  |  |  |  |　
| 50000 | 12 |  | 438 million |  |  |  | 
| 100000 | 15 |  |  |　2.7 billion |  |  |　
| 200000 | 20 |  |  |  | 14.6 billion |  |
| 500000 | 24 |  |  |  |  | 87.6 billion |

Given the collision probability as : (table collision probability) where N is the total space

| Collision Probability | Num of Attempts |
| ---------------|:-----------------:|
| 10% | 0.105 N |
| 20% | 0.22 N |
| 30% | 0.36 N |
| 40% | 0.51 N |
| 50% | 0.69 N |
| 60% | 0.92 N |
| 70% | 1.2 N |
| 80% | 1.6 N |
| 90% | 2.3 N |
| 95% | 3 N |

As the safecoin is capped at 4.3 billion, it is expected that 50% collision rate will be reached after 5 years. 95% collision rate will be reached after 10 years, as illustrated in the following diagram. <diagram Coin Distribution Projection>

![Diagram Projected Coin Distribution](https://raw.github.com/maidsafe/Whitepapers/master/resources/projected_coin_distribution_over_time.png)

As mentioned in Section 6, when a safecoin is converted into P.O.R, the SDV represents that token shall have its last owner to be updated to non-taken (set to be hash(kZeroId) ). With this recycling mechanism, there will always be non-mined coins available on market. 

#### 9.	Day 1 Injection
To reward investors and developers involved during early stage, it is suggested that certain amount of safecoin is reserved for them. At Day 1 of the network startup, say 10% of safecoin will be injected into network, with the owner to be the investor pool and the developer pool (each has 5% share of total safecoin). As MaidSafe as a company needs to provide the seed network, which makes ourselves a big miner as well, share holders will get rewarded via company mining.

This 10% safecoin will require a storage space of 1TB, given the average SDV size is estimated to be 0.5kB. Vaults holding these SDV will gain P.O.R, which means the same amount of P.O.R also got injected into network. This ensures there is certain amount of P.O.R available across the network for those client only users to startup with, via friends given as gift or purchase from others. As pointed out in the (table POR projection), sufficient POR will be generated via user behaviour during the early stage, it is expected this amount of initial injection will be enough to kick start providing storage service to public. 

#### 10.	Summary
To conclude, MaidSafe proposed the SAFE network, an economic system that contains two types of token and relies on a trusted group. The transfer mechanism is advantageous in many respects and has the functionality to prevent double-spending, while enabling the verification of transactions immediately. Transaction Managers handling SDV data type are included into the SAFE network to manage tokens and transactions. Proof of Resource (P.O.R) is introduced to smooth the exchange of storage space, while safecoin is introduced to incentivise stakeholders throughout the network. The total cap of safecoin is set to be 4.3 billion. With the proposed mining procedure (and assumptions) it is estimated that half the total volume will issued during the first 5 years, with 95% issued after 10 years. End users will mine coins based on their ability to provide computing resources to the network and, in addition to the other benefits offered by SAFE, this will be their main incentive for contributing storage space. When one safecoin is being converted into P.O.R to gain storage space allowance for the user, that safecoin token is recycled in order that other users can claim it via mining. Such re-mining procedure ensures there is always available empty tokens for mining. The tech stack of the token system is illustrated as <diagram Tech Stack>.

![Diagram Tech Stack](https://raw.github.com/maidsafe/Whitepapers/master/resources/tech_stack.png)

#### References 	 	 	

[ref Network] MaidSafe Network website : www.maidsafe.net

[ref Autonomous] Autonomous Network, David Irvine, Fraser Hutchison, Steve Mucklisch : https://github.com/maidsafe/MaidSafe/wiki/unpublished_papers/AutonomousNetwork.pdf?raw=true

[ref Routing] MaidSafe Routing github site : https://github.com/maidsafe/MaidSafe-Routing/wiki

[ref BitCoin] Bitcoin : A Peer-to-Peer Electronic Cash System, Satoshi Nakamoto, https://bitcoin.org/bitcoin.pdf‎

[ref RUDP] MaidSafe RUDP github site : https://github.com/maidsafe/MaidSafe-RUDP/wiki

[ref Persona] Vault Documentation : https://github.com/maidsafe/MaidSafe-Vault/wiki/Documentation

[ref Escrow] The Escrow service for Bitcoin : http://btcrow.com/

[ref BIP 16/17] BIP 16/17 in layman's term : https://bitcointalk.org/index.php?topic=61125.0

[ref MasterCoin] Master Coin : http://www.mastercoin.org/

