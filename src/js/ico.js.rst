ICO Handling
============
Utilities for generating and handling ICO data.


Create an ICO from PNG Data
---------------------------
Follows `Kevin Mårtensson`_'s `to-ico`_. See the `ICO File Format`_
specification on Wikipedia for reference.

.. _Kevin Mårtensson: https://github.com/kevva
.. _to-ico: https://github.com/kevva/to-ico
.. _ICO File Format: https://en.wikipedia.org/wiki/ICO_%28file_format%29#Outline

The ICO file header block is six bytes, or three 16-bit integers written in
little-endian order.

- The first 16-bit integer always 0.
- The second 16-bit integer is 1 in ICO files.
- The third 16-bit integer is the number of images in the ICO file.

.. code-block:: javascript

    const bufferAlloc = require('buffer-alloc');

    function createHeader(numImages) {
      const buf = bufferAlloc(6);

      buf.writeUInt16LE(0, 0);
      buf.writeUInt16LE(1, 2);
      buf.writeUInt16LE(numImages, 4);

      return buf;
    }

Each bitmap in the ICO file requires an ICONDIRENTRY block. These are 16 bytes
long and consist of the following entries in order.

- An 8-bit integer specifying the width of the bitmap in pixels. The maximum
  width of a bitmap in an ICO file is 256 pixels. Since zero pixels is an
  invalid width, the zero value instead corresponds to a bitmap that is 256
  pixels wide.
- An 8-bit integer specifying the height of the bitmap in pixels. The maximum
  height of a bitmap in an ICO file is 256 pixels. Since zero pixels is an
  invalid height, the zero value instead corresponds to a bitmap that is 256
  pixels high.
- An 8-bit integer for the number of colours in the colour palette. This will be
  zero for the ICO files here since they do not use a colour palette.
- 8-bit integer with value 0.
- 16-bit little endian colour plane value as an integer. This will be 1 for the
  ICO files here.
- 16-bit little endian integer for bits per pixel. This will be 32 for the ICO
  files here since the original Sketch layer PNG is generated in RGBA, i.e.
  8 bits for each colour channel and an 8 bit alpha.
- 32-bit little endian integer for the size of the image data block in bytes.
  This should include the size of the image data plus the 40 bytes of header
  needed for each image data block.
- 32-bit little endian integer for the offset of the image data block in bytes
  from the beginning of the ICO file.

.. code-block:: javascript

    function createDirectoryEntry(png, offset) {
      const buf = bufferAlloc(16);

      const width = png.width === 256 ? 0 : png.width;
      const height = png.height === 256 ? 0 : png.height;

      const colourPlanes = 1;
      const bitsPerPixel = 32;
      const size = 40 + png.data.length;

      buf.writeUInt8(width, 0);
      buf.writeUInt8(height, 1);
      buf.writeUInt8(0, 2);
      buf.writeUInt8(0, 3);
      buf.writeUInt16LE(colourPlanes, 4);
      buf.writeUInt16LE(bitsPerPixel, 6);
      buf.writeUInt32LE(size, 8);
      buf.writeUInt32LE(offset, 12);

      return buf;
    }

The image bitmap data is stored in `Windows BMP format`_. The file header block
is not required and the device-independent bitmap header used is the 40-byte
Windows BITMAPINFOHEADER version. (See `DIB Headers`_ from Wikipedia. Note the
40-byte variant is the second table in that second.)

- A 32-bit little endian integer containing the size in bytes of the header
  block, e.g. 40.
- A 32-bit little endian integer containing the width of the bitmap.
- A 32-bit little endian integer containing the height of the bitmap.
- A 16-bit little endian integer containing the number of colour planes. Must be
  1 here.
- A 16-bit little endian integer containing the number of bits per pixel. This
  will be 32 for RGBA PNG data here.
- A 32-bit little endian integer containing the compression method. Here we use
  no compression so the value is 0.
- A 32-bit little endian integer containing the size of the original bitmap
  data in bytes.
- A 32-bit little endian integer containing the horizontal resolution of the
  image. We leave this unspecified.
- A 32-bit little endian integer containing the vertical resolution of the
  image. We leave this unspecified.
- A 32-bit little endian integer containing the number of colors in the color
  palette. Use 0 to default to 2**n.
- A 32-bit little endian integer containing the number of important colors used,
  or 0 when every color is important; generally ignored.

.. _Windows BMP format: https://en.wikipedia.org/wiki/BMP_file_format
.. _DIB Headers: https://en.wikipedia.org/wiki/BMP_file_format#DIB_header_.28bitmap_information_header.29

