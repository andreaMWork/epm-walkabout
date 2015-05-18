So its been a while since I wrote any tutorials so I figured its about time. This is going to be a guide to understanding the machinery that is epm. I have been accused of being overly wordy or not wordy enough at times and this will be no exception I will be both of those things. But I figure its about time that someone wrote down what exactly this "EPM" thing is and why its such a fun (and useful) tool to use once you understand it a little bit. This is going to be an interactive tutorial so its expected that you will follow along and actually run the commands yourself and check what changes. One final caveat is that epm is constantly evolving. Whenever I am working with smart contracts and think of some tool that would make it simpler I ping Ethan and he makes it happen. As such this tutorial will be at least partially invalid almost immediately.

So lets start. First. To ensure we are all on the same page lets get EPM installed. The contracts we will be working with are included in this repo.

This assumes you have Go set up properly. I will not cover how to do that here but you NEED TO HAVE Go 1.4 OR HIGHER!

## Using Docker

You can use [Docker](http://docker.com/) to provide a Go environment:

```
$ docker run --name epm1 -it --expose 15256-15258 -v $PWD:/mnt golang:latest
# apt-get update 
# apt-get install -y libgmp3-dev vim
```

# EPM (Develop) setup

1.) Type in a terminal "go get github.com/eris-ltd/epm-go"

2.) cd $GOPATH/src/github.com/eris-ltd/epm-go

3.) git checkout develop (we will be using the develop branch to utilize some brand new features)

4.) git pull

5.) make

If you have a ~/.decerver folder, delete it. Alternatively, export a new path for the DECERVER environment variable. We want a clean slate for this.

6.) Docker step:

```
# cd /mnt
# exit
$ docker commit epm1 epm
$ docker start -i epm1
```

# 1) Init

Epm works out of the same ~/.decerver folder as decerver does. This is an intentional choice to standardize the working environment. To set up the directory structure simply type `epm init`

Lets checkout what has happened: navigate to ~/.decerver

```
dennis@Juzo:~/.decerver$ ls
blockchains  dapps  filesystems  keys  languages  logs  modules  scratch
```

As you can see this command has set up quite a few folders for us already.

blockchains - This folder stores data associated with all blockchains which epm is tracking. This is the most important folder for epm.
dapps - This is the main working folder for the decerver. It will be empty right now.
keys - This is folder to facilitate the import and export of keys. When you generate new keys this is where it goes. This is a temporary measure until key management can be handled properly
languages - This folder holds the configuration for compiling smart contract languages. We will show this config later.
logs - logs
scratch - This folder acts as a temporary workspace for epm and the languages compiler.


2)BLOCKCHAINZ!
When it comes down to it EPM has two primary functions. The first is to manage, create, and run blockchains. The second is to interact with those chains in a meaningful manner a universal commandline wallet of sorts. This section will deal with the first purpose.

So lets create a chain! The `new` sub command is used for this and takes several options.

To create a new thelonious chain type `epm new -type thel` this will create a new chain for you. Try it and you will see the following pop up in your command line editor.

```
  1 {
  2     "address": "0000000000THISISDOUG",
  3     "pdx": "",
  4     "doug": "",
  5     "unique": false,
  6     "private-key": "",
  7     "model": "",
  8     "no-gendoug": true,
  9     "consensus": "",
 10     "difficulty": 15,
 11     "public:mine": 0,
 12     "public:create": 0,
 13     "public:tx": 0,
 14     "maxgastx": "",
 15     "tapow": 0,
 16     "blocktime": 0,
 17     "vm": null,
 18     "accounts": [
 19         {
 20             "address": "0xbbbd0256041f7aed3ce278c56ee61492de96d001",
 21             "name": "",
 22             "balance": "1000000000000000000000000000000000000",
 23             "permissions": null,
 24             "stake": 0
 25         }
 26     ]
 27 }
 ```

 This is the genesis config file for a thelonious chain. Editing this json file can set/modify the permissions of the thelonious chain you are creating. I will not go into detail what all these options do here as for epm we don't care too much. Save and close this file. in vim this is done by typing `:wq`. After doing so you will see the following:

