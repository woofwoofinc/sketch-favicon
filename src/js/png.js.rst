PNG Handling
============
Utilities for generating and handling PNG data.


Convert a Layer to PNG
----------------------
This works by exporting the layer to a PNG file and then rereading the PNG file
data. Follows `Aby Nimbalkar`_'s `common.js`_ in Xport_ for the export.

.. _Aby Nimbalkar: https://github.com/abynim
.. _common.js: https://github.com/abynim/Xport/blob/master/Xport.sketchplugin/Contents/Sketch/common.js
.. _Xport: https://github.com/abynim/Xport

To export the layer to PNG, first create an MSExportRequest.

.. code-block:: javascript

    const base64DecodeToBuffer = require('./utils').base64DecodeToBuffer;
    const UPNG = require('upng-js');
    const utils = require('./utils');

    export function toPng(context, layer) {

      const slice = MSExportRequest.alloc().init();

      slice.setName(layer.name);
      slice.setFormat('png');
      slice.setShouldTrim(0);
      slice.setSaveForWeb(1);

Use the dimensions of the input layer to set the export rectangle.

.. code-block:: javascript

      const bounds = layer.sketchObject.absoluteRect();

      slice.setRect(bounds.rect());

Scale the image down to 256x256 if it is larger. The favicon bitmaps cannot be
larger than 256x256 and it will speed up performance of the later steps if we
scale early.

.. code-block:: javascript

      let scale = 256.0 / bounds.width();
      if (scale > 1) {
        scale = 1;
      }

      slice.setScale(scale);

And perform the export.

.. code-block:: javascript

      const path = utils.getTemporaryPath('sketch-', '.png');
      context.document.saveArtboardOrSlice_toFile(slice, path);

To read the PNG file back, we have to access the filesystem on the Cocoa side of
the bridge which will give us an ``NSData`` object. Working with ``NSData`` is
difficult. So we move JavaScript side by converting the ``NSData`` to a Base64
string and traverse the bridge with this string.

.. code-block:: javascript

      const data = NSData.dataWithContentsOfFile(path);
      NSFileManager.defaultManager().removeItemAtPath_error(path, null);

Once back on the JavaScript side we decode the Base64 data to a Buffer.

.. code-block:: javascript

      const base64 = data.base64EncodedStringWithOptions(0);

On the JavaScript side of the bridge this is still an MOBoxedObject so coerce it
to a proper JavaScript string first.

.. code-block:: javascript

      const coerced = `${ base64 }`;

Then decode and parse the PNG data.

.. code-block:: javascript

      const buffer = base64DecodeToBuffer(coerced);

      return UPNG.decode(buffer);
    }


Resize PNG
----------
Resize the provided PNG data for the sizes to be included in the output ICO
file. All requested output sizes must be at most 256 pixels.

.. code-block:: javascript

    const resizeImageData = require('resize-image-data');

    export function resize(data, sizes) {
      return sizes.map(size => {

Since the Sketch layer to PNG conversion also scales the PNG to 256x256, we can
save doing this rescale if requested. This is very advantageous since it
eliminates the largest/slowest resize size that would be typically be used.
Resize performance is a problem since it is performed on the JavaScript side of
the bridge and results in noticeable UI thread freezing.

.. code-block:: javascript

        if (size === data.height) {
          return data;
        }

        return resizeImageData(data, size, size, 'biliniear-interpolation');
      });
    }
