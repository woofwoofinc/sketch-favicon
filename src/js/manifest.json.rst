Sketch Favicon Plugin Manifest
==============================
Sketch Favicon requires a version of Sketch more recent than 3.5.0 for the save
layer to PNG file functionality.

.. code-block:: json

    {
      "compatibleVersion": "3.5.0",
      "bundleVersion": 1,
      "name": "Sketch Favicon",
      "description": "Plugin for favicon generation.",
      "homepage": "https://github.com/woofwoofinc/sketch-favicon",
      "identifier": "dog.woofwoofinc.sketch-favicon",

Commands provided:

- Export Layer as Favicon: (Ctrl+Shift+F) Convert the selected layer to a
  ``favicon.ico`` file with provided icon sizes at 256x256, 128x128, 64x64,
  48x48, 32x32, 24x24, and 16x16.

.. code-block:: json

      "commands": [{
        "name": "Export Selected Layer as Favicon",
        "identifier": "export-favicon",
        "script": "./export-favicon.js",
        "shortcut": "ctrl shift f"
      }],

Use ``isRoot`` to add the commands to the Plugin menu directly in Sketch instead
of to a submenu as default.

.. code-block:: json

      "menu": {
        "title": "Favicon",
        "items": [
          "export-favicon"
        ],
        "isRoot": true
      }
    }