```
dennis@Juzo:~/.decerver/modules$ epm new -type thel
thel
2015/04/23 21:59:10 [EPM-CLI] Deployed and installed chain: thelonious/f1f1131d39303ae871b76deb6a875c7b9cdadab5
```

 The hex string that is printed in this case `f1f1131d39303ae871b76deb6a875c7b9cdadab5` is whats called the "chainid" and uniquely identifies the chain. As such your chainid will likely differ from the one given here.

 Where did this chain go? Its in the blockchains folder. If you move to blockchains/ you will see that there is a file named "HEAD" (which is empty right now) a folder named "refs" and finally a folder named "thelonious". This last folder is where all thelonious blockchains will be stored.

```
dennis@Juzo:~/.decerver/blockchains/thelonious$ ls
config.json  f1f1131d39303ae871b76deb6a875c7b9cdadab5  genesis.json
```

The config and genesis jsons in here are the defaults for thelonious chains and we will deal with them later. But you can see there is folder now with the chainid of the chain we created. All information regarding this chain will be stored in this folder.

 So can we start using the chain now? ... Not quite. The reason is that we have not told EPM that we wish to work with this chain. YOu can see this for yourself by typing `epm head`. The "epm head" sub command works similarly to "git branch" command in that it tells you what chain is being worked on right now. If we type the command right now we get the following:

```
dennis@Juzo:~/.decerver/modules$ epm head
2015/04/23 22:05:39 [EPM-CLI] There is no chain checked out
```

No chain checked out. :( so lets check out the chain we just created. `epm checkout chainid` is the command for this. So if we type `epm checkout f1f1131d39303ae871b76deb6a875c7b9cdadab5` we get ...

```
dennis@Juzo:~/.decerver/modules$ epm checkout f1f1131d39303ae871b76deb6a875c7b9cdadab5
2015/04/23 22:08:22 [EPM-CLI] open /home/dennis/.decerver/blockchains/refs/f1f1131d39303ae871b76deb6a875c7b9cdadab5: no such file or directory
```

um... what happened? Typing `epm head` will show that the chain is still not checked out.

The reason for this is that epm is a tool designed for compatibility with many different blockchains. The chainid while uniquely identifying a thelonious blockchain amoung thel chains, does not tell epm that it is a thelonious chain. So lets try again with `epm checkout thelonious/f1f1131d39303ae871b76deb6a875c7b9cdadab5`

No errors! excellent and we can verify that we have been successful with `epm head` again which shows:

```
dennis@Juzo:~/.decerver/blockchains/thelonious$ epm head
thelonious/f1f1131d39303ae871b76deb6a875c7b9cdadab5
```

We now have the chain checked out and could start working with it. Also notice that the `blockchains/HEAD` folder is now non-empty and reflects the checked out chain!

But working with the chainid was a little bit awkward. Who wants to type, remember or copy that hex string all the time? it would be nice if we had a human friendly reference to the chain. Much like names for branches instead of remembering commit numbers. ;)

So lets give this chain a name. The refs subcommand can be used for listing and adding references to chains. You can see that there are no references right now by typing `epm refs` which will print out the following.

```
dennis@Juzo:~/.decerver/blockchains/thelonious$ epm refs
Name:               Blockchain:                                                 Address:
```

Lets name this chain "bob" after my uncle. `epm refs add bob` will give the current chain the name bob. Don't believe me? After running it run `epm refs` again and you should see the following:

```
dennis@Juzo:~/.decerver/blockchains/thelonious$ epm refs
Name:               Blockchain:                                                 Address:            
bob                 thelonious/f1f1131d39303ae871b76deb6a875c7b9cdadab5         0x27cdf9906ed14a0e0c7583962b74dc44ec1dad3d
```

