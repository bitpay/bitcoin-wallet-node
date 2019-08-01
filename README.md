### Purpose
To export/extract keys from a Bitcoin wallet.dat file. This module can be imported and used programmatically from another nodejs module, or you can use a convenient command line tool.

### Dependencies/Prerequisties
* C++ compiler and build tools
* Mac/Linux (Windows not yet supported)
* node version >= 4.x

### Installation
```bash
$ npm install
```

### Usage
The command line tool can be used as follows:
```bash
wallet-bdb2jsonl -f wallet.dat
```

Or you may load and use it as a nodejs module:
```javascript
var exporter = require('bitcore-bdb2jsonl');

exporter({ filePath: './wallet.dat' }); //to stdout -or- send in your own emitter, just needs to be able to emit a data, close and error signal

var EventEmitter = require('events');

var emitter = new EventEmitter();

exporter({ filePath: './wallet.dat', emitter: emitter }); //send in an emitter
emitter.on('data', function(value) {
  console.log(value);
});
exporter.start(); //this allows the db to be opened, read, and sent out record by record
export.close(); //this will finish sending the record in progress, then close the stream out for good
```

To pause/resume the exporter
```bash
exporter.pause();
exporter.resume();
```
### Implementation notes
The process doing the reading of the database is on a separate thread from the main node process (the one running your script).
We can't rely on the normal reactor pattern coding when it comes to signaling the worker process. The worker process doesn't know anything about setImmediate or nextTick, etc.. This means that a start function must be explicitly called if we are going to have any chance predicting when a subsequent pause signal might be considered by the worker thread. You may call pause() before start() for the purposes of emitting one record before pausing. Calling pause() after start() will ensure that will get at least one more record (but maybe 2 or 3) before pausing. Calling resume() will pick up where things left off.

### Common issues
* If you get an error/exception about crypto support not available, then please read the following:
  * This native addon includes Berkeley DB source code compatible with Bitcoin's wallet.dat file, but it does not come with cryptography support. Government export restrictions are to blame.
  * Wallet.dat files used in older versions of Bitcoin used Berkeley DB-based crypto (such aes-cbc routines). Newer Bitcoin wallet.dat files use crypto routines provided by Openssl or Bitcoin's source code directly. So if you have an older wallet.dat file needing crypto support in Berkeley DB, then get Berkeley DB's source code from Oracle, version 4.8.x, and compile it and install it. DO NOT get source code that has 'NC' on the file name. You can also get older versions of Berkeley DB from brew or apt-get. I've added a conditional check during the build process that checks your system's library paths for "libdb_cxx-4.8.{a|so|dylib}", the build system will use your installed library before compiling the included source code.
* This module does not handle unencrypted keys (e.g. wallet.dat files that are not encrypted). It is incredibly dangerous to store a wallet file unencrypted. Please always encrypt your wallet.dat file in Bitcoin.
* The "master" key is always streamed out first. This allows consumers of the output to decrypt the master key with a passphrase, then decrypt each key, if desired, for verification or construction of a new transaction.
