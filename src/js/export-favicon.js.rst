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
Start by ensuring a single layer has been selected as the favicon export source.

.. code-block:: javascript

    const png = require('./png');
    const ico = require('./ico');
    const base64EncodeToString = require('./utils').base64EncodeToString;

    export default function (context) {
      const sketch = context.api();

      const layers = sketch.selectedDocument.selectedLayers;

      if (layers.length === 0) {
        sketch.alert('Select a layer to export as favicon.', 'Error');
      } else if (layers.length > 1) {
        sketch.alert('Select a single layer to export as favicon.', 'Error');
      } else {

Prompt the user for the output favicon file location.

.. code-block:: javascript

        const dialog = NSSavePanel.savePanel();
        dialog.setTitle('Export Selected Layer as Favicon');
        dialog.setNameFieldStringValue('favicon.ico');
        dialog.setAllowedFileTypes([ 'ico' ]);
        dialog.setShowsTagField(false);
        dialog.setIsExtensionHidden(false);
        dialog.setCanSelectHiddenExtension(true);

        if (dialog.runModal() == NSOKButton) {

If the Save button was selected then iterate through the (single element) array
generating favicon files.

.. code-block:: javascript

          layers.iterate(layer => {
            const pngData = png.toPng(context, layer);
            const icoData = ico.fromPng(pngData);

Finally write the data to the output file location.

.. code-block:: javascript

            const encoded = base64EncodeToString(icoData);
            const data = NSData.alloc().initWithBase64EncodedString_options(encoded, 0);

            data.writeToURL_atomically(dialog.URL(), false);
          });
        }
      }
    }
