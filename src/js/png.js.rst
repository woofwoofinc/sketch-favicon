PNG Handling
------------
Utilities for generating and reading PNG files.


Convert a Layer to PNG
~~~~~~~~~~~~~~~~~~~~~~
Follows `Aby Nimbalkar`_'s `common.js`_ in Xport_.

.. _Aby Nimbalkar: https://github.com/abynim
.. _common.js: https://github.com/abynim/Xport/blob/master/Xport.sketchplugin/Contents/Sketch/common.js
.. _Xport: https://github.com/abynim/Xport

To export the layer to PNG, first create an MSExportRequest.

.. code-block:: javascript

    export function toFile(context, layer, scale) {

      const slice = MSExportRequest.alloc().init();

      slice.setName(layer.name);
      slice.setFormat('png');
      slice.setShouldTrim(0);
      slice.setSaveForWeb(1);

Use the dimensions of the input layer to set the export rectangle.

.. code-block:: javascript

      const rect = layer.sketchObject.absoluteRect().rect();

      slice.setRect(rect);

Specify an scaling if requested in the parameters.

.. code-block:: javascript

      if (typeof scale === 'undefined') {
        scale = 1;
      }

      slice.setScale(scale);

And perform the export.

.. code-block:: javascript

      const path = `${ NSTemporaryDirectory() }sketch.png`;
      context.document.saveArtboardOrSlice_toFile(slice, path);

      return path;
    }


Read a PNG File
~~~~~~~~~~~~~~~
To read a PNG file, we have to access the filesystem on the Cocoa side of the
bridge which will give us an ``NSData`` object. Working with ``NSData`` is
difficult. So we move JavaScript side by converting the ``NSData`` to a Base64
string and traverse the bridge with this string.

Once back on the JavaScript side we decode the Base64 data to a Buffer.

.. code-block:: javascript

    const base64Decoder = require('base64-arraybuffer').decode;
    const arrayBufferToBuffer = require('arraybuffer-to-buffer');
    const parsePng = require('pngparse-sync');

    export function fromFile(path) {
      const data = NSData.dataWithContentsOfFile(path);
      const base64 = data.base64EncodedStringWithOptions(0);

On the JavaScript side of the bridge this is still an MOBoxedObject so coerce it
to a proper JavaScript string first.

.. code-block:: javascript

      const coerced = `${ base64 }`;

Then decode and parse the PNG data.

.. code-block:: javascript

      const decoded = base64Decoder(coerced);
      const buffer = arrayBufferToBuffer(decoded);

      return parsePng(buffer);
    }
