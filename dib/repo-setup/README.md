repo-setup
==========

This element installs and runs the repo-setup[1] utility. Arguments can be
managed with the following environment variables (with these defaults):

* DIB_REPO_SETUP=current-podified
* DIB_REPO_SETUP_BRANCH=master
* DIB_REPO_SETUP_DISTRO_MIRROR=
* DIB_REPO_SETUP_MIRROR=https://trunk.rdoproject.org

Ifs DIB_YUM_REPO_CONF is defined, this element will take no action. This makes it
possible to pass in repo files from the host.

[1] https://github.com/openstack-k8s-operators/repo-setup