Congrats! Checked out and named blockchain! But that process was a little cumbersome no? Wouldn't it be nice if you didn't HAVE to type three separate commands? Well as it turns out we are lazy too! Thats why you can add flags to the chain creation command that will both checkout and name the chain after creating the chain. Lets try it out. Type: `epm new -type thel -name robert -o` once again vim will open `:wq` it again and you will see:

```
dennis@Juzo:~/.decerver/blockchains/thelonious$ epm new -type thel -name robert -o
thel
2015/04/23 22:38:12 [EPM-CLI] Deployed and installed chain: thelonious/d75dd3b7ee489f73670682390a4bdbaca97014e5
2015/04/23 22:38:12 [EPM-CLI] Checked out chain: thelonious/d75dd3b7ee489f73670682390a4bdbaca97014e5
2015/04/23 22:38:12 [EPM-CLI] Created ref robert to point to chain d75dd3b7ee489f73670682390a4bdbaca97014e5
```

It seems to have worked but what did we do? well the -name flag seems obvious, it will name the created chain as provided. The -o is a short flag for checking out the chain. Why o? Dunno ask ethan. -c was taken.

lets just double check that it all worked.

```
dennis@Juzo:~/.decerver/blockchains/thelonious$ epm head
thelonious/d75dd3b7ee489f73670682390a4bdbaca97014e5
dennis@Juzo:~/.decerver/blockchains/thelonious$ epm refs
Name:               Blockchain:                                                 Address:            
bob                 thelonious/f1f1131d39303ae871b76deb6a875c7b9cdadab5         0x27cdf9906ed14a0e0c7583962b74dc44ec1dad3d
robert              thelonious/d75dd3b7ee489f73670682390a4bdbaca97014e5         0xf06285513035838d9dd5af72b9e374622566defb
```

Yay! And now that we have more then one chain we can see a nice feature of the refs list. The currently checkout-ed chain will be coloured green and the others are white.

For completion sake I will also mention that there are ways to remove references from the refs list. `epm refs rm <name>` you can also delete chains all together with `epm rm <ref|chainid>` I don't think we need to demonstrate those since the lack of something is rather uninteresting. but if you want to try it, you can delete one of the chains above (don't worry we won't be using them) and verify that its folder in /blockchains/thelonious/ disappears. Also remember the /blockchains/refs/ folder? If you look in it now you will notice that small files now contain the references we have been creating.

The curious reader will also notice that the HEAD file has more then one line now and might wonder why. HEAD tracks a history of the chains of you have had checked out. The top line is the current HEAD. Is this useful? no clue.


2.b) Sharing is caring
Creating a chain that only lives on your computer is all well and good... but kind of lonely so lets make some friends. EPM has functionality for getting a chain from a peer called fetch. It works via `epm fetch <peerip>:<peerport>` the -name and -o flags work with this too. I'm not going to tell you to how to share a chain you have created locally with others. Mostly because I don't know how. Instead lets fetch an already existing thelonious chain! We (eris) have a collection of peer servers for the 2gather dapp's chain which (at the time of writing) has public permissions so we will be able to mine on it etc. Type: `epm fetch 104.236.146.58:15258 -name bobert -o`

zoooooom

```
dennis@Juzo:~/.decerver/blockchains/thelonious$ epm fetch 104.236.146.58:15258 -name bobert -o
2015/04/23 22:58:39 [EPM-CLI] Fetching state af50ddc5cc947d144fa7dc2b0bf3ab4b1da22d1e5366668b16cacf6266326f7b
2015/04/23 22:58:41 [EPM-CLI] Fetched genesis block for chain 49aa20cbeb2c1bf4a5d0863f1ac2e61b04cde50c
2015/04/23 22:58:41 [EPM-CLI] Checked out chain: thelonious/49aa20cbeb2c1bf4a5d0863f1ac2e61b04cde50c
2015/04/23 22:58:41 [EPM-CLI] Created ref bobert to point to chain 49aa20cbeb2c1bf4a5d0863f1ac2e61b04cde50c
```