.. NOTE::
   The header needs a doubled height for Windows BMP format images. See
   `ICO Icon resource structure information`_.

.. _ICO Icon resource structure information: https://en.wikipedia.org/wiki/ICO_(file_format)#Icon_resource_structure

.. code-block:: javascript

    function createBitmap(png) {
      const buf = bufferAlloc(40 + png.data.length);

      const colourPlanes = 1;
      const bytesPerPixel = 4;
      const bitsPerPixel = bytesPerPixel * 8;

      buf.writeUInt32LE(40, 0);
      buf.writeInt32LE(png.width, 4);
      buf.writeInt32LE(2 * png.height, 8);
      buf.writeUInt16LE(colourPlanes, 12);
      buf.writeUInt16LE(bitsPerPixel, 14);
      buf.writeUInt32LE(0, 16);
      buf.writeUInt32LE(png.data.length, 20);
      buf.writeInt32LE(0, 24);
      buf.writeInt32LE(0, 28);
      buf.writeUInt32LE(0, 32);
      buf.writeUInt32LE(0, 36);

`BMP pixel storage`_ is by row arrays.

.. _BMP pixel storage: https://en.wikipedia.org/wiki/BMP_file_format#Pixel_storage

.. code-block:: javascript

      const cols = png.width * bytesPerPixel;
      const rows = png.height * cols;
      const end = rows - cols;

      for (let row = 0; row < rows; row += cols) {
        for (let col = 0; col < cols; col += bytesPerPixel) {
          let pos = row + col;

          const r = png.data[pos];
          const g = png.data[pos + 1];
          const b = png.data[pos + 2];
          const a = png.data[pos + 3];

The pixels are stored in a reverse order to normal image raster scan. They start
in the lower left corner, go from left to right, and then row by row from the
bottom to the top of the image.

The ``pos`` value calculated here expands to ``(rows - row) - (cols - col)``.
Note that the ``row`` increment is ``png.width * bytesPerPixel`` and the
``col`` increment is ``bytesPerPixel`` so this ``pos`` value lines up correctly.

The output buffer is preallocated so out-of-order writing is supported with no
performance penalty.

.. code-block:: javascript

          pos = (end - row) + col;

          buf.writeUInt8(b, 40 + pos);
          buf.writeUInt8(g, 40 + pos + 1);
          buf.writeUInt8(r, 40 + pos + 2);
          buf.writeUInt8(a, 40 + pos + 3);
        }
      }

      return buf;
    }


Exports
~~~~~~~
Take the provided PNG data and start by resizing to the sizes to be included in
the output ICO file. All requested output sizes must be at most 256 pixels.

.. code-block:: javascript

    const resizeImageData = require('resize-image-data');

    export function fromPng(data, sizes) {
      const pngs = sizes.map(size => {

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

The output ICO file is constructed as a set of buffers corresponding to blocks
in the ICO file format. It saves a pass of the output buffer later if we also
track the size of the output buffer as we go.

.. code-block:: javascript

      const buffers = [ ];
      let length = 0;

The first ICO file format block is the header.

.. code-block:: javascript

      const header = createHeader(pngs.length);

      buffers.push(header);
      length += header.length;

Each image in the ICO output file requires a seperate directory entry in the
listing that follows the ICO header. The image data is included later in the
file.

Since the directory entry record needs a pointer to the image data for that
record, it is necessary to perform the offset calculation while preparing the
directory entry records. The directory entries themselves are 16 bytes, so the
first image data location will be the length of the header block plus 16 bytes
for each image directory entry.

.. code-block:: javascript

      let offset = length + (16 * pngs.length);

Create an ICO directory entry buffer for each image output size.

.. code-block:: javascript

      for (const png of pngs) {
        const dir = createDirectoryEntry(png, offset);

        buffers.push(dir);
        length += dir.length;

Update the image data offset. The next image will start at the point in the file
further along by the number of bytes in the image for the current directory
entry. An extra 40 bytes is also needed for the bitmap data block header.

.. code-block:: javascript

        offset += 40 + png.data.length;
      }

Create buffer blocks for the image data bitmaps.

.. code-block:: javascript

      for (const png of pngs) {
        const bitmap = createBitmap(png);

        buffers.push(bitmap);
        length += bitmap.length;
      }

And concatenate the ICO file block buffers to get the final ICO file data.

.. code-block:: javascript

      return Buffer.concat(buffers, length);
    }
