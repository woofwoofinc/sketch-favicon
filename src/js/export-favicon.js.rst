Writing Sketch Plugins
======================
The entry point to a Sketch plugin is a command. The ``manifest.json`` file
lists the available commands for a plugin, their JavaScript files and how they
are listed in the Plugins menu in Sketch.

On selection of the command menu item, or by shortcut, Sketch calls the exported
function in the corresponding JavaScript file and passes a context object. This
context object is described in the
`Sketch Plugins, Scripts, and Commands Documentation`_ and includes the
following fields:

* ``command``: MSPluginCommand object
* ``document``: MSDocument object
* ``plugin``: MSPluginBundle object
* ``selection``: NSArray of MSLayer objects

.. _Sketch Plugins, Scripts, and Commands Documentation: http://developer.sketchapp.com/introduction/plugin-scripts/

The MS-prefixed objects are Sketch internals. They are not well documented yet
in the Sketch plugin development guides but the header files have been extracted
and are available in `Sketch-Headers`_. For instance, the `MSLayer header file`_
lists methods and properties for MSLayer objects in the ``selection`` array.

.. _Sketch-Headers: https://github.com/abynim/Sketch-Headers
.. _MSLayer header file: https://github.com/abynim/Sketch-Headers/blob/master/Headers/MSLayer.h

Alternatively, the context object provides an ``api`` method which returns a
JavaScript interface. This is documented starting in `Application.js`_.

.. _Application.js: http://developer.sketchapp.com/reference/api/class/api/Application.js~Application.html


Using Dependencies
------------------
The Sketch Plugin Manager uses Webpack under the hood. This bundles the plugin
scripts together with the dependencies into a single JavaScript file. This is
output to the generated Sketch plugin. Therefore dependencies provided using
the Node module system and NPM will generally work unless they rely on system
specific code or violate the macOS application sandboxing.


Export Favicon Command
======================

Validation
----------
Start with validation, ensure a single layer has been selected as the favicon
export source and that it is square and 256x256 or larger.

.. code-block:: javascript

    const ui = require('./ui');
    const png = require('./png');
    const ico = require('./ico');
    const base64EncodeToString = require('./utils').base64EncodeToString;

    export default function (context) {
      const sketch = context.api();

      const errors = [];
      const layers = sketch.selectedDocument.selectedLayers;

      if (layers.length === 0) {
        errors.push('Select a layer to export as favicon.');
      }

      if (layers.length > 1) {
        errors.push('Select a single layer to export as favicon.');
      }

      layers.iterate(layer => {
        const bounds = layer.sketchObject.absoluteRect();

        if (bounds.width() !== bounds.height()) {
          errors.push('Select a square layer to export as favicon.');
        }
        if (bounds.width() < 256) {
          errors.push('Select a layer at least 256x256 to export as favicon.');
        }
      });

None of these errors are recoverable so error out. Only need to specify a single
error cause for this.

.. code-block:: javascript

      if (errors.length > 0) {
        sketch.alert(errors[0], 'Error');
        return;
      }


Convert Selected Layer to PNG
-----------------------------
For perceived performance, it is useful to convert the selected layer to PNG
before prompting the user for output sizes and location. This moves part of the
slow calculations needed to before the main generation step.

Iterate through the (single element) array of selected layers.

.. code-block:: javascript

      layers.iterate(layer => {
        const pngData = png.toPng(context, layer);


Dialogs
-------
Prompt the user with checkbox selections for the output favicon save sizes.

.. code-block:: javascript

        const sizesDialog = ui.faviconIconSizesDialog();
        if (sizesDialog.runModal() == '1000') {
          const sizes = sizesDialog.getSelectedSizes();

For perceived performance, do the PNG file resizing here. This will happen in
the UI between the user finishing the size checkbox dialog and the save location
dialog appearing. It is not a significant improvement but it is a little less
bad from the user's perspective compared to stalling the UI thread after the
user has finished the entire interaction by selecting the save location.

.. code-block:: javascript

          const pngs = png.resize(pngData, sizes);

If the Ok button was selected, next prompt the user for the output favicon save
location.

.. code-block:: javascript

          const saveDialog = ui.faviconIconSaveDialog();
          if (saveDialog.runModal() === NSOKButton) {


Output Favicon Icon Generation
------------------------------
If the Save button was selected then generate the output favicon file.

.. code-block:: javascript

            const icoData = ico.fromPngs(pngs);

Finally write the data to the output file location.

.. code-block:: javascript

            const encoded = base64EncodeToString(icoData);
            const data = NSData.alloc().initWithBase64EncodedString_options(encoded, 0);

            data.writeToURL_atomically(saveDialog.URL(), false);
          }
        }
      });
    }
