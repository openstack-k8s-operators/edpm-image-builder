BASE_BOOTC_IMAGE ?= quay.io/centos-bootc/centos-bootc:stream9
EDPM_BOOTC_REPO ?= quay.io/openstack-k8s-operators/edpm-bootc
EDPM_BOOTC_TAG ?= latest
EDPM_BOOTC_IMAGE ?= ${EDPM_BOOTC_REPO}:${EDPM_BOOTC_TAG}
EDPM_QCOW2_IMAGE ?= ${EDPM_BOOTC_REPO}:${EDPM_BOOTC_TAG}-qcow2
BUILDER_IMAGE ?= quay.io/centos-bootc/bootc-image-builder:latest
HOST_PACKAGES ?= podman osbuild-selinux
REPO_SETUP ?= current-podified
REPO_SETUP_BRANCH ?= master
REPO_SETUP_DISTRO_MIRROR ?=
REPO_SETUP_MIRROR ?= https://trunk.rdoproject.org
PACKAGES ?= \
            bind-utils \
            buildah \
            cephadm \
            chrony \
			cloud-init \
            crudini \
            crypto-policies-scripts \
            device-mapper-multipath \
            driverctl \
            grubby \
            iproute-tc \
            iptables-services \
            iscsi-initiator-utils \
            jq \
            lvm2 \
            nftables \
            numactl \
            openssh-server \
            openstack-selinux \
            openvswitch \
            os-net-config \
            podman \
            python3-libselinux \
            python3-pyyaml \
            rsync \
            tmpwatch \
            tuned-profiles-cpu-partitioning \
            sysstat

.ONESHELL:

.PHONY: host-deps
host-deps:
	sudo dnf install -y ${HOST_PACKAGES}

output:
	mkdir -p output

output/repo-setup: output
	ls output/repo-setup && exit 0 || true
	cd output
	curl -sL https://github.com/openstack-k8s-operators/repo-setup/archive/refs/heads/main.tar.gz | tar xvz
	cd repo-setup-main
	python3 -m venv ./venv
	source ./venv/bin/activate
	PBR_VERSION=0.0.0 python3 -m pip install ./
	cp venv/bin/repo-setup ../repo-setup

output/yum.repos.d: output/repo-setup
	ls output/yum.repos.d && exit 0 || true
	cd output
	mkdir -p yum.repos.d
	./repo-setup --output-path yum.repos.d --branch ${REPO_SETUP_BRANCH} --rdo-mirror ${REPO_SETUP_MIRROR} ${REPO_SETUP}


.PHONY: build
build: output/yum.repos.d
	sudo buildah inspect ${EDPM_BOOTC_IMAGE} > /dev/null && exit 0 || true
	CONTAINER=$$(sudo buildah from ${BASE_BOOTC_IMAGE})
	sudo buildah run $$CONTAINER -- rm -rf /etc/yum.repos.d/*.repo
	sudo buildah copy $$CONTAINER output/yum.repos.d /etc/yum.repos.d
	sudo buildah run $$CONTAINER -- dnf -y install ${PACKAGES}
	sudo buildah run $$CONTAINER -- dnf -y update
	sudo buildah commit $$CONTAINER ${EDPM_BOOTC_IMAGE}

.PHONY: edpm-bootc.qcow2
edpm-bootc.qcow2: build
	ls output/edpm-bootc.qcow2 && exit 0 || true
	sudo podman run --rm -it --privileged \
		--security-opt label=type:unconfined_t \
		-v ./output:/output \
		-v /var/lib/containers/storage:/var/lib/containers/storage \
		${BUILDER_IMAGE} \
		--type qcow2 \
		--local \
		${EDPM_BOOTC_IMAGE}
	cd output
	sudo mv qcow2/disk.qcow2 edpm-bootc.qcow2
	sudo sha256sum edpm-bootc.qcow2 > edpm-bootc.qcow2.sha256

.PHONY: package
package: edpm-bootc.qcow2
	sudo buildah inspect ${EDPM_QCOW2_IMAGE} > /dev/null && exit 0 || true
	cp ../copy_out.sh output/
	cp ../Containerfile.image output/
	cd output
	sudo buildah bud --build-arg IMAGE_NAME=edpm-bootc -f ./Containerfile.image -t ${EDPM_QCOW2_IMAGE}


.PHONY: all
all: build edpm-bootc.qcow2 package


.PHONY: clean
clean:
	sudo rm -rf output

.PHONY: realclean
realclean: clean
	sudo podman rmi --force ${EDPM_BOOTC_IMAGE} || true
	sudo podman rmi --force ${BUILDER_IMAGE} || true
	sudo podman rmi --force ${BASE_BOOTC_IMAGE} || true
	sudo podman rmi --force ${EDPM_QCOW2_IMAGE} || true
