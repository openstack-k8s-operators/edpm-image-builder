========================
EDPM bootc image builder
========================

This repository builds and tests the CentOS Stream 9 EDPM bootc host image.
CentOS Stream 10 builds and deployment tests live in the sibling
``s2i-openstack-containers`` repository.

Install prerequisites and build the CS9 container image::

    make host-deps
    make build

The defaults use the CentOS Stream 9 bootc base and the Antelope
``current-podified`` RPM repositories. Override ``EDPM_BOOTC_REPO`` and
``EDPM_BOOTC_TAG`` to publish to your own registry.

To build with RHEL 9 subscriptions instead, customize ``rhsm.sh`` as
``rhsm-custom.sh`` and set ``RHSM_SCRIPT=rhsm-custom.sh`` and
``EDPM_BASE_IMAGE=registry.redhat.io/rhel9/rhel-bootc:latest``.

Convert the container into ``output/edpm-bootc.qcow2`` and package it for
EDPM bare-metal deployment::

    make edpm-bootc.qcow2
    make package-push

To convert an existing container image without rebuilding it, pull it into
root's container storage and run ``make output edpm-bootc.qcow2-no-build``
with the appropriate ``EDPM_BOOTC_IMAGE`` override.

To deploy the packaged image, set ``baremetalSetTemplate`` as follows::

    kind: OpenStackDataPlaneNodeSet
    spec:
      baremetalSetTemplate:
        osContainerImageUrl: <EDPM_BOOTC_REPO>:<EDPM_BOOTC_TAG>-qcow2
        osImage: edpm-bootc.qcow2
