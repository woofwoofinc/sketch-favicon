User Interfaces
===============

Output Favicon Icon Sizes
-------------------------
Layout an alert dialog with the supported favicon sizes for checkbox selection.

.. code-block:: javascript

    export function faviconIconSizesDialog() {
      const dialog = NSAlert.alloc().init();
      dialog.setMessageText('Select Favicon Icon Sizes');

      dialog.addButtonWithTitle('Ok');
      dialog.addButtonWithTitle('Cancel');

Create the view container and checkbox buttons with layout position and size
configurations.

.. code-block:: javascript

      const container = NSView.alloc().initWithFrame(NSMakeRect(0, 0, 300, 100));

      const options = [
        {
          size: 16,
          frame: NSMakeRect(0, 75, 100, 23),
        },
        {
          size: 24,
          frame: NSMakeRect(0, 50, 100, 23),
        },
        {
          size: 32,
          frame: NSMakeRect(0, 25, 100, 23),
        },
        {
          size: 48,
          frame: NSMakeRect(100, 75, 100, 23),
        },
        {
          size: 64,
          frame: NSMakeRect(100, 50, 100, 23),
        },
        {
          size: 128,
          frame: NSMakeRect(100, 25, 100, 23),
        },
        {
          size: 256,
          frame: NSMakeRect(200, 75, 100, 23),
        },
      ];

      const checkboxes = options.map(option => {
        const checkbox = NSButton.alloc().initWithFrame(option.frame);
        checkbox.setState(true);
        checkbox.setButtonType(NSSwitchButton);
        checkbox.setBezelStyle(0);
        checkbox.setTitle(`${ option.size }x${ option.size }`);

        checkbox.isSelected = function isSelected() {
            return checkbox.state() !== 0;
        };

        checkbox.getSize = function getSize() {
          return option.size;
        };

        return checkbox;
      });

Extend the dialog JavaScript object with an accessor to return the current
selected sizes as an array.

.. code-block:: javascript

      dialog.getSelectedSizes = function getSelectedSizes() {
        const selected = checkboxes.filter(checkbox =>
          checkbox.isSelected()
        );

        return selected.map(checkbox =>
          checkbox.getSize()
        );
      };

Finally add each checkbox button to the container and then add the container to
the main view and return the dialog object to the caller.

.. code-block:: javascript

      checkboxes.forEach(checkbox => {
        container.addSubview(checkbox);
      });

      dialog.setAccessoryView(container);

      return dialog;
    }


Output Favicon Icon Location
----------------------------
Layout for a save location dialog.

.. code-block:: javascript

    export function faviconIconSaveDialog() {
      const dialog = NSSavePanel.savePanel();
      dialog.setTitle('Export Selected Layer as Favicon');
      dialog.setNameFieldStringValue('favicon.ico');
      dialog.setAllowedFileTypes([ 'ico' ]);
      dialog.setShowsTagField(false);
      dialog.setIsExtensionHidden(false);
      dialog.setCanSelectHiddenExtension(true);

      return dialog;
    }