We just got a chain from a peer that can have other people working off it. But what you have fetched at this point is just the genesis block of the chain. The chain is potentially far behind your peers. To start up the chain and allow it to sync type `epm run` which will run the currently checked out chain.

.... and you see nothing ....

This is because by default epm is going to hide the log messages from running the chain (it can be quite noisy) you and stop the chain from running with `ctrl+c` as normal. If you want to see the chain running you can use the global `-log #` flag where # is the log level (between 1 - lowest and 5 - highest) to see more detailed stuff while running for example `epm -log 3 run`. Note that it goes before the subcommand. you can experiment with it. At log level 5 you are going to be seeing vm execution at log level 3 you can see peering messages and is enough to see new block messages.

By itself `epm run` will only open yourself up to the network. If you want to mine you can add the "--mine" flag to the end. For example `epm -log 3 run --mine` this will show your computer mining. The peerservers for 2gather do not mine the chain themselves so if you aren't mining (and there is no one else on the chain) no new blocks will come in. 

## BUDDY BONUS!

If you have a friend who is also working with epm you can have a little bit of fun here. If one of you types `epm -log 3 run` and the other types `epm run --mine` the first person will not mine but will be able to see new blocks come in from the second user. Its kind of cool if you ask me.

You can do this yourself easily if you're using Docker:

```
# epm -log 3 run
```

Then in another shell:

```
$ docker run -it --name=epm2 epm
# epm init
# epm fetch 104.236.146.58:15258 -name bobert -o
# epm run --mine
```

How do we know this is working?  I don't see visual feedback on either node.

# **KEYS!** TODO


3) Poking the chain
The above covers the first purpose of EPM... that is the chain management aspect. Believe it or not that is the boring simple side of epm. Its real power is in the interactions with a chain. In essence EPM allows you to create transactions, inspect the chain, deploy individual smart contracts, and script the deployment of multi-contract suites (more useful for LLL but will be revamped soon to be an invaluable too for solidity contracts also)

There is a lot to this so i'm going to cover the basics of interaction via commandline and then move into how to write pdx files.

Blockchain interaction in epm revolves around vars, and 4 interaction commands. Lets start with vars. Vars are persistent chain specific variables which can be accessed and used at any time via {{varname}}. This will become more clear with examples. The "set" command is specifically designed to allow you to set the value stored in vars. For example `epm cmd set billy 0xdeadbeef` will set {{billy}} to 0xdeadbeef

```
dennis@Juzo:~/.decerver/blockchains/thelonious$ epm cmd set billy 0xdeadbeef
Storing: billy 0xdeadbeef
2015/04/23 23:59:47 [EPM] Executing job: set
```

We can inspect the value of a var through the "plop" command for example `epm plop vars billy` will give:

```
dennis@Juzo:~/.decerver/blockchains/thelonious$ epm plop vars billy
0xdeadbeef
```

The value stored in a var can be used in almost any command through the use of {{varname}} a simple example is `epm set bobby {{billy}}` which should copy the value of the var "billy" into the var "bobby" and sure enough:

```
dennis@Juzo:~/.decerver/blockchains/thelonious$ epm cmd set bobby {{billy}}
Storing: bobby 0xdeadbeef
2015/04/24 00:03:12 [EPM] Executing job: set
```

Vars will be very useful for what comes next.

The four main interaction commands are deploy, modify-deploy, transact, and call each serving its own purpose a brief description of each is:
deploy - compiles a smart contract and creates it on the checked out chain
modify-deploy - performs a substitution on the smart contract file and then deploys it. (useful if it depends on information from previous commands)
transact - create a transaction. For smart contracts with abi this will use the abi in the formation of the transaction
call - similar to transact but does not actually run the transaction. Also stores a returned value in a var.

The goal of this guide is to be very hands on with respect to what is happening with epm to give an intuitive feel for how it works and more importantly how you work with it. As such describing the format of each of these in a theory perspective will be of limited use. EPM is very well documented in its README. At the same time however in the interest of limiting scope of this document I can not give a detailed explanation of how to write smart contracts. Instead, a set of pre-written smart contracts in the solidity language are provided named example#.sol which we will be using.

