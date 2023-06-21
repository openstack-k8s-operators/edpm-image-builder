==================
edpm-image-builder
==================

A repository to build OS images for deploying EDPM bare metal nodes using
diskimage-builder.

Image build elements
--------------------

Images are built with elements from ``diskimage-builder``,
``ironic-python-agent-builder``, and the ``dib`` directory of this repository.
These can be installed in a venv, for example from the ``edpm-image-builder``
directory::

  python3 -m venv ./venv
  source ./venv/bin/activate
  pip install -r requirements.txt
  export ELEMENTS_PATH=$(pwd)/dib:$(pwd)/venv/share/ironic-python-agent-builder/dib

edpm-hardened-uefi
------------------

The CentOS-9-Stream version of ``edpm-hardened-uefi.qcow2`` can be built with
master branch ``current-podified`` by running::

    diskimage-builder ./images/edpm-hardened-uefi-centos-9-stream.yaml

See dib/repo-setup/README.md for environment variables to control which RDO
repositories to configure.

``edpm-hardened-uefi.qcow2`` can be packaged inside a container image for
distribution by running::

    buildah bud -f ./Containerfile.image -t edpm-hardened-uefi:latest

It can then be copied out of the container image, for example into
``/path/to/images``::

    podman run --volume /path/to/images:/target:Z --rm edpm-hardened-uefi:latest

ironic-python-agent
-------------------

The CentOS-9-Stream version of ``ironic-python-agent.initramfs`` and
``ironic-python-agent.kernel`` can be built with master branch
``current-podified`` by running::

    diskimage-builder ./images/ironic-python-agent-centos-9-stream.yaml

Similarly, the rhel-9 version can be built with master branch
``current-podified`` by running::

    diskimage-builder ./images/ironic-python-agent-rhel-9.yaml

``ironic-python-agent.qcow2`` can be packaged inside a container image for
distribution by running::

    buildah bud -f ./Containerfile.ramdisk -t ironic-python-agent:latest

It can then be copied out of the container image, for example into
``/path/to/images``::

    podman run --volume /path/to/images:/target:Z --rm ironic-python-agent:latest
