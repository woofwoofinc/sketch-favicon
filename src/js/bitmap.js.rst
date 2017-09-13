Bitmap Handling
---------------
Utilities for generating and outputing bitmap files.


Convert a Layer to Bitmap
~~~~~~~~~~~~~~~~~~~~~~~~~
Follows `Aby Nimbalkar`_'s `common.js`_ in Xport_.

.. _Aby Nimbalkar: https://github.com/abynim
.. _common.js: https://github.com/abynim/Xport/blob/master/Xport.sketchplugin/Contents/Sketch/common.js
.. _Xport: https://github.com/abynim/Xport

To export the layer to bitmap, first create an MSExportRequest.

.. code-block:: javascript

    export function asBitmap(context, layer, scale) {

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
      log(`Exported: ${ path }`);
    }