So lets work with example1.sol to start. This is a simple contract which returns the value you sent it last. This will be very informative for the difference between call and transact.

To deploy the smart contract we are going to need an ethereum chain checked out (this is due to vm compatibility reasons Thelonious' vm will be updated soon to accomadate the solidity language). Creating an ethereum chain is exactly like any other chain. EPM is designed to be (mostly) chain agnostic.

`epm new -type eth -name ethereum -o`

One difficulty when working with ethereum however is their use of gas. In order to do ANYTHING on this chain we will need to have some ether meaning we need to mine this chain for a few minutes. From before we know this just requires:

`epm run --mine`

ctrl+c out of that after a few minutes have passed. If you want to watch the blocks being mined you can add in the log level flag as discussed earlier.

Now that we have ether we can deploy this contract. (Note if you deployed without mining first, it would appear as if you were successful but typing `epm accounts` would show that this actually failed as there are no "contract" accounts.) 

From command line deploying a single contract is enacted via:

`epm cmd deploy example1.sol cav`

where "cav" is a var name where the deployed contract address is stored. For the example above we would get the following output:

```
dennis@Juzo:~/Documents/epm-walkabout$ epm cmd deploy example1.sol test
Chain type: ethereum
Chain type: ethereum

2015/04/27 11:30:05 [EPM] Executing job: deploy
2015/04/27 11:30:05 [EPM] Deployed 0x30983a2aa1e4e50d337e145574843156b6f8cbb1 as test
Storing: test 0x30983a2aa1e4e50d337e145574843156b6f8cbb1
```
We can verify that the contract has been created via `epm accounts` 

```
dennis@Juzo:~/Documents/epm-walkabout$ epm accounts
Chain type: ethereum
Chain type: ethereum
1a26338f0d905e295fccb71fa9ea849ffa12aaf4 account
2ef47100e0787b915105fd5e3f4ff6752079d5cb account
30983a2aa1e4e50d337e145574843156b6f8cbb1 contract
6c386a4b26f73c802f34673f7248bb118f97424a account
a0e923740509ff2173981bd62c297daa43f6411b account
b9c015918bdaba24b4ff057a92a3873d6eb201be account
c5ac1950c7fa7f8f0abd54e83495425fc0c5fc2e account
cd2a3d9f938e13cd947ec05abc7fe734df8dd826 account
dbdbdb2cbd23b783741e8d7fcf51e459b497e4a6 account
e4157b34ea9615cfbde6b4fda419828124b70c78 account
e6716f9544a56c530d868e4bfbacb172315bdead account
```

This contract we have now created on the blockchain has two functions in it "get" and "set" which will get and set a single 32 byte value. This is about as simple as contracts get. By default the value stored is 0xFEEDFACE which we can inspect via:

`epm cmd call {{test}} get output`

What is this command doing? Its calling the contract stored in {{test}} which we set during the deploy and calling its function get which returns the stored value. This value is stored in the var by the name "output". The output of the above command is:

```
dennis@Juzo:~/Documents/epm-walkabout$ epm cmd call {{test}} get output
Chain type: ethereum
Chain type: ethereum

ABI Spec {map[get:get() set:set(bytes32)]}
2015/04/27 12:06:01 [EPM] Executing job: call
2015/04/27 12:06:01 [EPM] Sent [get] to 0x30983a2aa1e4e50d337e145574843156b6f8cbb1
2015/04/27 12:06:01 [EPM] Result: 0x00000000000000000000000000000000000000000000000000000000feedface
Storing: output 0x00000000000000000000000000000000000000000000000000000000feedface
```

Huzzah! We have retrieved the appropriate value. We can check that it really is in the "output" var:

```
dennis@Juzo:~/Documents/epm-walkabout$ epm plop vars output
Chain type: ethereum
Chain type: ethereum
0x00000000000000000000000000000000000000000000000000000000feedface
```

Now lets store a value by calling the set function in the contract!

```
dennis@Juzo:~/Documents/epm-walkabout$ epm cmd call {{test}} set 0xdeadbeef output2 
Chain type: ethereum
Chain type: ethereum

ABI Spec {map[set:set(bytes32) get:get()]}
0 [222 173 190 239] [0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 222 173 190 239]
2015/04/27 12:09:34 [EPM] Executing job: call
Storing: output2 0x
2015/04/27 12:09:34 [EPM] Sent [set 0xdeadbeef] to 0x30983a2aa1e4e50d337e145574843156b6f8cbb1
2015/04/27 12:09:34 [EPM] Result: 0x
dennis@Juzo:~/Documents/epm-walkabout$ epm cmd call {{test}} get output3
Chain type: ethereum
Chain type: ethereum

ABI Spec {map[get:get() set:set(bytes32)]}
2015/04/27 12:10:35 [EPM] Executing job: call
Storing: output3 0x00000000000000000000000000000000000000000000000000000000feedface
2015/04/27 12:10:35 [EPM] Sent [get] to 0x30983a2aa1e4e50d337e145574843156b6f8cbb1
2015/04/27 12:10:35 [EPM] Result: 0x00000000000000000000000000000000000000000000000000000000feedface
dennis@Juzo:~/Documents/epm-walkabout$ epm plop vars output3
Chain type: ethereum
Chain type: ethereum
0x00000000000000000000000000000000000000000000000000000000feedface
```

Wait... what? The stored value didn't change!!! EPM is broken! blockchains don't work! the sky is falling!

Ok maybe not. The reason nothing changed is because we used `call` for the set transaction and call does not commit the state change it just "simulates" the transaction and stores the return value. In order to actually do a proper state interaction, we need to `transact` with the chain. As of right now transact does not store return values so we will do the following:

```
dennis@Juzo:~/Documents/epm-walkabout$ epm cmd transact {{test}} set 0xdeadbeef
Chain type: ethereum
Chain type: ethereum

ABI Spec {map[get:get() set:set(bytes32)]}
0 [222 173 190 239] [0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 222 173 190 239]
2015/04/27 12:40:12 [EPM] Executing job: transact
2015/04/27 12:40:12 [EPM] Sent [set 0xdeadbeef] to 0x30983a2aa1e4e50d337e145574843156b6f8cbb1
dennis@Juzo:~/Documents/epm-walkabout$ epm cmd call {{test}} get output4
Chain type: ethereum
Chain type: ethereum

ABI Spec {map[get:get() set:set(bytes32)]}
2015/04/27 12:40:30 [EPM] Executing job: call
Storing: output4 0x00000000000000000000000000000000000000000000000000000000deadbeef
2015/04/27 12:40:30 [EPM] Sent [get] to 0x30983a2aa1e4e50d337e145574843156b6f8cbb1
2015/04/27 12:40:30 [EPM] Result: 0x00000000000000000000000000000000000000000000000000000000deadbeef
dennis@Juzo:~/Documents/epm-walkabout$ epm plop vars output4
Chain type: ethereum
Chain type: ethereum
0x00000000000000000000000000000000000000000000000000000000deadbeef
```

AWESOME its working. What if you wanted to see the vm actually execute these instructions? Add the global `-log 5` flag to the command and you will get it all spewed out at you (try it for fun). This will be mostly meaningless to anyone not intimately familiar with the workings of the evm so I omit further discussion of it here.

BUDDY BONUS

If you have a Buddy and you are working on a shared chain you can get values out that your buddy has put in. This works the same as above just remember to have both computers run the chain to sync inbetween commands!

**PDX**

All of the above is extremely useful. pdx files are the icing on the cake. pdx files are a simple script syntax parsable by epm to perform the commands we have used above. Its formatting is a tad unusual but you will get use to it. pdx files have two main uses at this point. Automated testing and deployment. For both it works the same way, Testing makes use of an additional command we did not use above... `Assert`.

So lets start simple we want to automate the (extremely) complex process we did above. So the first step is to deploy the contract. If you look in example1_1.pdx you will see the following section

```
#Things after a # are comments
deploy:
	example1.sol => test1
```

This works identical to `epm cmd deploy example1.sol test` that we did above. To run a .pdx file type in the command line `epm deploy pdxfilename.pdx`. The syntax is simply a remnant of earlier formatting used and will in all likelyhood be changed to something a little friendlier.

For this .pdx file this is all that it has in it. This is because I only one this file to deploy this one contract. What use is that? very little honsetly but this is a simple contract. As things increase in complexity pdx files help automate complex deployment proceedures.

For example lets say we wanted to always overwrite the default value stored in the contract with the contracts own address how would we do this via command line? First we would deploy with `epm cmd deploy example1.sol test` and would follow it with `epm cmd transact {{test}} set {{test}}` This will transact with the {{test}} smart contract and set the value it stores to the value stored in {{test}}. Take a look at example1_2.pdx and you can see the formatting for this:

```
#Deploying the contract 
deploy:
	example1.sol => test2

#Storing the contracts address in the contract
transact:
	{{test}} => set {{test}}
```

This is pretty nice and it should be becoming clear how .pdx files can help your work flow but where they really begin to shine are as a debugging tool. This is where `call` and `assert` come in. Take a look at example1_3.pdx. This file does both deployment and testing of the contract we have written.

```
#Same old deployment
deploy:
	example1.sol => test3

#Time for some testing

#This is unecessary but included for completeness
set:
	0xdeadbeef => testval

#Check that default value is 0xFEEDFACE 
call:
	{{test3}} => get => output1
assert:
	{{output1}} => 0xfeedface

#Set the value and then test that it actually changed
transact:
	{{test3}} => set {{testval}}
commit:

call:
	{{test3}} => get => output2
assert:
	{{output2}} => {{testval}}
```

This is a far more complex file. It has multiple stages. First it deploys the contract and then stores a value for testing with. this is meerly a demonstration of the use of the set command in pdx's. After that we get a call-assert pairing. Call retrieves a value from the contract and stores it in output1. Assert then checks if the value retrieved matches 0xFEEDFACE. If for some reason it did not match assert would throw an error and stop execution. A behaviour which can be customized to make it friendly for automated testing structures such as circleci.

This is followed by a transaction and a ... commit? WTF? Am I just making this up as I go along? Well kinda. Commit is a command only required in pdx files. When using command line commands like deploy and transact are automatically mined in after the fact. But in a pdx for maximal efficiency mining (or commiting the state changes) only happens when explicitly stated. We commit after the transact to ensure the state has been committed to for the call that comes next. The rest of the file follows from the same as before.

If you want to see what happens when an assert try editing either the .pdx or the contract itself.

One tiny note about pdx's and vars. Vars are stored PER CHAIN. A var created in one chain is not accessible from another chain. ALSO the vars created and edited in pdx sessions are persistent! So they can be used from commandline or other pdx files.

For a take-home assignment take a look at the files example2.sol (which is actually the solidity version of gendoug which runs permissions on thelonious chains) and the associated example2_1.pdx which is the deployment and testing suite. It follows much the same formula as above. One important thing to note is that is assumes you have set the var named "myaddr" to the address you are using. The easiest method to do this is the following:

`epm plop addr` will show you the address you are using on the chain. Copy this value (lets say its a12b3de22...)

`epm cmd set myaddr 0xa12b3de22...` Note that the address has been pre-pended with "0x" indicating its a hex value. This is important.

now you can deploy the example2_1.pdx properly. If you do not do the previous steps most of the assertions will fail. 

Thats all for today. Despite the length of it, this is just the basics of epm. And we are always looking to improve its functionality. I hope this helped give you a sense of how epm works and how you work with epm! 

Class dismissed

Thanks to everyone who has helped in the testing and given feedback to this tutorial. It is invaluable.
