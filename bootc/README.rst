========================
EDPM bootc image builder
========================

To build a CentOS-9-Stream based bootc EDPM container image, run the following
command::

    export EDPM_BOOTC_REPO=quay.io/<account>/edpm-bootc
    make build
    sudo podman push $EDPM_BOOTC_REPO:latest

To convert this container image to ``output/edpm-bootc.qcow2``, run the
following command::

    make edpm-bootc.qcow2

To package ``edpm-bootc.qcow2`` inside a container image EDPM baremetal
deployment, run the following command::

    make package
    sudo podman push $EDPM_BOOTC_REPO:latest-qcow2

To deploy EDPM baremetal with this image, customize the ``baremetalSetTemplate`` with::

    kind: OpenStackDataPlaneNodeSet
    spec:
    baremetalSetTemplate:
        osContainerImageUrl: quay.io/<account>/edpm-bootc:latest-qcow2
        osImage: edpm-bootc.qcow2

