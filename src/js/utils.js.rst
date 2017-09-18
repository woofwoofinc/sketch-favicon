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
