Releasing
=========

Release Checklist
-----------------
1. Check version number is correct in ``package.json`` and ``src/conf.py``.
2. Build release zip and upload to GitHub releases.
3. Update the ``.appcast.xml`` file.
4. Build documentation and upload to GitHub pages.
5. Update version numbers for next development cycle.


Building a Release
------------------
Prepare a release by building the distributable zip archive file to contain the
plugin binary.

First, install the project development dependencies if this hasn't been done
already.

::

    yarn

Build the ``favicon.sketchplugin`` directory contents for publication.

::

    rm -fr favicon.sketchplugin favicon.sketchplugin.zip && yarn build

Create the final compressed format.

::

    zip -r9 favicon.sketchplugin.zip favicon.sketchplugin


Preparing the GitHub Release
----------------------------
Take the SHA256 of the ``favicon.sketchplugin.zip`` binary:

::

    shasum -a 256 favicon.sketchplugin.zip

Then create a new Sketch Favicon release on the `GitHub Releases`_ page. The
release should be named the same as the version number in ``package.json``
and also be tagged with this value.

.. _GitHub Releases: https://github.com/woofwoofinc/sketch-favicon/releases

The release text should include the SHA256 value generated earlier. The
following is a template for the release text.

::

    Release <version> of Sketch Favicon.


    # Release Notes
    - Important change 1.
    - Important change 2.


    # Checksum
    - sha256: 2c972966358a79b35e5c6e73143415d86bbaa6dd2ef3a4cee0fda00cc81a053c


Update the Appcast File
-----------------------
Add an item entry in the ``.appcast.xml`` file containing information about the
release. This enables the `Sketch plugin update feature`_ to prompt users to
update the Sketch Favicon plugin when then new version is available.

.. _Sketch plugin update feature: http://developer.sketchapp.com/introduction/updating-plugins/

The item stanza for a new version should look similar to the following:

::

    <item>
      <title>Version 1.0.0</title>
      <description>
        <![CDATA[
          <ul>
          <li>Major update v1.0.0</li>
          </ul>
        ]]>
      </description>
      <enclosure
        url="https://github.com/woofwoofinc/sketch-favicon/releases/download/1.0.0/favicon.sketchplugin.zip"
        sparkle:version="1.0.0"/>
    </item>


Publishing the Documentation
----------------------------
Project documentation is published to `woofwoofinc.github.io/sketch-favicon`_
using `GitHub Pages`_.

.. _woofwoofinc.github.io/sketch-favicon: https://woofwoofinc.github.io/sketch-favicon
.. _GitHub Pages: https://pages.github.com

First build the documentation as described in :ref:`documentation`.

The GitHub configuration for this project is to serve documentation from the
``gh-pages`` branch. Rather than attempt to build a new ``gh-pages`` in the
current repository, it is simpler to copy the repository, change to ``gh-pages``
in the repository copy, and clean everything from there. This has the advantage
of not operating in the current repository too so it is non-destructive.

Create a copy of the repository.

::

    cp -r sketch-favicon sketch-favicon-gh-pages

Then change into the new repository and swap to the ``gh-pages`` branch.

::

    pushd sketch-favicon-gh-pages > /dev/null
    git checkout -b gh-pages

Clear out everything in the branch. This uses dot globing and extended glob
options to arrange deletion of everything except the .git directory.

::

    shopt -s dotglob
    shopt -s extglob
    rm -fr !(.git)

    shopt -u extglob
    shopt -u dotglob

Next, copy in the contents of ``src/_build/html`` from the main project
repository. This is the latest build of the documentation. Dot globing is
used again since the dot files in the ``src/_build/html`` directory are also
needed.

::

    shopt -s dotglob
    cp -r ../sketch-favicon/src/_build/html/* .

    shopt -u dotglob

Commit the documentation and push the ``gh-pages`` branch to GitHub.

::

    git add -A
    git commit -m "Add latest documentation."
    git push origin gh-pages

Then clean up the temporary repository.

::

    popd > /dev/null
    rm -fr sketch-favicon-gh-pages
