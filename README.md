stockfish.wasm
==============

WebAssembly port of the multi variant chess engine [Fairy-Stockfish](https://github.com/ianfab/Fairy-Stockfish) .

See it in action on [PyChess](https://www.pychess.org) .

Building
--------

Assuming [em++](https://github.com/kripken/emscripten) (>= 1.39.6) is available:

```
cd src
make ARCH=wasm build -j
```

Usage
-----

Install `http-server`:

```
npm install http-server -g
```

Standing in the `src` directory type:

```
http-server
```

Then open your browser and visit [localhost:8080](http://localhost:8080) .

Requirements
------------

Uses the latest WebAssembly threading proposal.

### Chrome

* Since 79: Can allocate additional memory. The default allocation suffices
  for up to 2 threads and 16 MB hash.
* Enabled by default since 74 (desktop only)
* [Origin Trial from 70 to 75](https://developers.chrome.com/origintrials/#/view_trial/-5026017184145473535)
* Flag `chrome://flags/#enable-webassembly-threads` since 70

### Firefox

* Since Firefox 72: Structured cloning can no longer be enabled with flags.
* Since Firefox 71: Requires `javascript.options.shared_memory` and `dom.postMessage.sharedArrayBuffer.withCOOP_COEP` to be enabled in `about:flags` and these HTTP headers to be set on the top level response:

  ```
  Cross-Origin-Embedder-Policy: require-corp
  Cross-Origin-Opener-Policy: same-origin
  ```

* Firefox 68 to 70: Requires `javascript.options.shared_memory` to be enabled in `about:flags`

### Other browsers

No support.

### Feature detection

```javascript
function wasmThreadsSupported() {
  // WebAssembly 1.0
  var source = Uint8Array.of(0x0, 0x61, 0x73, 0x6d, 0x01, 0x00, 0x00, 0x00);
  if (typeof WebAssembly !== 'object' || !WebAssembly.validate(source)) return false;

  // SharedArrayBuffer
  if (typeof SharedArrayBuffer !== 'function') return false;

  // Atomics
  if (typeof Atomics !== 'object') return false;

  // Shared memory
  var mem = new WebAssembly.Memory({shared: true, initial: 8, maximum: 16});
  if (!(mem.buffer instanceof SharedArrayBuffer)) return false;

  // Structured cloning
  try {
    // You have to make sure nobody cares about these messages!
    window.postMessage(mem, '*');
  } catch (e) {
    return false;
  }

  // Growable shared memory (optional)
  try {
    mem.grow(8);
  } catch (e) {
    return false;
  }

  return true;
}
```

Current limitations
-------------------

* Hashtable: 1024 MB.
* Threads: 16.
* Can hang when UCI protocol is misused. (Do not send invalid commands or
  positions. While the engine is searching, do not change options or start
  additional searches).
* No Syzygy tablebase support.

License
-------

Thanks to the Stockfish team for sharing the engine under the GPL3.
