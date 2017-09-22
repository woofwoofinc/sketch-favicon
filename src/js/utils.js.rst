Utility Functions
=================
General purpose utility functions.


Base64 Encoding/Decoding
------------------------
Decode a Base64 encoded string and return a Buffer.

.. code-block:: javascript

    const base64Decoder = require('base64-arraybuffer').decode;
    const arrayBufferToBuffer = require('arraybuffer-to-buffer');

    export function base64DecodeToBuffer(string) {
      const decoded = base64Decoder(string);

      return arrayBufferToBuffer(decoded);
    }

Encode a Buffer to Base64 string.

.. code-block:: javascript

    const base64Encoder = require('base64-arraybuffer').encode;

    export function base64EncodeToString(buffer) {
      const arrayBuffer = buffer.buffer;

      return base64Encoder(arrayBuffer);
    }


Temporary Files
---------------
Use the Objective C binding to get a temporary filename. Since this is created
in the macOS user temp directory it will be deleted automatically after three
days if it is not deleted otherwise.

UUID since don't have access to ``mkstemp`` through CocoaScript bridge.

.. code-block:: javascript

    export function getTemporaryPath(prefix, postfix) {
      const uuid = NSUUID.UUID().UUIDString();

      const filename = [ prefix || 'tmp-', uuid, postfix || '' ].join('');
      const path = `${ NSTemporaryDirectory() }${ filename }`;

      return path;
    }
