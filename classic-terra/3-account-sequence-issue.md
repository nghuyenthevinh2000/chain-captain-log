# Terra Classic dyncomm account sequence incident

A very fascinating case about an exploit during my maintenance of Terra Classic (LUNC). In the beginning, there was a movement to implement dynamic commission on LUNC to make it more fair and combat validator centralization. The higher your voting power is, the higher the commission. In case of AllNodes, they have 16% voting power thus their dynamic commission is 20%. I didn't participate in building that but I had to review it since I managed the Terra Classic repository. So, that movement was gov - approved and then implemented on chain. However, in the next few days, validators started reporting issues about "account sequence mismatch". Some even had huge mismatch. This is a critical problem actually because it prevents user from submitting all kinds of transactions, so basically they are blocked from interacting with the chain. This small issue if not handled soon can lead to chain halt in the future and here is why.

We first have to understand that account sequence is unique to each on - chain account and ensures linearizability. So, it will increase by 1 for each new tx and block all other future txs until the current one is handled. 

About the code for dynamic commission decorator, it is supposed to prevent "EditValidator request" with commission smaller than minimum threshold. There is a bug in its decorator that skips CheckTx stage. The decorator also sits after SigVerification and IncrementSequence decorator allowing the illegal increases of account sequence in CheckTx's account keeper leading to account verification failure. To understand this more clearly, we need to look at some definitions first. You can replicate the situation by trying the test I made at here:

https://github.com/classic-terra/core/blob/main/x/dyncomm/ante/ante_test.go#L210

## Some questions to help you explore:
1. What is checkTx and what is its role in this issue?
2. Why account sequence mismatch happen?
3. Why failed txs are included in main net blocks?

Let's first talk about CheckTx. There are two stages of transaction processing in a Cosmos chain which are CheckTx and DeliverTx. Pay close attention to CheckTx.
1. CheckTx: first stage of msgs processing at local node. Tendermint will request Cosmos Application to check messages at local node before broadcasting those messages to other nodes. This request is called as CheckTx. If failed, msgs will return err and not be broadcasted. If success, msgs will be broadcasted for round - robin. Upon received in Cosmos Application, CheckTx will run through ante handler only, no msgs will be processed. CheckTx has its own context storage and will remain alive for the whole blockchain lifetime.
2. DeliverTx: second stage of msgs processing. DeliverTx request from Tendermint will actually have its msgs handled in CosmosApp. The result will then be committed and  applied to block. DeliverTx has its own context storage and will last from BeginBlock till Commit before being reset.

Now here is what has happened, Tendermint sends a CheckTx request to CosmosApp for verification of some EditValidator messages. When it reaches Dyncomm decorator, it is skipped without checking invalid commission. Consequently, it leads to the illegal increase of account sequence in CheckTx account keeper. I will explain more later why illegal increase of account sequence will lead to account mismatch. Those dirty EditValidator messages are considered as "good message" by Tendermint. These "good messages" will then be broadcasted to all other nodes for round - robin. Another danger is that these "good messages" take up spaces in a block and push real good messages away. 

Let's create a mock simulation on a distributed environment to understand more. Provided that we have node A, B, C constituting a network. We have
1. CLI A, B, C:
* builds a transaction with context accSeq then broadcast to both local node and other nodes
2. Node A, B, C:
* handles transaction submitted by others
* stores result in CheckTx account keeper accSeq

Assume that we are on local node A, with CLI A to interact. Account sequence mismatch happens when accSeq of a message is different from (stored accSeq + 1). This indicates that there is a linearizability violation here, so ante handler will reject the transaction with mismatched accSeq.

An EditValidator tx is first constructed on CLI A with accSeq 0, then broadcasted to local node A. In ante handler of CheckTx phase, it first passes IncrementSequenceDecorator incrementing CheckTx account keeper accSeq. A: 0 -> 1. Now, A: 1 is persisted into local CheckTx account keeper store on A. Tendermint perceives this as a "good message", EditValidator tx is then broadcasted to B, and C. When B and C receives this "good message", they also run EditValidator tx again in CheckTx state which increase CheckTx account keeper to B: 1, and C: 1

Now, we have in storage of A, B, C the value of accSeq like so:
1. A: 1 (CheckTx)
2. B: 1 (CheckTx)
3. C: 1 (CheckTx)
4. A: 0 (DeliverTx)
5. B: 0 (DeliverTx)
6. C: 0 (DeliverTx)

Moving on DeliverTx stage, on node A, B, C, EditValidator tx first goes through SigVerification and passes because CheckTx accSeq = DeliverTx accSeq + 1 so acceptable. It then goes through Dyncomm decorator and fails because commission is smaller than required minimum. DeliverTx context storage is reverted.

Now, we have in storage of A, B, C the value of accSeq like so:
1. A: 1 (CheckTx)
2. B: 1 (CheckTx)
3. C: 1 (CheckTx)
4. A: 0 (DeliverTx)
5. B: 0 (DeliverTx)
6. C: 0 (DeliverTx)

Another EditValidator tx is broadcasted going through two stage CheckTx and DeliverTx same as before. Only that this time in DeliverTx stage, it returns account sequence mismatch instead. Here is the new state of accSeq
1. A: 2 (CheckTx)
2. B: 2 (CheckTx)
3. C: 2 (CheckTx)
4. A: 0 (DeliverTx)
5. B: 0 (DeliverTx)
6. C: 0 (DeliverTx)

Continuing like so, Ante Handler will keep returning account sequence mismatch in DeliverTx stage. Failed messages are included in block, taking up spaces of other valid messages.

The solution to this is like so:
1. First fix the ordering of Dyncomm Decorator to be before SigVerification and IncrementSequence decorator
2. Remove CheckTx skip in Dyncomm Decorator
3. Fix CheckTx store to allow affected addresses to send transactions again