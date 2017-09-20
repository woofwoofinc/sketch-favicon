Building
========
The build stack uses `Node.js`_ and the Yarn_ package manager. Install these on
your system if they are not already available.

.. _Node.js: https://nodejs.org
.. _Yarn: https://yarnpkg.com

A rkt_ container build script is included in the project repository and
provides an installation which can be used to build the project also. See the
description on building and running the container in the :ref:`dev` section
of this document for details.

.. _rkt: https://coreos.com/rkt

For macOS, RktMachine_ provides a CoreOS VM which supports developing using
the rkt container system.

.. _RktMachine: https://github.com/woofwoofinc/rktmachine

Start by installing `Sketch Plugin Manager`_ using Yarn.

.. _Sketch Plugin Manager: https://github.com/skpm/skpm

::

    $ yarn global add skpm

Add the other project dependencies.

::

    $ yarn

And build the Sketch Plugin:

::

    $ yarn build

It is convenient to symlink to the compiled plugin from the Sketch Plugin
directory. This means the plugin does not have to be reimported into Sketch
every time it is built. The Sketch Plugin Manager has a command for this.

::

    skpm link favicon.sketchplugin

This doesn't work for container development since the Sketch installation and
the skpm command are not in the same execution contexts. In this case, create
the symlink manually.

::

    ln -s /path/to/favicon.sketchplugin \
      ~/Library/Application\ Support/com.bohemiancoding.sketch3/Plugins/favicon.sketchplugin

The Sketch Plugin Manager also gives the option of disabling the plugin cache
when creating the symlink. This is convenient during development. To do this
for the container development case, use the following command:

::

    defaults write ~/Library/Preferences/com.bohemiancoding.sketch3.plist AlwaysReloadScript -bool YES
