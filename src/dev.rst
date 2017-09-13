.. _dev:

Development Tools Container
===========================
The project source comes with a ``dev`` directory which contains a script for
building a rkt Ubuntu container with useful development tools for Sketch
Favicon development.

To build this you must have a system with an installation of rkt and acbuild.
For macOS, the RktMachine_ project provides an xhyve-based VM running CoreOS
with installations of rkt, acbuild, docker2aci, and other useful container
tools.

.. _RktMachine: https://github.com/woofwoofinc/rktmachine


Building
--------
Build the container using the provided build script:

::

    ./dev-sketch-favicon.acbuild.sh

This will make a ``dev-sketch-favicon.oci`` in the directory. Convert this to
``dev-sketch-favicon.aci`` for installation into rkt:

::

    gunzip < dev-sketch-favicon.oci > dev-sketch-favicon.oci.tar
    docker2aci dev-sketch-favicon.oci.tar
    rm dev-sketch-favicon.oci.tar
    mv dev-sketch-favicon-latest.aci dev-sketch-favicon.aci

Install this into rkt:

::

    rkt --insecure-options=image fetch ./dev-sketch-favicon.aci

This container is intended for interactive use, so to run it with rkt use:

::

    sudo rkt run \
        --interactive \
        --volume sketch-favicon,kind=host,source=$(pwd) \
        dev-sketch-favicon \
        --mount volume=sketch-favicon,target=/sketch-favicon

The current working directory is available on the container at
``/sketch-favicon``.
