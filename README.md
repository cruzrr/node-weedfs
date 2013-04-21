# node-weedfs (weed-fs)

This project is a node.js client library for the Weed-FS REST interface.

# What is Weed-FS?

[Weed-FS](http://code.google.com/p/weed-fs/) is a simple and highly scalable distributed file system. It focuses on two objectives:
* storing billions of files!
* and serving them fast!

Weed-FS chose to implement only a key~file mapping instead of supporting full POSIX file system semantics. This can be called "NoFS". (Similar to "NoSQL")

Instead of managing all file metadata in a central master, Weed-FS manages file volumes in the central master, and allows volume servers
to manage files and the metadata. This relieves concurrency pressure from the central master and spreads file metadata into memory on the volume servers
allowing faster file access with just one disk read operation!

Weed-FS models after [Facebook's Haystack design paper](http://static.usenix.org/event/osdi10/tech/full_papers/Beaver.pdf) and costs only 40 bytes disk storage for each file's metadata. It is so simple with O(1) disk read that anyone is more than welcome to challenge the
performance with actual use cases.

# Examples
```javascript
var weedClient = require("weed-fs");

var weedfs     = new weedClient({
	server:		"localhost",
	port:		"9333"
});

weedfs.write("./file.png", function(fileInfo) {
	console.log(fileInfo);
});
```

# write(file(s), [{opts}], cb)

Use the <code>write()</code> function to store files.  The callback recieves the parsed JSON response.

Anything passed to the <code>{opts}</code> is made into a query string and
is used with the <code>/dir/assign</code> HTTP request.  You can use this to define the replication strategy.

```javascript
client.write("./file.png", {replication: 000}, function(fileInfo) {
	if (fileInfo.error) {
		throw fileInfo.error;
	}

	console.log(fileinfo);
});
```

You can also write multiple files:
```javascript
client.write(["./fileA.jpg", "./fileB.jpg"], function(fileInfo) {
	// This callback will be called for both fileA and fileB.
	// The fid's will be the same, to access each variaton just
	// add _ARRAYINDEX to the end of the fid. In this case fileB
	// would be: fid + "_1"
	
	var fidA = fileInfo;
	var fidB = fileInfo + "_1";
	
	console.log(fileInfo);
}
```

# read(fileId, [stream, cb])

The read function supports streaming.  To use simply do:

```javascript
client.read(fileId, fs.createWriteStream("read.png"));
```

If you prefer not to use streams just use:

```javascript
client.read(fileId, function(err, response, body) {
	if (err) {
		throw err;
	}

	// Here's your data:
	var filedata = body;
});
```

# find(file, cb)

This function can be used to find the location(s) of a file amongst the cluster.

```javascript
client.find(fileId, function(public, servers) {
	console.log(public[0]);

	// servers contains the non-public URLs.  Use this for editing and removing.
});
```

# remove(file, [server,] cb)

This function will delete a file from the store.  If <code>server</code> is specified, the
file will only be removed from that location.  Otherwise it will be deleted from all locations.

```javascript
client.remove(fileId, function(err) {
	if (err) {
		throw err;
	}

	console.log("removed file.");
});
```
