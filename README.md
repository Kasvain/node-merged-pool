[![Build Status](https://travis-ci.org/sigwo/node-merged-pool.png?branch=master)](https://travis-ci.org/sigwo/node-merged-pool)

High performance Stratum poolserver in Node.js. One instance of this software can startup and manage multiple coin
pools, each with their own daemon and stratum ports :)

#### Notice
I am trying to make this useable software. On my list of up-front todos:

* Multiple PoWAUX coins on each main chain - Issues
* Profit Switching - Complete (UNOMP)
* Built-in stratum redundancy 

Frontend and payment enhancements will be a separate repo. [UNOMP](//github.com/UNOMP/unified-node-open-mining-portal)

#### Why
The software that I forked was a wonderful beginning that never came to fruition. I hope to bring Node stratum into the
mainstream of merged mining.

Features
----------------------------------
* Merged Mining Support
* Daemon RPC interface
* Stratum TCP socket server
* Block template / job manager
* P2P to get block notifications as peer node
* Optimized generation transaction building
* Connecting to multiple daemons for redundancy
* Process share submissions
* Session managing for purging DDoS/flood initiated zombie workers
* Auto ban IPs that are flooding with invalid shares
* __POW__ (proof-of-work) & __POS__ (proof-of-stake) support
* Transaction messages support
* Vardiff (variable difficulty / share limiter)
* When started with a coin daemon that hasn't finished syncing to the network it shows the blockchain download progress and initializes once synced

#### Hashing algorithms supported:
* ✓ __SHA256__ (Bitcoin, Freicoin, Peercoin/PPCoin, Terracoin, etc..)
* ✓ __Scrypt__ (Litecoin, Dogecoin, Feathercoin, etc..)
* ✓ __Scrypt-Jane__ (YaCoin, CopperBars, Pennies, Tickets, etc..)
* ✓ __Scrypt-N__ (Vertcoin [VTC])
* ✓ __Quark__ (Quarkcoin [QRK])
* ✓ __X11__ (Darkcoin [DRK], Hirocoin, Limecoin)
* ✓ __X13__ (MaruCoin, BoostCoin)
* ✓ __NIST5__ (Talkcoin)
* ✓ __Keccak__ (Maxcoin [MAX], HelixCoin, CryptoMeth, Galleon, 365coin, Slothcoin, BitcointalkCoin)
* ✓ __Skein__ (Skeincoin [SKC])
* ✓ __Groestl__ (Groestlcoin [GRS])

May be working (needs additional testing):
* ? *Blake* (Blakecoin [BLC])
* ? *Fugue* (Fuguecoin [FC])
* ? *Qubit* (Qubitcoin [Q2C], Myriadcoin [MYR])
* ? *SHAvite-3* (INKcoin [INK])
* ? *Sha1* (Sha1coin [SHA], Yaycoin [YAY])
* ? *Lyra2RE*
* ? *Lyra2REv2* (Vertcoin [VTC])

Not working currently:
* *Groestl* - for Myriadcoin
* *Keccak* - for eCoin & Copperlark
* *Hefty1* (Heavycoin [HVC])


Requirements
------------
* Node v0.10 (For help, see https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-an-ubuntu-14-04-server)
* Coin daemon for primary and auxillary coins
* Patience :)

Example Usage
-------------

#### Install as a node module by cloning repository

```bash
git clone https://github.com/sigwo/node-merged-pool
sudo apt-get install build-essential libssl-dev
cd node-merged-pool
npm update
```

#### Module usage

=======
Create the configuration for your coin:

Possible options for `algorithm`: *sha256, scrypt, scrypt-jane, scrypt-n, quark, x11, keccak, blake,
skein, groestl, fugue, shavite3, hefty1, qubit, or sha1*.

```javascript
var myCoin = {
    "name": "Dogecoin",
    "symbol": "DOGE",
    "algorithm": "scrypt",
    "nValue": 1024, //optional - defaults to 1024
    "rValue": 1, //optional - defaults to 1
    "txMessages": false, //optional - defaults to false,

    /* Magic value only required for setting up p2p block notifications. It is found in the daemon
       source code as the pchMessageStart variable.
       For example, litecoin mainnet magic: http://git.io/Bi8YFw
       And for litecoin testnet magic: http://git.io/NXBYJA */
     "peerMagic": "fbc0b6db" //optional
     "peerMagicTestnet": "fcc1b7dc" //optional
};
```

If you are using the `scrypt-jane` algorithm there are additional configurations:

```javascript
var myCoin = {
    "name": "Freecoin",
    "symbol": "FEC",
    "algorithm": "scrypt-jane",
    "chainStartTime": 1375801200, //defaults to 1367991200 (YACoin) if not used
    "nMin": 6, //defaults to 4 if not used
    "nMax": 32 //defaults to 30 if not used
};
```

If you are using the `scrypt-n` algorithm there is an additional configuration:
```javascript
var myCoin = {
    "name": "Execoin",
    "symbol": "EXE",
    "algorithm": "scrypt-n",
    /* This defaults to Vertcoin's timetable if not used. It is required for scrypt-n coins that
       have modified their N-factor timetable to be different than Vertcoin's. */
    "timeTable": {
        "2048": 1390959880,
        "4096": 1438295269,
        "8192": 1485630658,
        "16384": 1532966047,
        "32768": 1580301436,
        "65536": 1627636825,
        "131072": 1674972214,
        "262144": 1722307603
    }
};
```

If you are using the `keccak` algorithm there are additional configurations *(The rare `normalHashing` keccak coins
such as Copperlark and eCoin don't appear to work yet - only the popular ones like Maxcoin are)*:
```javascript
var myCoin = {
    "name": "eCoin",
    "symbol": "ECN",
    "algorithm": "keccak",

    /* This is not required and set to false by default. Some coins such as Copperlark and eCoin
       require it to be set to true. Maxcoin and most others are false. */
    "normalHashing": true
};
```


Create and start new pool with configuration options and authentication function, complete example:

```javascript
var Stratum = require('./lib/index.js');

var myCoin = {
    "name": "Catcoin",
    "symbol": "CTC",
    "algorithm": "scrypt"
};

var myAuxCoins = [{
	"name": "Syscoin",
	"symbol": "SYS",
	"algorithm": "scrypt",

	/*  */
	"daemons": [
		{   //Main daemon instance
	    	"host": "127.0.0.1",
	        "port": 23327, // **NOT ACTUAL PORT**
	        "user": "syscoinrpc",
	        "password": "By66dCmyX44uUbA7P3qqXJQeT3Ywd8dZ4dJdfgxCAxbg"
	    }
	],
}];

var pool = Stratum.createPool({

    "coin": myCoin,

    "auxes": myAuxCoins,

    // Payout address - for primary node only
    "address": "9iKR9FwwEbWCiU3ZhCAs5dQRVkYoC288Go", //Address to where block rewards are given;
    // shared between all coins, so copy your private key over with dumpprivkey and importprivkey

    /* Block rewards go to the configured pool wallet address to later be paid out to miners,
       except for a percentage that can go to, for examples, pool operator(s) as pool fees or
       or to donations address. Addresses or hashed public keys can be used. Here is an example
       of rewards going to the main pool op, a pool co-owner, and NOMP donation. */
    "rewardRecipients": {
        /* 0.1% donation to NOMP. This pubkey can accept any type of coin, please leave this in
           your config to help support NOMP development. */
        "22851477d63a085dbc2398c8430af1c09e7343f6": 0.1
    },

    "blockRefreshInterval": 1000, //How often to poll RPC daemons for new blocks, in milliseconds


    /* Some miner apps will consider the pool dead/offline if it doesn't receive anything new jobs
       for around a minute, so every time we broadcast jobs, set a timeout to rebroadcast
       in this many seconds unless we find a new job. Set to zero or remove to disable this. */
    "jobRebroadcastTimeout": 55,

    //instanceId: 37, //Recommend not using this because a crypto-random one will be generated

    /* Some attackers will create thousands of workers that use up all available socket connections,
       usually the workers are zombies and don't submit shares after connecting. This features
       detects those and disconnects them. */
    "connectionTimeout": 600, //Remove workers that haven't been in contact for this many seconds

    /* Sometimes you want the block hashes even for shares that aren't block candidates. */
    "emitInvalidBlockHashes": false,

    /* Enable for client IP addresses to be detected when using a load balancer with TCP proxy
       protocol enabled, such as HAProxy with 'send-proxy' param:
       http://haproxy.1wt.eu/download/1.5/doc/configuration.txt */
    "tcpProxyProtocol": false,

    /* If a worker is submitting a high threshold of invalid shares we can temporarily ban their IP
       to reduce system/network load. Also useful to fight against flooding attacks. If running
       behind something like HAProxy be sure to enable 'tcpProxyProtocol', otherwise you'll end up
       banning your own IP address (and therefore all workers). */
    "banning": {
        "enabled": true,
        "time": 600, //How many seconds to ban worker for
        "invalidPercent": 50, //What percent of invalid shares triggers ban
        "checkThreshold": 500, //Check invalid percent when this many shares have been submitted
        "purgeInterval": 300 //Every this many seconds clear out the list of old bans
    },

    /* Each pool can have as many ports for your miners to connect to as you wish. Each port can
       be configured to use its own pool difficulty and variable difficulty settings. varDiff is
       optional and will only be used for the ports you configure it for. */
    "ports": {
        "3032": { //A port for your miners to connect to
            "diff": 32, //the pool difficulty for this port

            /* Variable difficulty is a feature that will automatically adjust difficulty for
               individual miners based on their hashrate in order to lower networking overhead */
            "varDiff": {
                "minDiff": 8, //Minimum difficulty
                "maxDiff": 512, //Network difficulty will be used if it is lower than this
                "targetTime": 15, //Try to get 1 share per this many seconds
                "retargetTime": 90, //Check to see if we should retarget every this many seconds
                "variancePercent": 30 //Allow time to very this % from target without retargeting
            }
        },
        "3256": { //Another port for your miners to connect to, this port does not use varDiff
            "diff": 256 //The pool difficulty
        }
    },

    /* Recommended to have at least two daemon instances running in case one drops out-of-sync
       or offline. For redundancy, all instances will be polled for block/transaction updates
       and be used for submitting blocks. Creating a backup daemon involves spawning a daemon
       using the "-datadir=/backup" argument which creates a new daemon instance with it's own
       RPC config. For more info on this see:
          - https://en.bitcoin.it/wiki/Data_directory
          - https://en.bitcoin.it/wiki/Running_bitcoind */
    // This is just for the primary coin
    "daemons": [
        {   //Main daemon instance
            "host": "127.0.0.1",
            "port": 9932,
            "user": "catcoinrpc",
            "password": "74QL5rZ9h8xZbLmwdNzW3cBRjJNQ6fy3b8pB5bJ6oURF"
        }
    ],


    /* This allows the pool to connect to the daemon as a node peer to receive block updates.
       It may be the most efficient way to get block updates (faster than polling, less
       intensive than blocknotify script). It requires the additional field "peerMagic" in
       the coin config. */
    // Again, this is just for the primary coin
    "p2p": {
        "enabled": false,

        /* Host for daemon */
        "host": "127.0.0.1",

        /* Port configured for daemon (this is the actual peer port not RPC port) */
        "port": 19333,

        /* If your coin daemon is new enough (i.e. not a shitcoin) then it will support a p2p
           feature that prevents the daemon from spamming our peer node with unnecessary
           transaction data. Assume its supported but if you have problems try disabling it. */
        "disableTransactions": true

    }

// This is the authorize function. Connect this to the database in your code to check if the client is valid
}, function(ip, port , workerName, password, callback){ //stratum authorization function
    console.log("Authorize " + workerName + ":" + password + "@" + ip);
    callback({
        error: null,
        authorized: true,
        disconnect: false
    });
});

/*

'data' object contains:
    job: 4, //stratum work job ID
    ip: '71.33.19.37', //ip address of client
    port: 3333, //port of the client
    worker: 'matt.worker1', //stratum worker name
    height: 443795, //block height
    blockReward: 5000000000, //the number of satoshis received as payment for solving this block
    difficulty: 64, //stratum worker difficulty
    shareDiff: 78, //actual difficulty of the share
    blockDiff: 3349, //block difficulty adjusted for share padding
    blockDiffActual: 3349 //actual difficulty for this block


    //AKA the block solution - set if block was found
    blockHash: '110c0447171ad819dd181216d5d80f41e9218e25d833a2789cb8ba289a52eee4',

    //Exists if "emitInvalidBlockHashes" is set to true
    blockHashInvalid: '110c0447171ad819dd181216d5d80f41e9218e25d833a2789cb8ba289a52eee4'

    //txHash is the coinbase transaction hash from the block
    txHash: '41bb22d6cc409f9c0bae2c39cecd2b3e3e1be213754f23d12c5d6d2003d59b1d,

    error: 'low share difficulty' //set if share is rejected for some reason
*/
pool.on('share', function(isValidShare, isValidBlock, data){

    //maincoin
    var coin = myCoin.name;
    storeShare(isValidShare, isValidBlock, data, coin);

	//loop through auxcoins
	for(var i = 0; i < myAuxCoins.length; i++) {
		coin = myAuxCoins[i].name;
		storeShare(isValidShare, isValidBlock, data, coin);
	}
    
});

//store each share/block submission for a given coin
function storeShare(isValidShare, isValidBlock, data, coin) {
	var redisCommands = [];
	
	if (isValidShare){
        redisCommands.push(['hincrbyfloat', coin + ':shares:roundCurrent', data.worker, data.difficulty]);
        redisCommands.push(['hincrby', coin + ':stats', 'validShares', 1]);
    }
    else{
        redisCommands.push(['hincrby', coin + ':stats', 'invalidShares', 1]);
    }
    /* Stores share diff, worker, and unique value with a score that is the timestamp. Unique value ensures it
       doesn't overwrite an existing entry, and timestamp as score lets us query shares from last X minutes to
       generate hashrate for each worker and pool. */
    var dateNow = Date.now();
    var hashrateData = [ isValidShare ? data.difficulty : -data.difficulty, data.worker, dateNow];
    redisCommands.push(['zadd', coin + ':hashrate', dateNow / 1000 | 0, hashrateData.join(':')]);

    if (isValidBlock){
        redisCommands.push(['rename', coin + ':shares:roundCurrent', coin + ':shares:round' + data.height]);
        redisCommands.push(['sadd', coin + ':blocksPending', [data.blockHash, data.txHash, data.height].join(':')]);
        redisCommands.push(['hincrby', coin + ':stats', 'validBlocks', 1]);
    }
    else if (shareData.blockHash){
        redisCommands.push(['hincrby', coin + ':stats', 'invalidBlocks', 1]);
    }

    connection.multi(redisCommands).exec(function(err, replies){
        if (err)
            logger.error(logSystem, logComponent, logSubCat, 'Error with share processor multi ' + JSON.stringify(err));
    });
    
    console.log('share data: ' + JSON.stringify(data));
}

/*
    Called when a block, auxillery or primary, is found
    coin: The symbol of the coin found. ex: 'LTC'
    blockHash: The hash of the block found and confirmed (at least for now) is in the blockchain.
*/
pool.on('block', function(coin, height, blockHash, txHash) {
    console.log('Mined block on ' + coin + ' network!');
    console.log('HEIGHT: ' + height);
    console.log('HASH: ' + blockHash);
    console.log('TX: ' + txHash);
});

/*
'severity': can be 'debug', 'warning', 'error'
'logKey':   can be 'system' or 'client' indicating if the error
            was caused by our system or a stratum client
*/
// Go ahead and combine these with your logs if you want :)
pool.on('log', function(severity, logKey, logText){
    console.log(severity + ': ' + '[' + logKey + '] ' + logText);
});
```

Start pool
```javascript
pool.start();
```

Credits
-------
* [zone117x](//github.com/zone117x) - Head developer of the original stratum mining pool for node.js
* [vekexasia](//github.com/vekexasia) - co-developer & great tester
* [LucasJones](//github.com/LucasJones) - got p2p block notify working and implemented additional hashing algos
* [TheSeven](//github.com/TheSeven) - answering an absurd amount of my questions, found the block 1-16 problem, provided example code for peer node functionality
* [pronooob](https://dogehouse.org) - knowledgeable & helpful
* [Slush0](//github.com/slush0/stratum-mining) - stratum protocol, documentation and original python code
* [viperaus](//github.com/viperaus/stratum-mining) - scrypt adaptions to python code
* [ahmedbodi](//github.com/ahmedbodi/stratum-mining) - more algo adaptions to python code and unomp dev
* [steveshit](//github.com/steveshit) - ported X11 hashing algo from python to node module
* [KillerByte](//github.com/KillerByte) - for beginning this creation

Donations
---------
Below is my donation address. The original dev addresses are listed because I felt scammy if I removed them. They no longer are supporting the current development
effort. Please donate to:

* BTC: `1BRUcdAdjdQoAgUwcN7nbNDzqhL92ub3xE`
* Cryptsy Trade Key: `197f17af3751709b2c7f076a2d3393e064022e91`


Original author (zone117x):

* BTC: `1KRotMnQpxu3sePQnsVLRy3EraRFYfJQFR`
* LTC: `LKfavSDJmwiFdcgaP1bbu46hhyiWw5oFhE`
* VTC: `VgW4uFTZcimMSvcnE4cwS3bjJ6P8bcTykN`
* MAX: `mWexUXRCX5PWBmfh34p11wzS5WX2VWvTRT`
* QRK: `QehPDAhzVQWPwDPQvmn7iT3PoFUGT7o8bC`
* DRK: `XcQmhp8ANR7okWAuArcNFZ2bHSB81jpapQ`
* DOGE: `DBGGVtwAAit1NPZpRm5Nz9VUFErcvVvHYW`
* Cryptsy Trade Key: `254ca13444be14937b36c44ba29160bd8f02ff76`

License
-------
Released under the GNU General Public License v2

http://www.gnu.org/licenses/gpl-2.0.html
