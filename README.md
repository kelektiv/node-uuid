# node-uuid

Simple, fast generation of [RFC4122 (v1 and v4)](http://www.ietf.org/rfc/rfc4122.txt) UUIDS.  It runs in node.js and all major browsers.

## Installation

    npm install node-uuid

### In browser

```html
<script src="uuid.js"></script>
```

### In node.js

```javascript
var uuid = require('node-uuid');
uuid.v1(); // -> v1 uuid
uuid.v4(); // -> v4 uuid

// ... or if you just need to generate uuids of one type and don't need helpers ...
var uuid = require('node-uuid').v1;
uuid(); // -> v1 uuid

// ... or ...
var uuid = require('node-uuid').v4;
uuid(); // -> v4 uuid
```

## Usage

### Generate a String UUID

```javascript
var id = uuid.v4([options, [buffer, [offset]]]); // -> '92329d39-6f5c-4520-abfc-aab64544e172'
```

### Generate a Binary UUID

```javascript
// Simple form - allocates a Buffer/Array for you
var buffer = uuid.v4('binary');
// node.js -> <Buffer 08 50 05 c8 9c b2 4c 07 ac 07 d1 4f b9 f5 04 51>
// browser -> [8, 80, 5, 200, 156, 178, 76, 7, 172, 7, 209, 79, 185, 245, 4, 81]

// Provide your own Buffer or Array
var buffer = new Array(16);
uuid.v4('binary', buffer); // -> [8, 80, 5, 200, 156, 178, 76, 7, 172, 7, 209, 79, 185, 245, 4, 81]
var buffer = new Buffer(16);
uuid.v4('binary', buffer); // -> <Buffer 08 50 05 c8 9c b2 4c 07 ac 07 d1 4f b9 f5 04 51>

// Let node-uuid decide whether to use Buffer or Array
var buffer = new uuid.BufferClass(16);
uuid.v4('binary', buffer); // -> see above, depending on the environment

// Provide your own Buffer/Array, plus specify offset
// (e.g. here we fill an array with 3 uuids)
var buffer = new Buffer(16 * 3);
uuid.v4('binary', buffer, 0);
uuid.v4('binary', buffer, 16);
uuid.v4('binary', buffer, 32);
```

### Options

There are several options that can be passed to `v1()` and `v4()`:

```javascript
var options = {
  format: (String),    // (v1/v4) 'binary' or 'string'
  random: (Array),     // (v1/v4) array of four 32bit random #'s to use instead of rnds
  time: (Number),      // (v1) timestamp to use (in UNIX epoch, in msec)
  timesubms: (Number), // (v1) number of 100ns intervals since msec time
  clockseq: (Number),  // (v1) clockseq to use
  node: (Array)        // (v1) node. Array of hexoctets, e.g. a MAC-address
};
```

If `options` is a string it is interpreted as the `format` option.

Using the `options` parameter you can get the UUIDs that would sort first and last for a given millisecond timestamp.
This is useful whenever you need to find UUIDs that have been generated during a certain timespan.

```javascript
var now = new Date().getTime();
var first = uuid.v1({
  time: now,
  timesubms: 0,
  clockseq: 0,
  node: [0, 0, 0, 0, 0, 0]
});
var last = uuid.v1({
  time: now,
  timesubms: 9999,
  clockseq: 0x3fff, // 14bit
  node: [0xff, 0xff, 0xff, 0xff, 0xff, 0xff]
});
// first: 038ee0a0-11df-11e1-8000-000000000000
// last:  038f07af-11df-11e1-bfff-ffffffffffff
```

### Helpers

node-uuid provides helper-functions for converting UUIDs between buffer/array and string representations:

```javascript
var binary = uuid.parse('797ff043-11eb-11e1-80d6-510998755d10');
// -> <Buffer 79 7f f0 43 11 eb 11 e1 80 d6 51 09 98 75 5d 10>
var string = uuid.unparse(binary);
// -> '797ff043-11eb-11e1-80d6-510998755d10'
```


## Testing

`test/test.js` generates performance data (similar to `benchmark/benchmark.js`). It also verifies the syntax of 100K string UUIDs, and logs the distribution of hex digits found therein.  For example:

    - - - Performance Data - - -
    uuid.v4(): 1470588 uuids/second
    uuid.v4('binary'): 1041666 uuids/second
    uuid.v4('binary', buffer): 3125000 uuids/second
    uuid.v1(): 869565 uuids/second
    uuid.v1('binary'): 625000 uuids/second
    uuid.v1('binary', buffer): 1123595 uuids/second

    - - - Distribution of Hex Digits (% deviation from ideal) - - -
    0 |================================| 187378 (-0.07%)
    1 |================================| 186972 (-0.28%)
    2 |================================| 187274 (-0.12%)
    3 |================================| 187392 (-0.06%)
    4 |==================================================| 286998 (-0.17%)
    5 |================================| 187525 (0.01%)
    6 |================================| 188019 (0.28%)
    7 |================================| 187541 (0.02%)
    8 |=====================================| 212941 (0.21%)
    9 |====================================| 212308 (-0.09%)
    a |====================================| 211923 (-0.27%)
    b |=====================================| 212605 (0.05%)
    c |================================| 187608 (0.06%)
    d |================================| 188473 (0.52%)
    e |================================| 187547 (0.03%)
    f |================================| 187496 (0%)

Note that the increased values for 4 and 8-B are expected as part of the RFC4122 syntax (and are accounted for in the deviation calculation). BTW, if someone wants to do the calculation to determine what a statistically significant deviation would be, I'll gladly add that to the test.

### In browser

Open `test/test.html`

### In node.js

```
node test/test.js
node test/options.js
```

## Performance

### In node.js

node-uuid is designed to be fast. That said, the target platform is node.js, where it is screaming fast. You can find out more about how to benchmark node-uuid in `benchmark/README.md` or just have a look at our results: https://github.com/broofa/node-uuid/wiki/Benchmark

### In browser

node-uuid performance varies dramatically across browsers. For comprehensive test results, please [checkout the JSPerf tests](http://jsperf.com/node-uuid-performance).
