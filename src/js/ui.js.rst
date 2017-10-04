User Interfaces
===============

Output Favicon Icon Sizes
-------------------------
Layout an alert dialog with the supported favicon sizes for checkbox selection.

.. code-block:: javascript

    export function faviconIconSizesDialog(pngWidth) {
      const dialog = NSAlert.alloc().init();
      dialog.setMessageText('Select Favicon Icon Sizes');

      dialog.addButtonWithTitle('Ok');
      dialog.addButtonWithTitle('Cancel');

The main view container has variable height depending on whether a message
string is needed to notify users how to make missing sizes available.

.. code-block:: javascript

      let containerHeight = 80;
      if (pngWidth < 256) {
        containerHeight = 110;
      }

Create the view container and checkbox buttons with layout position and size
configurations.

.. code-block:: javascript

      const container = NSView.alloc().initWithFrame(NSMakeRect(0, 0, 300, containerHeight));

      const options = [
        {
          size: 16,
          frame: NSMakeRect(0, containerHeight - 30, 100, 23),
        },
        {
          size: 24,
          frame: NSMakeRect(0, containerHeight - 55, 100, 23),
        },
        {
          size: 32,
          frame: NSMakeRect(0, containerHeight - 80, 100, 23),
        },
        {
          size: 48,
          frame: NSMakeRect(100, containerHeight - 30, 100, 23),
        },
        {
          size: 64,
          frame: NSMakeRect(100, containerHeight - 55, 100, 23),
        },
        {
          size: 128,
          frame: NSMakeRect(100, containerHeight - 80, 100, 23),
        },
        {
          size: 256,
          frame: NSMakeRect(200, containerHeight - 30, 100, 23),
        },
      ];

      const checkboxes = options.map(option => {
        const checkbox = NSButton.alloc().initWithFrame(option.frame);
        checkbox.setButtonType(NSSwitchButton);
        checkbox.setBezelStyle(0);
        checkbox.setTitle(`${ option.size }x${ option.size }`);

        const enabled = option.size <= pngWidth;
        checkbox.setState(enabled);
        checkbox.setEnabled(enabled);

        checkbox.isSelected = function isSelected() {
            return checkbox.state() !== 0;
        };

        checkbox.getSize = function getSize() {
          return option.size;
        };

Finally add each checkbox button to the checkboxes subview container.

.. code-block:: javascript

        container.addSubview(checkbox);

        return checkbox;
      });

If necessary, add a user message explaining how to make disabled export sizes
available.

.. code-block:: javascript

      if (pngWidth < 256) {
        const label = NSTextField.alloc().initWithFrame(NSMakeRect(0, 0, 300, 14));

        label.setStringValue('Export a larger layer to enable missing size options.');
        label.setEditable(false);
        label.setBezeled(false);
        label.setDrawsBackground(false);

Use a small italic font for the user message.

.. code-block:: javascript

        const fontManager = NSFontManager.sharedFontManager();

        const current = label.font();
        const small = fontManager.convertFont_toSize(current, NSFont.smallSystemFontSize());
        const italic = fontManager.convertFont_toHaveTrait(small, NSItalicFontMask);

        label.setFont(italic);

        container.addSubview(label);
      }

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

Then add the container to the main view and return the dialog object to the
caller.

.. code-block:: javascript

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
