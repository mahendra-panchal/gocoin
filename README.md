Gocoin
==============
Gocoin is a bitcoin client written in Go language (golang).

If you would like to express your interest in a further development of this project, send 0.01 BTC to 1WEyRRbAgPTpAPUgCSxLrJjXgiTU86WKt (please don't send  more - I  have enough, give it to orphans).


Limitations
--------------
At the current version it only works with entire blocks and does not rely transactions, except from announcing own transactions to the network.


Architecture
--------------
The entire software package consists of two applications:
* client - bitcoin node
* wallet � deterministic wallet

� and two libraries:
* btc � the bitcoin library, with a persistent storage database
* blockdb � a helper lib, used for importing blocks from the official client


Dependencies
==============
Allow Go to fetch the dependencies for you:

	go get github.com/piotrnar/qdb
	go get code.google.com/p/go.crypto/ripemd160


Requirements
==============

Client
--------------
Because of the required memory space, the client side (network node) is only supposed to be used with a 64-bit architecture (OS), though for Testnet only purposes, 32-bit arch should also be enough.

When the full bitcoin block chain is loaded, the node needs about 2GB of system memory, so have at least as much free RAM.

The entire block chain is stored in one large file, so your file system must support files larger than 4GB.


Wallet
--------------
The wallet application has very little requirements and it should work with literally any platform where you have a working Go compiler. That includes ARM (i.e. for Raspberry Pi).

Please keep in mind that the wallet uses unencrypted files to store the private keys, so make sure that these files are stored on an encrypted disk. Also use an encrypted swap file.


Building
==============
For anyone familiar with Go language, building should not be a problem. You can fetch the source code using i.e.:

	go get github.com/piotrnar/gocoin/btc

After you have the sources in your local disk, building them is usually as simple as executing "go build", in either the client or the wallet directory.


User Manual
==============
Both the applications (client and wallet) are console only (no GUI).

The client, after started, provides an additional text command based interface, that is referred later as UI.


Client
--------------
Command line switches for executing the client:

	-t - use Testnet (instead of regular bitcoin network)
	-l � also listen for incoming TCP connections (no UPnP support)
	-ul=NN - set maximum upload speed to NN kilobytes per second
	-dl=NN - set maximum download speed to NN kilobytes per second
	-r - rescan the block chain and re-create unspent database
	-c="1.2.3.4" - connect only to this host

When the client is already running you have the UI where you can issue commands. Type "help" to see  the possible commands..



Wallet
--------------
Wallet is deterministic and the only thing you need to setup it is the seed password, therefore as long as you remember the password, you do not need to backup the wallet ever.

You can either enter this password each time when running the wallet (not advised since characters are shown on the screen) - or you can store it, once and for all, in a file called wallet.sec

Make sure that the disk with wallet.sec file is encrypted. Also keep in mind that wallet.sec should not contain any special characters, nor new line characters, because they count while calculating the seed while you don't see them on screen, so you may have problems re-creating the same file after you would loose it.

Additionally, you can also import keys from your existing bitcoin wallet. To do this use �dumprivkey� RPC and store the base58 encoded value in a file named others.sec - each key must be in a separate line. You can also place a key's label in each line, after a space.


Spending money
--------------
After you setup your wallet in a secured environment, you should export its public addresses. In order to to this, just run:

	wallet -l

The wallet's public addresses will be written to wallet.txt. Now you can take this file safely to your client's PC and place it in the folder where it looks for wallet.txt (the path is printed when starting the client). Execute �wal� from the UI to reload the wallet.

From the moment when wallet.txt is properly loaded into your client, you can check your balance using the UI:

	bal

Each time you execute �bal�, a directory "balance/" is (re)created, in the folder where you run your client from.

To spend your money, move the most recent "balance/" folder to the PC with your wallet. If you execute �wallet� without any parameters, it should show you how many bitcoins you are able to spend, from the current balance. If it is more than zero, to spend the coins, order the wallet to make and sign a transaction using a command like:

	wallet -send 1JbdKe4eBwtexisGTbCKY5v5CfphtdZXJs=0.01

There are also additional switches which you may find useful at this stage. To see them all, try:

	wallet -h

If everything goes well with the "-send �" order, the wallet creates a text file with a signed transaction. The file is named like 01234567.txt

Now move this transaction file to your running client and use its UI to execute:

	tx 01234567.txt

The node should decode the transaction and display its details, for your verification. It will also output the transaction ID.

After making sure that the transaction does what you wanted, you can broadcast it to the network:

	stx  <transactionid>

Please note that the current version of client does not re-broadcast transactions, so if your transaction does not appear in the chain soon enough, you may want  to repeat the "stx ..." command. There may of course be other reasons why your transaction does not get confirmed (i.e. the fee was to small), in which case repeating "stx ..." will not help much.
