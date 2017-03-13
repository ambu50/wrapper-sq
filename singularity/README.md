Singularity Containers
=======
----------
Singularity containers can be used to package HPC applications with all their dependencies which simplifies HPC system configuration. As a result, HPC systems can be used to run much more application without reconfiguring the system. In addition, this strengthen the security and relieve the regular security patching.

In this work, we are showing how Singularity can be used to run workloads utilizing Infiniband and MPI. Our solution consists of job launcher script and ssh client hack to support starting MPI processes in remote hosts without the need to start ssh server in each Docker containers.

**User Access and Network Filesystems**

By default, singularity launches the container under the user privilege and takes care of the passwd and the group configuration in the container. It also automatically mount some filesystems such as /tmp /home and /sys.
To bind extra filesystems the user can use “–B” switch when launching the container.

    singularity shell singularity_image/rhel72.img

**Infiniband Interconnect**
Since Singularity containers are using the host kernel, all what is needed to access Infiniband is to map the device and install the user space driver. For this, we have built a repo that contains the user space drivers and use this repo during the time of building the Singularity image. Sample image definition file is included in this repo. No special binding is needed to bind the Infiniband devices as it is done automatically.


**MPI Workload**
MPI is the most common middleware used by most HPC applications to facilitate inter-processes communication. Different MPI implementations are available such as Openmpi and Mvapich and all are following the same standard. 
MPI commonly uses ssh to instantiate MPI workload using the available network with TCP/IP support then switches to Infiniband for all following communications. This causes a problem when using Docker, especially when you don’t want to start ssh server in each of your Docker containers. To solve this problem, we have written a wrapper for ssh client and included that wrapper in our Docker images.

Our launcher script starts the master MPI process in a container then the ssh wrapper script appends Docker run command to whatever command originally passed to ssh. The ssh wrapper script also takes care of cases where the MPI process tries to ssh to the localhost to start other MPI processes locally. 


Below is an example of how to build and use an image to run simple MPI test.

<pre>
Using root do the following:

[root@myserver singularity_image]# singularity create --size 2028 rhel72.img
Creating a new image with a maximum size of 2028MiB...
Executing image create helper
Formatting image with ext3 file system
Done.

[root@myserver singularity_image]# singularity bootstrap  rhel72.img rhel72.def
Bootstrap initialization
Checking bootstrap definition
Executing Prebootstrap module
Executing Bootstrap 'yum' module
Found YUM at: /bin/yum
base                                                                                                                                                                                                                  | 2.9 kB  00:00:00     
base/primary_db                                                                                                                                                                                                       | 3.7 MB  00:00:00     
Resolving Dependencies
--> Running transaction check
---> Package coreutils.x86_64 0:8.22-15.el7 will be installed
--> Processing Dependency: rtld(GNU_HASH) for package: coreutils-8.22-15.el7.x86_64
--> Processing Dependency: ncurses for package: coreutils-8.22-15.el7.x86_64
--> Processing Dependency: librt.so.1(GLIBC_2.3.3)(64bit) for package: coreutils-8.22-15.el7.x86_64
--> Processing Dependency: libpthread.so.0(GLIBC_2.3.2)(64bit) for package: coreutils-8.22-15.el7.x86_64
--> Processing Dependency: libpthread.so.0(GLIBC_2.2.5)(64bit) for package: coreutils-8.22-15.el7.x86_64
--> Processing Dependency: libcrypto.so.10(libcrypto.so.10)(64bit) for package: coreutils-8.22-15.el7.x86_64
--> Processing Dependency: libc.so.6(GLIBC_2.7)(64bit) for package: coreutils-8.22-15.el7.x86_64
--> Processing Dependency: libattr.so.1(ATTR_1.1)(64bit) for package: coreutils-8.22-15.el7.x86_64
--> Processing Dependency: libacl.so.1(ACL_1.0)(64bit) for package: coreutils-8.22-15.el7.x86_64
--> Processing Dependency: grep for package: coreutils-8.22-15.el7.x86_64
--> Processing Dependency: gmp for package: coreutils-8.22-15.el7.x86_64
--> Processing Dependency: /sbin/install-info for package: coreutils-8.22-15.el7.x86_64
--> Processing Dependency: /sbin/install-info for package: coreutils-8.22-15.el7.x86_64
--> Processing Dependency: /bin/sh for package: coreutils-8.22-15.el7.x86_64
--> Processing Dependency: /bin/sh for package: coreutils-8.22-15.el7.x86_64
--> Processing Dependency: libselinux.so.1()(64bit) for package: coreutils-8.22-15.el7.x86_64
--> Processing Dependency: librt.so.1()(64bit) for package: coreutils-8.22-15.el7.x86_64
--> Processing Dependency: libpthread.so.0()(64bit) for package: coreutils-8.22-15.el7.x86_64
--> Processing Dependency: libgmp.so.10()(64bit) for package: coreutils-8.22-15.el7.x86_64
--> Processing Dependency: libcrypto.so.10()(64bit) for package: coreutils-8.22-15.el7.x86_64
--> Processing Dependency: libcap.so.2()(64bit) for package: coreutils-8.22-15.el7.x86_64
--> Processing Dependency: libattr.so.1()(64bit) for package: coreutils-8.22-15.el7.x86_64
--> Processing Dependency: libacl.so.1()(64bit) for package: coreutils-8.22-15.el7.x86_64
---> Package openssh-clients.x86_64 0:6.6.1p1-22.el7 will be installed
--> Processing Dependency: openssh = 6.6.1p1-22.el7 for package: openssh-clients-6.6.1p1-22.el7.x86_64
--> Processing Dependency: fipscheck-lib(x86-64) >= 1.3.0 for package: openssh-clients-6.6.1p1-22.el7.x86_64
--> Processing Dependency: libgssapi_krb5.so.2(gssapi_krb5_2_MIT)(64bit) for package: openssh-clients-6.6.1p1-22.el7.x86_64
--> Processing Dependency: libz.so.1()(64bit) for package: openssh-clients-6.6.1p1-22.el7.x86_64
--> Processing Dependency: libtinfo.so.5()(64bit) for package: openssh-clients-6.6.1p1-22.el7.x86_64
--> Processing Dependency: libldap-2.4.so.2()(64bit) for package: openssh-clients-6.6.1p1-22.el7.x86_64
--> Processing Dependency: liblber-2.4.so.2()(64bit) for package: openssh-clients-6.6.1p1-22.el7.x86_64
--> Processing Dependency: libkrb5.so.3()(64bit) for package: openssh-clients-6.6.1p1-22.el7.x86_64
--> Processing Dependency: libk5crypto.so.3()(64bit) for package: openssh-clients-6.6.1p1-22.el7.x86_64
--> Processing Dependency: libgssapi_krb5.so.2()(64bit) for package: openssh-clients-6.6.1p1-22.el7.x86_64
--> Processing Dependency: libfipscheck.so.1()(64bit) for package: openssh-clients-6.6.1p1-22.el7.x86_64
--> Processing Dependency: libedit.so.0()(64bit) for package: openssh-clients-6.6.1p1-22.el7.x86_64
--> Processing Dependency: libcom_err.so.2()(64bit) for package: openssh-clients-6.6.1p1-22.el7.x86_64
---> Package procps-ng.x86_64 0:3.3.10-3.el7 will be installed
--> Processing Dependency: libsystemd-login.so.0(LIBSYSTEMD_LOGIN_38)(64bit) for package: procps-ng-3.3.10-3.el7.x86_64
--> Processing Dependency: libsystemd-login.so.0(LIBSYSTEMD_LOGIN_31)(64bit) for package: procps-ng-3.3.10-3.el7.x86_64
--> Processing Dependency: libsystemd-login.so.0(LIBSYSTEMD_LOGIN_205)(64bit) for package: procps-ng-3.3.10-3.el7.x86_64
--> Processing Dependency: libsystemd-login.so.0(LIBSYSTEMD_LOGIN_202)(64bit) for package: procps-ng-3.3.10-3.el7.x86_64
--> Processing Dependency: libsystemd-login.so.0()(64bit) for package: procps-ng-3.3.10-3.el7.x86_64
---> Package redhat-release-server.x86_64 0:7.2-9.el7 will be installed
---> Package util-linux.x86_64 0:2.23.2-26.el7 will be installed
--> Processing Dependency: libuuid = 2.23.2-26.el7 for package: util-linux-2.23.2-26.el7.x86_64
--> Processing Dependency: libmount = 2.23.2-26.el7 for package: util-linux-2.23.2-26.el7.x86_64
--> Processing Dependency: libblkid = 2.23.2-26.el7 for package: util-linux-2.23.2-26.el7.x86_64
--> Processing Dependency: pam >= 1.1.3-7 for package: util-linux-2.23.2-26.el7.x86_64
--> Processing Dependency: audit-libs >= 1.0.6 for package: util-linux-2.23.2-26.el7.x86_64
--> Processing Dependency: libuuid.so.1(UUID_1.0)(64bit) for package: util-linux-2.23.2-26.el7.x86_64
--> Processing Dependency: libutempter.so.0(UTEMPTER_1.1)(64bit) for package: util-linux-2.23.2-26.el7.x86_64
--> Processing Dependency: libpam_misc.so.0(LIBPAM_MISC_1.0)(64bit) for package: util-linux-2.23.2-26.el7.x86_64
--> Processing Dependency: libpam.so.0(LIBPAM_1.0)(64bit) for package: util-linux-2.23.2-26.el7.x86_64
--> Processing Dependency: libmount.so.1(MOUNT_2.25)(64bit) for package: util-linux-2.23.2-26.el7.x86_64
--> Processing Dependency: libmount.so.1(MOUNT_2.23)(64bit) for package: util-linux-2.23.2-26.el7.x86_64
--> Processing Dependency: libmount.so.1(MOUNT_2.22)(64bit) for package: util-linux-2.23.2-26.el7.x86_64
--> Processing Dependency: libmount.so.1(MOUNT_2.21)(64bit) for package: util-linux-2.23.2-26.el7.x86_64
--> Processing Dependency: libmount.so.1(MOUNT_2.20)(64bit) for package: util-linux-2.23.2-26.el7.x86_64
--> Processing Dependency: libmount.so.1(MOUNT_2.19)(64bit) for package: util-linux-2.23.2-26.el7.x86_64
--> Processing Dependency: libblkid.so.1(BLKID_2.21)(64bit) for package: util-linux-2.23.2-26.el7.x86_64
--> Processing Dependency: libblkid.so.1(BLKID_2.20)(64bit) for package: util-linux-2.23.2-26.el7.x86_64
--> Processing Dependency: libblkid.so.1(BLKID_2.18)(64bit) for package: util-linux-2.23.2-26.el7.x86_64
--> Processing Dependency: libblkid.so.1(BLKID_2.17)(64bit) for package: util-linux-2.23.2-26.el7.x86_64
--> Processing Dependency: libblkid.so.1(BLKID_2.15)(64bit) for package: util-linux-2.23.2-26.el7.x86_64
--> Processing Dependency: libblkid.so.1(BLKID_1.0)(64bit) for package: util-linux-2.23.2-26.el7.x86_64
--> Processing Dependency: /etc/pam.d/system-auth for package: util-linux-2.23.2-26.el7.x86_64
--> Processing Dependency: libuuid.so.1()(64bit) for package: util-linux-2.23.2-26.el7.x86_64
--> Processing Dependency: libutempter.so.0()(64bit) for package: util-linux-2.23.2-26.el7.x86_64
--> Processing Dependency: libuser.so.1()(64bit) for package: util-linux-2.23.2-26.el7.x86_64
--> Processing Dependency: libpam_misc.so.0()(64bit) for package: util-linux-2.23.2-26.el7.x86_64
--> Processing Dependency: libpam.so.0()(64bit) for package: util-linux-2.23.2-26.el7.x86_64
--> Processing Dependency: libmount.so.1()(64bit) for package: util-linux-2.23.2-26.el7.x86_64
--> Processing Dependency: libcap-ng.so.0()(64bit) for package: util-linux-2.23.2-26.el7.x86_64
--> Processing Dependency: libblkid.so.1()(64bit) for package: util-linux-2.23.2-26.el7.x86_64
--> Processing Dependency: libaudit.so.1()(64bit) for package: util-linux-2.23.2-26.el7.x86_64
---> Package yum.noarch 0:3.4.3-132.el7 will be installed
--> Processing Dependency: python(abi) = 2.7 for package: yum-3.4.3-132.el7.noarch
--> Processing Dependency: yum-metadata-parser >= 1.1.0 for package: yum-3.4.3-132.el7.noarch
--> Processing Dependency: rpm >= 4.4.2 for package: yum-3.4.3-132.el7.noarch
--> Processing Dependency: python-urlgrabber >= 3.9.0-8 for package: yum-3.4.3-132.el7.noarch
--> Processing Dependency: python >= 2.4 for package: yum-3.4.3-132.el7.noarch
--> Processing Dependency: rpm-python for package: yum-3.4.3-132.el7.noarch
--> Processing Dependency: pyxattr for package: yum-3.4.3-132.el7.noarch
--> Processing Dependency: python-sqlite for package: yum-3.4.3-132.el7.noarch
--> Processing Dependency: python-iniparse for package: yum-3.4.3-132.el7.noarch
--> Processing Dependency: pyliblzma for package: yum-3.4.3-132.el7.noarch
--> Processing Dependency: pygpgme for package: yum-3.4.3-132.el7.noarch
--> Processing Dependency: diffutils for package: yum-3.4.3-132.el7.noarch
--> Processing Dependency: cpio for package: yum-3.4.3-132.el7.noarch
--> Processing Dependency: /usr/bin/python for package: yum-3.4.3-132.el7.noarch
---> Package yum-utils.noarch 0:1.1.31-34.el7 will be installed
--> Processing Dependency: python-kitchen for package: yum-utils-1.1.31-34.el7.noarch
--> Running transaction check
---> Package audit-libs.x86_64 0:2.4.1-5.el7 will be installed
---> Package bash.x86_64 0:4.2.46-19.el7 will be installed
---> Package cpio.x86_64 0:2.11-24.el7 will be installed
---> Package diffutils.x86_64 0:3.3-4.el7 will be installed
---> Package fipscheck-lib.x86_64 0:1.4.1-5.el7 will be installed
--> Processing Dependency: /usr/bin/fipscheck for package: fipscheck-lib-1.4.1-5.el7.x86_64
---> Package glibc.x86_64 0:2.17-105.el7 will be installed
--> Processing Dependency: glibc-common = 2.17-105.el7 for package: glibc-2.17-105.el7.x86_64
--> Processing Dependency: libgcc for package: glibc-2.17-105.el7.x86_64
--> Processing Dependency: libfreebl3.so(NSSRAWHASH_3.12.3)(64bit) for package: glibc-2.17-105.el7.x86_64
--> Processing Dependency: basesystem for package: glibc-2.17-105.el7.x86_64
--> Processing Dependency: libfreebl3.so()(64bit) for package: glibc-2.17-105.el7.x86_64
---> Package gmp.x86_64 1:6.0.0-11.el7 will be installed
--> Processing Dependency: libstdc++.so.6(GLIBCXX_3.4.11)(64bit) for package: 1:gmp-6.0.0-11.el7.x86_64
--> Processing Dependency: libstdc++.so.6(GLIBCXX_3.4)(64bit) for package: 1:gmp-6.0.0-11.el7.x86_64
--> Processing Dependency: libstdc++.so.6(CXXABI_1.3)(64bit) for package: 1:gmp-6.0.0-11.el7.x86_64
--> Processing Dependency: libstdc++.so.6()(64bit) for package: 1:gmp-6.0.0-11.el7.x86_64
---> Package grep.x86_64 0:2.20-2.el7 will be installed
--> Processing Dependency: libpcre.so.1()(64bit) for package: grep-2.20-2.el7.x86_64
---> Package info.x86_64 0:5.1-4.el7 will be installed
---> Package krb5-libs.x86_64 0:1.13.2-10.el7 will be installed
--> Processing Dependency: keyutils-libs >= 1.5.8 for package: krb5-libs-1.13.2-10.el7.x86_64
--> Processing Dependency: sed for package: krb5-libs-1.13.2-10.el7.x86_64
--> Processing Dependency: libkeyutils.so.1(KEYUTILS_1.5)(64bit) for package: krb5-libs-1.13.2-10.el7.x86_64
--> Processing Dependency: libkeyutils.so.1(KEYUTILS_1.0)(64bit) for package: krb5-libs-1.13.2-10.el7.x86_64
--> Processing Dependency: libkeyutils.so.1(KEYUTILS_0.3)(64bit) for package: krb5-libs-1.13.2-10.el7.x86_64
--> Processing Dependency: gawk for package: krb5-libs-1.13.2-10.el7.x86_64
--> Processing Dependency: libverto.so.1()(64bit) for package: krb5-libs-1.13.2-10.el7.x86_64
--> Processing Dependency: libkeyutils.so.1()(64bit) for package: krb5-libs-1.13.2-10.el7.x86_64
---> Package libacl.x86_64 0:2.2.51-12.el7 will be installed
---> Package libattr.x86_64 0:2.4.46-12.el7 will be installed
---> Package libblkid.x86_64 0:2.23.2-26.el7 will be installed
---> Package libcap.x86_64 0:2.22-8.el7 will be installed
---> Package libcap-ng.x86_64 0:0.7.5-4.el7 will be installed
---> Package libcom_err.x86_64 0:1.42.9-7.el7 will be installed
---> Package libedit.x86_64 0:3.0-12.20121213cvs.el7 will be installed
---> Package libmount.x86_64 0:2.23.2-26.el7 will be installed
---> Package libselinux.x86_64 0:2.2.2-6.el7 will be installed
--> Processing Dependency: libsepol >= 2.1.9-1 for package: libselinux-2.2.2-6.el7.x86_64
--> Processing Dependency: liblzma.so.5(XZ_5.0)(64bit) for package: libselinux-2.2.2-6.el7.x86_64
--> Processing Dependency: liblzma.so.5()(64bit) for package: libselinux-2.2.2-6.el7.x86_64
---> Package libuser.x86_64 0:0.60-7.el7_1 will be installed
--> Processing Dependency: libpopt.so.0(LIBPOPT_0)(64bit) for package: libuser-0.60-7.el7_1.x86_64
--> Processing Dependency: libpopt.so.0()(64bit) for package: libuser-0.60-7.el7_1.x86_64
--> Processing Dependency: libgobject-2.0.so.0()(64bit) for package: libuser-0.60-7.el7_1.x86_64
--> Processing Dependency: libgmodule-2.0.so.0()(64bit) for package: libuser-0.60-7.el7_1.x86_64
--> Processing Dependency: libglib-2.0.so.0()(64bit) for package: libuser-0.60-7.el7_1.x86_64
---> Package libutempter.x86_64 0:1.1.6-4.el7 will be installed
--> Processing Dependency: shadow-utils for package: libutempter-1.1.6-4.el7.x86_64
---> Package libuuid.x86_64 0:2.23.2-26.el7 will be installed
---> Package ncurses.x86_64 0:5.9-13.20130511.el7 will be installed
---> Package ncurses-libs.x86_64 0:5.9-13.20130511.el7 will be installed
--> Processing Dependency: ncurses-base = 5.9-13.20130511.el7 for package: ncurses-libs-5.9-13.20130511.el7.x86_64
---> Package openldap.x86_64 0:2.4.40-8.el7 will be installed
--> Processing Dependency: nss-tools for package: openldap-2.4.40-8.el7.x86_64
--> Processing Dependency: libssl3.so(NSS_3.7.4)(64bit) for package: openldap-2.4.40-8.el7.x86_64
--> Processing Dependency: libssl3.so(NSS_3.4)(64bit) for package: openldap-2.4.40-8.el7.x86_64
--> Processing Dependency: libssl3.so(NSS_3.2)(64bit) for package: openldap-2.4.40-8.el7.x86_64
--> Processing Dependency: libssl3.so(NSS_3.14)(64bit) for package: openldap-2.4.40-8.el7.x86_64
--> Processing Dependency: libnss3.so(NSS_3.9.3)(64bit) for package: openldap-2.4.40-8.el7.x86_64
--> Processing Dependency: libnss3.so(NSS_3.9.2)(64bit) for package: openldap-2.4.40-8.el7.x86_64
--> Processing Dependency: libnss3.so(NSS_3.8)(64bit) for package: openldap-2.4.40-8.el7.x86_64
--> Processing Dependency: libnss3.so(NSS_3.6)(64bit) for package: openldap-2.4.40-8.el7.x86_64
--> Processing Dependency: libnss3.so(NSS_3.4)(64bit) for package: openldap-2.4.40-8.el7.x86_64
--> Processing Dependency: libnss3.so(NSS_3.3)(64bit) for package: openldap-2.4.40-8.el7.x86_64
--> Processing Dependency: libnss3.so(NSS_3.2)(64bit) for package: openldap-2.4.40-8.el7.x86_64
--> Processing Dependency: libnss3.so(NSS_3.12.9)(64bit) for package: openldap-2.4.40-8.el7.x86_64
--> Processing Dependency: libnss3.so(NSS_3.12.5)(64bit) for package: openldap-2.4.40-8.el7.x86_64
--> Processing Dependency: libnss3.so(NSS_3.12.1)(64bit) for package: openldap-2.4.40-8.el7.x86_64
--> Processing Dependency: libnss3.so(NSS_3.12)(64bit) for package: openldap-2.4.40-8.el7.x86_64
--> Processing Dependency: libnss3.so(NSS_3.11.1)(64bit) for package: openldap-2.4.40-8.el7.x86_64
--> Processing Dependency: libnss3.so(NSS_3.11)(64bit) for package: openldap-2.4.40-8.el7.x86_64
--> Processing Dependency: libnss3.so(NSS_3.10)(64bit) for package: openldap-2.4.40-8.el7.x86_64
--> Processing Dependency: findutils for package: openldap-2.4.40-8.el7.x86_64
--> Processing Dependency: libssl3.so()(64bit) for package: openldap-2.4.40-8.el7.x86_64
--> Processing Dependency: libsmime3.so()(64bit) for package: openldap-2.4.40-8.el7.x86_64
--> Processing Dependency: libsasl2.so.3()(64bit) for package: openldap-2.4.40-8.el7.x86_64
--> Processing Dependency: libplds4.so()(64bit) for package: openldap-2.4.40-8.el7.x86_64
--> Processing Dependency: libplc4.so()(64bit) for package: openldap-2.4.40-8.el7.x86_64
--> Processing Dependency: libnssutil3.so()(64bit) for package: openldap-2.4.40-8.el7.x86_64
--> Processing Dependency: libnss3.so()(64bit) for package: openldap-2.4.40-8.el7.x86_64
--> Processing Dependency: libnspr4.so()(64bit) for package: openldap-2.4.40-8.el7.x86_64
---> Package openssh.x86_64 0:6.6.1p1-22.el7 will be installed
---> Package openssl-libs.x86_64 1:1.0.1e-42.el7_1.9 will be installed
--> Processing Dependency: ca-certificates >= 2008-5 for package: 1:openssl-libs-1.0.1e-42.el7_1.9.x86_64
---> Package pam.x86_64 0:1.1.8-12.el7_1.1 will be installed
--> Processing Dependency: libpwquality >= 0.9.9 for package: pam-1.1.8-12.el7_1.1.x86_64
--> Processing Dependency: cracklib-dicts >= 2.8 for package: pam-1.1.8-12.el7_1.1.x86_64
--> Processing Dependency: libdb-5.3.so()(64bit) for package: pam-1.1.8-12.el7_1.1.x86_64
--> Processing Dependency: libcrack.so.2()(64bit) for package: pam-1.1.8-12.el7_1.1.x86_64
---> Package pygpgme.x86_64 0:0.3-9.el7 will be installed
--> Processing Dependency: libgpgme.so.11(GPGME_1.1)(64bit) for package: pygpgme-0.3-9.el7.x86_64
--> Processing Dependency: libgpgme.so.11(GPGME_1.0)(64bit) for package: pygpgme-0.3-9.el7.x86_64
--> Processing Dependency: libpython2.7.so.1.0()(64bit) for package: pygpgme-0.3-9.el7.x86_64
--> Processing Dependency: libgpgme.so.11()(64bit) for package: pygpgme-0.3-9.el7.x86_64
---> Package pyliblzma.x86_64 0:0.5.3-11.el7 will be installed
---> Package python.x86_64 0:2.7.5-34.el7 will be installed
---> Package python-iniparse.noarch 0:0.4-9.el7 will be installed
---> Package python-kitchen.noarch 0:1.1.1-5.el7 will be installed
--> Processing Dependency: python-chardet for package: python-kitchen-1.1.1-5.el7.noarch
---> Package python-urlgrabber.noarch 0:3.10-7.el7 will be installed
--> Processing Dependency: python-pycurl for package: python-urlgrabber-3.10-7.el7.noarch
---> Package pyxattr.x86_64 0:0.5.1-5.el7 will be installed
---> Package rpm.x86_64 0:4.11.3-17.el7 will be installed
--> Processing Dependency: curl for package: rpm-4.11.3-17.el7.x86_64
--> Processing Dependency: /usr/bin/db_stat for package: rpm-4.11.3-17.el7.x86_64
--> Processing Dependency: librpmio.so.3()(64bit) for package: rpm-4.11.3-17.el7.x86_64
--> Processing Dependency: librpm.so.3()(64bit) for package: rpm-4.11.3-17.el7.x86_64
--> Processing Dependency: liblua-5.1.so()(64bit) for package: rpm-4.11.3-17.el7.x86_64
--> Processing Dependency: libelf.so.1()(64bit) for package: rpm-4.11.3-17.el7.x86_64
--> Processing Dependency: libbz2.so.1()(64bit) for package: rpm-4.11.3-17.el7.x86_64
---> Package rpm-python.x86_64 0:4.11.3-17.el7 will be installed
--> Processing Dependency: librpmsign.so.1()(64bit) for package: rpm-python-4.11.3-17.el7.x86_64
--> Processing Dependency: librpmbuild.so.3()(64bit) for package: rpm-python-4.11.3-17.el7.x86_64
--> Processing Dependency: libmagic.so.1()(64bit) for package: rpm-python-4.11.3-17.el7.x86_64
---> Package systemd-libs.x86_64 0:219-19.el7 will be installed
--> Processing Dependency: libgcrypt.so.11(GCRYPT_1.2)(64bit) for package: systemd-libs-219-19.el7.x86_64
--> Processing Dependency: libgpg-error.so.0()(64bit) for package: systemd-libs-219-19.el7.x86_64
--> Processing Dependency: libgcrypt.so.11()(64bit) for package: systemd-libs-219-19.el7.x86_64
--> Processing Dependency: libdw.so.1()(64bit) for package: systemd-libs-219-19.el7.x86_64
---> Package yum-metadata-parser.x86_64 0:1.1.4-10.el7 will be installed
--> Processing Dependency: libxml2.so.2(LIBXML2_2.4.30)(64bit) for package: yum-metadata-parser-1.1.4-10.el7.x86_64
--> Processing Dependency: libxml2.so.2()(64bit) for package: yum-metadata-parser-1.1.4-10.el7.x86_64
--> Processing Dependency: libsqlite3.so.0()(64bit) for package: yum-metadata-parser-1.1.4-10.el7.x86_64
---> Package zlib.x86_64 0:1.2.7-15.el7 will be installed
--> Running transaction check
---> Package basesystem.noarch 0:10.0-7.el7 will be installed
--> Processing Dependency: setup for package: basesystem-10.0-7.el7.noarch
--> Processing Dependency: filesystem for package: basesystem-10.0-7.el7.noarch
---> Package bzip2-libs.x86_64 0:1.0.6-13.el7 will be installed
---> Package ca-certificates.noarch 0:2015.2.4-71.el7 will be installed
--> Processing Dependency: p11-kit-trust >= 0.17.3 for package: ca-certificates-2015.2.4-71.el7.noarch
--> Processing Dependency: p11-kit >= 0.17.3 for package: ca-certificates-2015.2.4-71.el7.noarch
---> Package cracklib.x86_64 0:2.9.0-11.el7 will be installed
--> Processing Dependency: gzip for package: cracklib-2.9.0-11.el7.x86_64
---> Package cracklib-dicts.x86_64 0:2.9.0-11.el7 will be installed
---> Package curl.x86_64 0:7.29.0-25.el7 will be installed
--> Processing Dependency: libcurl = 7.29.0-25.el7 for package: curl-7.29.0-25.el7.x86_64
--> Processing Dependency: libcurl.so.4()(64bit) for package: curl-7.29.0-25.el7.x86_64
---> Package cyrus-sasl-lib.x86_64 0:2.1.26-19.2.el7 will be installed
---> Package elfutils-libelf.x86_64 0:0.163-3.el7 will be installed
---> Package elfutils-libs.x86_64 0:0.163-3.el7 will be installed
---> Package file-libs.x86_64 0:5.11-31.el7 will be installed
---> Package findutils.x86_64 1:4.5.11-5.el7 will be installed
---> Package fipscheck.x86_64 0:1.4.1-5.el7 will be installed
---> Package gawk.x86_64 0:4.0.2-4.el7 will be installed
---> Package glib2.x86_64 0:2.42.2-5.el7 will be installed
--> Processing Dependency: shared-mime-info for package: glib2-2.42.2-5.el7.x86_64
--> Processing Dependency: libffi.so.6()(64bit) for package: glib2-2.42.2-5.el7.x86_64
---> Package glibc-common.x86_64 0:2.17-105.el7 will be installed
--> Processing Dependency: tzdata >= 2003a for package: glibc-common-2.17-105.el7.x86_64
---> Package gpgme.x86_64 0:1.3.2-5.el7 will be installed
--> Processing Dependency: libassuan.so.0(LIBASSUAN_1.0)(64bit) for package: gpgme-1.3.2-5.el7.x86_64
--> Processing Dependency: gnupg2 for package: gpgme-1.3.2-5.el7.x86_64
--> Processing Dependency: libassuan.so.0()(64bit) for package: gpgme-1.3.2-5.el7.x86_64
---> Package keyutils-libs.x86_64 0:1.5.8-3.el7 will be installed
---> Package libdb.x86_64 0:5.3.21-19.el7 will be installed
---> Package libdb-utils.x86_64 0:5.3.21-19.el7 will be installed
---> Package libgcc.x86_64 0:4.8.5-4.el7 will be installed
---> Package libgcrypt.x86_64 0:1.5.3-12.el7_1.1 will be installed
---> Package libgpg-error.x86_64 0:1.12-3.el7 will be installed
---> Package libpwquality.x86_64 0:1.2.3-4.el7 will be installed
---> Package libsepol.x86_64 0:2.1.9-3.el7 will be installed
---> Package libstdc++.x86_64 0:4.8.5-4.el7 will be installed
---> Package libverto.x86_64 0:0.2.5-4.el7 will be installed
---> Package libxml2.x86_64 0:2.9.1-5.el7_1.2 will be installed
---> Package lua.x86_64 0:5.1.4-14.el7 will be installed
--> Processing Dependency: libreadline.so.6()(64bit) for package: lua-5.1.4-14.el7.x86_64
---> Package ncurses-base.noarch 0:5.9-13.20130511.el7 will be installed
---> Package nspr.x86_64 0:4.10.8-2.el7_1 will be installed
---> Package nss.x86_64 0:3.19.1-18.el7 will be installed
--> Processing Dependency: nss-softokn(x86-64) >= 3.16.2.3-13 for package: nss-3.19.1-18.el7.x86_64
--> Processing Dependency: nss-system-init for package: nss-3.19.1-18.el7.x86_64
--> Processing Dependency: /usr/sbin/update-alternatives for package: nss-3.19.1-18.el7.x86_64
--> Processing Dependency: /usr/sbin/update-alternatives for package: nss-3.19.1-18.el7.x86_64
--> Processing Dependency: libsoftokn3.so()(64bit) for package: nss-3.19.1-18.el7.x86_64
--> Processing Dependency: libnssdbm3.so()(64bit) for package: nss-3.19.1-18.el7.x86_64
---> Package nss-softokn-freebl.x86_64 0:3.16.2.3-13.el7_1 will be installed
---> Package nss-tools.x86_64 0:3.19.1-18.el7 will be installed
---> Package nss-util.x86_64 0:3.19.1-4.el7_1 will be installed
---> Package pcre.x86_64 0:8.32-15.el7 will be installed
---> Package popt.x86_64 0:1.13-16.el7 will be installed
---> Package python-chardet.noarch 0:2.2.1-1.el7_1 will be installed
---> Package python-libs.x86_64 0:2.7.5-34.el7 will be installed
--> Processing Dependency: expat >= 2.1.0 for package: python-libs-2.7.5-34.el7.x86_64
--> Processing Dependency: libgdbm_compat.so.4()(64bit) for package: python-libs-2.7.5-34.el7.x86_64
--> Processing Dependency: libgdbm.so.4()(64bit) for package: python-libs-2.7.5-34.el7.x86_64
--> Processing Dependency: libexpat.so.1()(64bit) for package: python-libs-2.7.5-34.el7.x86_64
---> Package python-pycurl.x86_64 0:7.19.0-17.el7 will be installed
---> Package rpm-build-libs.x86_64 0:4.11.3-17.el7 will be installed
---> Package rpm-libs.x86_64 0:4.11.3-17.el7 will be installed
---> Package sed.x86_64 0:4.2.2-5.el7 will be installed
---> Package shadow-utils.x86_64 2:4.1.5.1-18.el7 will be installed
--> Processing Dependency: libsemanage.so.1(LIBSEMANAGE_1.0)(64bit) for package: 2:shadow-utils-4.1.5.1-18.el7.x86_64
--> Processing Dependency: libsemanage.so.1()(64bit) for package: 2:shadow-utils-4.1.5.1-18.el7.x86_64
---> Package sqlite.x86_64 0:3.7.17-8.el7 will be installed
---> Package xz-libs.x86_64 0:5.1.2-12alpha.el7 will be installed
--> Running transaction check
---> Package chkconfig.x86_64 0:1.3.61-5.el7 will be installed
---> Package expat.x86_64 0:2.1.0-8.el7 will be installed
---> Package filesystem.x86_64 0:3.2-20.el7 will be installed
---> Package gdbm.x86_64 0:1.10-8.el7 will be installed
---> Package gnupg2.x86_64 0:2.0.22-3.el7 will be installed
--> Processing Dependency: pinentry for package: gnupg2-2.0.22-3.el7.x86_64
--> Processing Dependency: libpth.so.20()(64bit) for package: gnupg2-2.0.22-3.el7.x86_64
---> Package gzip.x86_64 0:1.5-8.el7 will be installed
---> Package libassuan.x86_64 0:2.1.0-3.el7 will be installed
---> Package libcurl.x86_64 0:7.29.0-25.el7 will be installed
--> Processing Dependency: libssh2(x86-64) >= 1.4.3 for package: libcurl-7.29.0-25.el7.x86_64
--> Processing Dependency: libidn.so.11(LIBIDN_1.0)(64bit) for package: libcurl-7.29.0-25.el7.x86_64
--> Processing Dependency: libssh2.so.1()(64bit) for package: libcurl-7.29.0-25.el7.x86_64
--> Processing Dependency: libidn.so.11()(64bit) for package: libcurl-7.29.0-25.el7.x86_64
---> Package libffi.x86_64 0:3.0.13-16.el7 will be installed
---> Package libsemanage.x86_64 0:2.1.10-18.el7 will be installed
--> Processing Dependency: libustr-1.0.so.1(USTR_1.0.1)(64bit) for package: libsemanage-2.1.10-18.el7.x86_64
--> Processing Dependency: libustr-1.0.so.1(USTR_1.0)(64bit) for package: libsemanage-2.1.10-18.el7.x86_64
--> Processing Dependency: libustr-1.0.so.1()(64bit) for package: libsemanage-2.1.10-18.el7.x86_64
---> Package nss-softokn.x86_64 0:3.16.2.3-13.el7_1 will be installed
---> Package nss-sysinit.x86_64 0:3.19.1-18.el7 will be installed
---> Package p11-kit.x86_64 0:0.20.7-3.el7 will be installed
---> Package p11-kit-trust.x86_64 0:0.20.7-3.el7 will be installed
--> Processing Dependency: libtasn1.so.6(LIBTASN1_0_3)(64bit) for package: p11-kit-trust-0.20.7-3.el7.x86_64
--> Processing Dependency: libtasn1.so.6()(64bit) for package: p11-kit-trust-0.20.7-3.el7.x86_64
---> Package readline.x86_64 0:6.2-9.el7 will be installed
---> Package setup.noarch 0:2.8.71-6.el7 will be installed
---> Package shared-mime-info.x86_64 0:1.1-9.el7 will be installed
--> Processing Dependency: /usr/bin/pkg-config for package: shared-mime-info-1.1-9.el7.x86_64
---> Package tzdata.noarch 0:2015g-1.el7 will be installed
--> Running transaction check
---> Package libidn.x86_64 0:1.28-4.el7 will be installed
---> Package libssh2.x86_64 0:1.4.3-10.el7 will be installed
---> Package libtasn1.x86_64 0:3.8-2.el7 will be installed
---> Package pinentry.x86_64 0:0.8.1-14.el7 will be installed
---> Package pkgconfig.x86_64 1:0.27.1-4.el7 will be installed
---> Package pth.x86_64 0:2.0.7-23.el7 will be installed
---> Package ustr.x86_64 0:1.0.4-16.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=============================================================================================================================================================================================================================================
 Package                                                          Arch                                              Version                                                            Repository                                       Size
=============================================================================================================================================================================================================================================
Installing:
 coreutils                                                        x86_64                                            8.22-15.el7                                                        base                                            3.2 M
 openssh-clients                                                  x86_64                                            6.6.1p1-22.el7                                                     base                                            638 k
 procps-ng                                                        x86_64                                            3.3.10-3.el7                                                       base                                            286 k
 redhat-release-server                                            x86_64                                            7.2-9.el7                                                          base                                             27 k
 util-linux                                                       x86_64                                            2.23.2-26.el7                                                      base                                            1.9 M
 yum                                                              noarch                                            3.4.3-132.el7                                                      base                                            1.2 M
 yum-utils                                                        noarch                                            1.1.31-34.el7                                                      base                                            113 k
Installing for dependencies:
 audit-libs                                                       x86_64                                            2.4.1-5.el7                                                        base                                             80 k
 basesystem                                                       noarch                                            10.0-7.el7                                                         base                                            4.9 k
 bash                                                             x86_64                                            4.2.46-19.el7                                                      base                                            1.0 M
 bzip2-libs                                                       x86_64                                            1.0.6-13.el7                                                       base                                             40 k
 ca-certificates                                                  noarch                                            2015.2.4-71.el7                                                    base                                            442 k
 chkconfig                                                        x86_64                                            1.3.61-5.el7                                                       base                                            173 k
 cpio                                                             x86_64                                            2.11-24.el7                                                        base                                            210 k
 cracklib                                                         x86_64                                            2.9.0-11.el7                                                       base                                             80 k
 cracklib-dicts                                                   x86_64                                            2.9.0-11.el7                                                       base                                            3.6 M
 curl                                                             x86_64                                            7.29.0-25.el7                                                      base                                            263 k
 cyrus-sasl-lib                                                   x86_64                                            2.1.26-19.2.el7                                                    base                                            155 k
 diffutils                                                        x86_64                                            3.3-4.el7                                                          base                                            322 k
 elfutils-libelf                                                  x86_64                                            0.163-3.el7                                                        base                                            200 k
 elfutils-libs                                                    x86_64                                            0.163-3.el7                                                        base                                            260 k
 expat                                                            x86_64                                            2.1.0-8.el7                                                        base                                             80 k
 file-libs                                                        x86_64                                            5.11-31.el7                                                        base                                            339 k
 filesystem                                                       x86_64                                            3.2-20.el7                                                         base                                            1.0 M
 findutils                                                        x86_64                                            1:4.5.11-5.el7                                                     base                                            559 k
 fipscheck                                                        x86_64                                            1.4.1-5.el7                                                        base                                             21 k
 fipscheck-lib                                                    x86_64                                            1.4.1-5.el7                                                        base                                             11 k
 gawk                                                             x86_64                                            4.0.2-4.el7                                                        base                                            873 k
 gdbm                                                             x86_64                                            1.10-8.el7                                                         base                                             70 k
 glib2                                                            x86_64                                            2.42.2-5.el7                                                       base                                            2.2 M
 glibc                                                            x86_64                                            2.17-105.el7                                                       base                                            3.6 M
 glibc-common                                                     x86_64                                            2.17-105.el7                                                       base                                             11 M
 gmp                                                              x86_64                                            1:6.0.0-11.el7                                                     base                                            280 k
 gnupg2                                                           x86_64                                            2.0.22-3.el7                                                       base                                            1.5 M
 gpgme                                                            x86_64                                            1.3.2-5.el7                                                        base                                            146 k
 grep                                                             x86_64                                            2.20-2.el7                                                         base                                            344 k
 gzip                                                             x86_64                                            1.5-8.el7                                                          base                                            129 k
 info                                                             x86_64                                            5.1-4.el7                                                          base                                            233 k
 keyutils-libs                                                    x86_64                                            1.5.8-3.el7                                                        base                                             25 k
 krb5-libs                                                        x86_64                                            1.13.2-10.el7                                                      base                                            837 k
 libacl                                                           x86_64                                            2.2.51-12.el7                                                      base                                             27 k
 libassuan                                                        x86_64                                            2.1.0-3.el7                                                        base                                             63 k
 libattr                                                          x86_64                                            2.4.46-12.el7                                                      base                                             18 k
 libblkid                                                         x86_64                                            2.23.2-26.el7                                                      base                                            166 k
 libcap                                                           x86_64                                            2.22-8.el7                                                         base                                             47 k
 libcap-ng                                                        x86_64                                            0.7.5-4.el7                                                        base                                             25 k
 libcom_err                                                       x86_64                                            1.42.9-7.el7                                                       base                                             40 k
 libcurl                                                          x86_64                                            7.29.0-25.el7                                                      base                                            215 k
 libdb                                                            x86_64                                            5.3.21-19.el7                                                      base                                            718 k
 libdb-utils                                                      x86_64                                            5.3.21-19.el7                                                      base                                            102 k
 libedit                                                          x86_64                                            3.0-12.20121213cvs.el7                                             base                                             92 k
 libffi                                                           x86_64                                            3.0.13-16.el7                                                      base                                             30 k
 libgcc                                                           x86_64                                            4.8.5-4.el7                                                        base                                             95 k
 libgcrypt                                                        x86_64                                            1.5.3-12.el7_1.1                                                   base                                            263 k
 libgpg-error                                                     x86_64                                            1.12-3.el7                                                         base                                             87 k
 libidn                                                           x86_64                                            1.28-4.el7                                                         base                                            209 k
 libmount                                                         x86_64                                            2.23.2-26.el7                                                      base                                            168 k
 libpwquality                                                     x86_64                                            1.2.3-4.el7                                                        base                                             84 k
 libselinux                                                       x86_64                                            2.2.2-6.el7                                                        base                                            145 k
 libsemanage                                                      x86_64                                            2.1.10-18.el7                                                      base                                            123 k
 libsepol                                                         x86_64                                            2.1.9-3.el7                                                        base                                            154 k
 libssh2                                                          x86_64                                            1.4.3-10.el7                                                       base                                            134 k
 libstdc++                                                        x86_64                                            4.8.5-4.el7                                                        base                                            298 k
 libtasn1                                                         x86_64                                            3.8-2.el7                                                          base                                            320 k
 libuser                                                          x86_64                                            0.60-7.el7_1                                                       base                                            398 k
 libutempter                                                      x86_64                                            1.1.6-4.el7                                                        base                                             25 k
 libuuid                                                          x86_64                                            2.23.2-26.el7                                                      base                                             74 k
 libverto                                                         x86_64                                            0.2.5-4.el7                                                        base                                             16 k
 libxml2                                                          x86_64                                            2.9.1-5.el7_1.2                                                    base                                            664 k
 lua                                                              x86_64                                            5.1.4-14.el7                                                       base                                            201 k
 ncurses                                                          x86_64                                            5.9-13.20130511.el7                                                base                                            304 k
 ncurses-base                                                     noarch                                            5.9-13.20130511.el7                                                base                                             68 k
 ncurses-libs                                                     x86_64                                            5.9-13.20130511.el7                                                base                                            316 k
 nspr                                                             x86_64                                            4.10.8-2.el7_1                                                     base                                            126 k
 nss                                                              x86_64                                            3.19.1-18.el7                                                      base                                            852 k
 nss-softokn                                                      x86_64                                            3.16.2.3-13.el7_1                                                  base                                            305 k
 nss-softokn-freebl                                               x86_64                                            3.16.2.3-13.el7_1                                                  base                                            204 k
 nss-sysinit                                                      x86_64                                            3.19.1-18.el7                                                      base                                             54 k
 nss-tools                                                        x86_64                                            3.19.1-18.el7                                                      base                                            484 k
 nss-util                                                         x86_64                                            3.19.1-4.el7_1                                                     base                                             71 k
 openldap                                                         x86_64                                            2.4.40-8.el7                                                       base                                            348 k
 openssh                                                          x86_64                                            6.6.1p1-22.el7                                                     base                                            435 k
 openssl-libs                                                     x86_64                                            1:1.0.1e-42.el7_1.9                                                base                                            949 k
 p11-kit                                                          x86_64                                            0.20.7-3.el7                                                       base                                            107 k
 p11-kit-trust                                                    x86_64                                            0.20.7-3.el7                                                       base                                            126 k
 pam                                                              x86_64                                            1.1.8-12.el7_1.1                                                   base                                            714 k
 pcre                                                             x86_64                                            8.32-15.el7                                                        base                                            418 k
 pinentry                                                         x86_64                                            0.8.1-14.el7                                                       base                                             72 k
 pkgconfig                                                        x86_64                                            1:0.27.1-4.el7                                                     base                                             54 k
 popt                                                             x86_64                                            1.13-16.el7                                                        base                                             42 k
 pth                                                              x86_64                                            2.0.7-23.el7                                                       base                                             89 k
 pygpgme                                                          x86_64                                            0.3-9.el7                                                          base                                             63 k
 pyliblzma                                                        x86_64                                            0.5.3-11.el7                                                       base                                             47 k
 python                                                           x86_64                                            2.7.5-34.el7                                                       base                                             88 k
 python-chardet                                                   noarch                                            2.2.1-1.el7_1                                                      base                                            227 k
 python-iniparse                                                  noarch                                            0.4-9.el7                                                          base                                             39 k
 python-kitchen                                                   noarch                                            1.1.1-5.el7                                                        base                                            266 k
 python-libs                                                      x86_64                                            2.7.5-34.el7                                                       base                                            5.6 M
 python-pycurl                                                    x86_64                                            7.19.0-17.el7                                                      base                                             80 k
 python-urlgrabber                                                noarch                                            3.10-7.el7                                                         base                                            107 k
 pyxattr                                                          x86_64                                            0.5.1-5.el7                                                        base                                             28 k
 readline                                                         x86_64                                            6.2-9.el7                                                          base                                            192 k
 rpm                                                              x86_64                                            4.11.3-17.el7                                                      base                                            1.2 M
 rpm-build-libs                                                   x86_64                                            4.11.3-17.el7                                                      base                                            103 k
 rpm-libs                                                         x86_64                                            4.11.3-17.el7                                                      base                                            272 k
 rpm-python                                                       x86_64                                            4.11.3-17.el7                                                      base                                             79 k
 sed                                                              x86_64                                            4.2.2-5.el7                                                        base                                            231 k
 setup                                                            noarch                                            2.8.71-6.el7                                                       base                                            165 k
 shadow-utils                                                     x86_64                                            2:4.1.5.1-18.el7                                                   base                                            1.1 M
 shared-mime-info                                                 x86_64                                            1.1-9.el7                                                          base                                            371 k
 sqlite                                                           x86_64                                            3.7.17-8.el7                                                       base                                            394 k
 systemd-libs                                                     x86_64                                            219-19.el7                                                         base                                            356 k
 tzdata                                                           noarch                                            2015g-1.el7                                                        base                                            431 k
 ustr                                                             x86_64                                            1.0.4-16.el7                                                       base                                             92 k
 xz-libs                                                          x86_64                                            5.1.2-12alpha.el7                                                  base                                            102 k
 yum-metadata-parser                                              x86_64                                            1.1.4-10.el7                                                       base                                             28 k
 zlib                                                             x86_64                                            1.2.7-15.el7                                                       base                                             90 k

Transaction Summary
=============================================================================================================================================================================================================================================
Install  7 Packages (+110 Dependent packages)

Total download size: 60 M
Installed size: 281 M
Downloading packages:
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                                                                         47 MB/s |  60 MB  00:00:01     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : libgcc-4.8.5-4.el7.x86_64                                                                                                                                                                                               1/117 
  Installing : redhat-release-server-7.2-9.el7.x86_64                                                                                                                                                                                  2/117 
  Installing : setup-2.8.71-6.el7.noarch                                                                                                                                                                                               3/117 
  Installing : filesystem-3.2-20.el7.x86_64                                                                                                                                                                                            4/117 
  Installing : basesystem-10.0-7.el7.noarch                                                                                                                                                                                            5/117 
  Installing : ncurses-base-5.9-13.20130511.el7.noarch                                                                                                                                                                                 6/117 
  Installing : tzdata-2015g-1.el7.noarch                                                                                                                                                                                               7/117 
  Installing : nss-softokn-freebl-3.16.2.3-13.el7_1.x86_64                                                                                                                                                                             8/117 
  Installing : glibc-common-2.17-105.el7.x86_64                                                                                                                                                                                        9/117 
  Installing : glibc-2.17-105.el7.x86_64                                                                                                                                                                                              10/117 
  Installing : xz-libs-5.1.2-12alpha.el7.x86_64                                                                                                                                                                                       11/117 
  Installing : libstdc++-4.8.5-4.el7.x86_64                                                                                                                                                                                           12/117 
  Installing : ncurses-libs-5.9-13.20130511.el7.x86_64                                                                                                                                                                                13/117 
  Installing : bash-4.2.46-19.el7.x86_64                                                                                                                                                                                              14/117 
  Installing : libsepol-2.1.9-3.el7.x86_64                                                                                                                                                                                            15/117 
  Installing : pcre-8.32-15.el7.x86_64                                                                                                                                                                                                16/117 
  Installing : libselinux-2.2.2-6.el7.x86_64                                                                                                                                                                                          17/117 
  Installing : zlib-1.2.7-15.el7.x86_64                                                                                                                                                                                               18/117 
  Installing : info-5.1-4.el7.x86_64                                                                                                                                                                                                  19/117 
  Installing : bzip2-libs-1.0.6-13.el7.x86_64                                                                                                                                                                                         20/117 
  Installing : libdb-5.3.21-19.el7.x86_64                                                                                                                                                                                             21/117 
  Installing : nspr-4.10.8-2.el7_1.x86_64                                                                                                                                                                                             22/117 
  Installing : nss-util-3.19.1-4.el7_1.x86_64                                                                                                                                                                                         23/117 
  Installing : popt-1.13-16.el7.x86_64                                                                                                                                                                                                24/117 
  Installing : elfutils-libelf-0.163-3.el7.x86_64                                                                                                                                                                                     25/117 
  Installing : audit-libs-2.4.1-5.el7.x86_64                                                                                                                                                                                          26/117 
  Installing : libgpg-error-1.12-3.el7.x86_64                                                                                                                                                                                         27/117 
  Installing : libattr-2.4.46-12.el7.x86_64                                                                                                                                                                                           28/117 
  Installing : libacl-2.2.51-12.el7.x86_64                                                                                                                                                                                            29/117 
  Installing : libcap-2.22-8.el7.x86_64                                                                                                                                                                                               30/117 
  Installing : readline-6.2-9.el7.x86_64                                                                                                                                                                                              31/117 
  Installing : lua-5.1.4-14.el7.x86_64                                                                                                                                                                                                32/117 
  Installing : libcom_err-1.42.9-7.el7.x86_64                                                                                                                                                                                         33/117 
  Installing : libffi-3.0.13-16.el7.x86_64                                                                                                                                                                                            34/117 
  Installing : sqlite-3.7.17-8.el7.x86_64                                                                                                                                                                                             35/117 
  Installing : chkconfig-1.3.61-5.el7.x86_64                                                                                                                                                                                          36/117 
  Installing : libuuid-2.23.2-26.el7.x86_64                                                                                                                                                                                           37/117 
  Installing : nss-softokn-3.16.2.3-13.el7_1.x86_64                                                                                                                                                                                   38/117 
  Installing : p11-kit-0.20.7-3.el7.x86_64                                                                                                                                                                                            39/117 
  Installing : libassuan-2.1.0-3.el7.x86_64                                                                                                                                                                                           40/117 
  Installing : libgcrypt-1.5.3-12.el7_1.1.x86_64                                                                                                                                                                                      41/117 
  Installing : grep-2.20-2.el7.x86_64                                                                                                                                                                                                 42/117 
  Installing : sed-4.2.2-5.el7.x86_64                                                                                                                                                                                                 43/117 
  Installing : file-libs-5.11-31.el7.x86_64                                                                                                                                                                                           44/117 
  Installing : libxml2-2.9.1-5.el7_1.2.x86_64                                                                                                                                                                                         45/117 
  Installing : keyutils-libs-1.5.8-3.el7.x86_64                                                                                                                                                                                       46/117 
  Installing : glib2-2.42.2-5.el7.x86_64                                                                                                                                                                                              47/117 
  Installing : 1:pkgconfig-0.27.1-4.el7.x86_64                                                                                                                                                                                        48/117 
  Installing : shared-mime-info-1.1-9.el7.x86_64                                                                                                                                                                                      49/117 
  Installing : pinentry-0.8.1-14.el7.x86_64                                                                                                                                                                                           50/117 
  Installing : elfutils-libs-0.163-3.el7.x86_64                                                                                                                                                                                       51/117 
  Installing : cyrus-sasl-lib-2.1.26-19.2.el7.x86_64                                                                                                                                                                                  52/117 
  Installing : libdb-utils-5.3.21-19.el7.x86_64                                                                                                                                                                                       53/117 
  Installing : gawk-4.0.2-4.el7.x86_64                                                                                                                                                                                                54/117 
  Installing : libidn-1.28-4.el7.x86_64                                                                                                                                                                                               55/117 
  Installing : cpio-2.11-24.el7.x86_64                                                                                                                                                                                                56/117 
  Installing : 1:findutils-4.5.11-5.el7.x86_64                                                                                                                                                                                        57/117 
  Installing : diffutils-3.3-4.el7.x86_64                                                                                                                                                                                             58/117 
  Installing : libedit-3.0-12.20121213cvs.el7.x86_64                                                                                                                                                                                  59/117 
  Installing : ncurses-5.9-13.20130511.el7.x86_64                                                                                                                                                                                     60/117 
  Installing : 1:gmp-6.0.0-11.el7.x86_64                                                                                                                                                                                              61/117 
  Installing : pth-2.0.7-23.el7.x86_64                                                                                                                                                                                                62/117 
  Installing : ustr-1.0.4-16.el7.x86_64                                                                                                                                                                                               63/117 
  Installing : libsemanage-2.1.10-18.el7.x86_64                                                                                                                                                                                       64/117 
  Installing : libverto-0.2.5-4.el7.x86_64                                                                                                                                                                                            65/117 
  Installing : expat-2.1.0-8.el7.x86_64                                                                                                                                                                                               66/117 
  Installing : gdbm-1.10-8.el7.x86_64                                                                                                                                                                                                 67/117 
  Installing : libcap-ng-0.7.5-4.el7.x86_64                                                                                                                                                                                           68/117 
  Installing : libtasn1-3.8-2.el7.x86_64                                                                                                                                                                                              69/117 
  Installing : p11-kit-trust-0.20.7-3.el7.x86_64                                                                                                                                                                                      70/117 
  Installing : krb5-libs-1.13.2-10.el7.x86_64                                                                                                                                                                                         71/117 
  Installing : 1:openssl-libs-1.0.1e-42.el7_1.9.x86_64                                                                                                                                                                                72/117 
  Installing : coreutils-8.22-15.el7.x86_64                                                                                                                                                                                           73/117 
  Installing : ca-certificates-2015.2.4-71.el7.noarch                                                                                                                                                                                 74/117 
  Installing : python-libs-2.7.5-34.el7.x86_64                                                                                                                                                                                        75/117 
  Installing : python-2.7.5-34.el7.x86_64                                                                                                                                                                                             76/117 
  Installing : libblkid-2.23.2-26.el7.x86_64                                                                                                                                                                                          77/117 
  Installing : libmount-2.23.2-26.el7.x86_64                                                                                                                                                                                          78/117 
  Installing : python-iniparse-0.4-9.el7.noarch                                                                                                                                                                                       79/117 
  Installing : yum-metadata-parser-1.1.4-10.el7.x86_64                                                                                                                                                                                80/117 
  Installing : pyliblzma-0.5.3-11.el7.x86_64                                                                                                                                                                                          81/117 
  Installing : pyxattr-0.5.1-5.el7.x86_64                                                                                                                                                                                             82/117 
  Installing : python-chardet-2.2.1-1.el7_1.noarch                                                                                                                                                                                    83/117 
  Installing : python-kitchen-1.1.1-5.el7.noarch                                                                                                                                                                                      84/117 
  Installing : 2:shadow-utils-4.1.5.1-18.el7.x86_64                                                                                                                                                                                   85/117 
  Installing : libutempter-1.1.6-4.el7.x86_64                                                                                                                                                                                         86/117 
  Installing : nss-sysinit-3.19.1-18.el7.x86_64                                                                                                                                                                                       87/117 
  Installing : nss-3.19.1-18.el7.x86_64                                                                                                                                                                                               88/117 
  Installing : nss-tools-3.19.1-18.el7.x86_64                                                                                                                                                                                         89/117 
  Installing : gzip-1.5-8.el7.x86_64                                                                                                                                                                                                  90/117 
  Installing : cracklib-2.9.0-11.el7.x86_64                                                                                                                                                                                           91/117 
  Installing : cracklib-dicts-2.9.0-11.el7.x86_64                                                                                                                                                                                     92/117 
  Installing : pam-1.1.8-12.el7_1.1.x86_64                                                                                                                                                                                            93/117 
  Installing : libpwquality-1.2.3-4.el7.x86_64                                                                                                                                                                                        94/117 
  Installing : systemd-libs-219-19.el7.x86_64                                                                                                                                                                                         95/117 
  Installing : fipscheck-lib-1.4.1-5.el7.x86_64                                                                                                                                                                                       96/117 
  Installing : fipscheck-1.4.1-5.el7.x86_64                                                                                                                                                                                           97/117 
  Installing : libssh2-1.4.3-10.el7.x86_64                                                                                                                                                                                            98/117 
  Installing : libcurl-7.29.0-25.el7.x86_64                                                                                                                                                                                           99/117 
  Installing : curl-7.29.0-25.el7.x86_64                                                                                                                                                                                             100/117 
  Installing : rpm-libs-4.11.3-17.el7.x86_64                                                                                                                                                                                         101/117 
  Installing : rpm-4.11.3-17.el7.x86_64                                                                                                                                                                                              102/117 
  Installing : openldap-2.4.40-8.el7.x86_64                                                                                                                                                                                          103/117 
  Installing : gnupg2-2.0.22-3.el7.x86_64                                                                                                                                                                                            104/117 
  Installing : rpm-build-libs-4.11.3-17.el7.x86_64                                                                                                                                                                                   105/117 
  Installing : rpm-python-4.11.3-17.el7.x86_64                                                                                                                                                                                       106/117 
  Installing : gpgme-1.3.2-5.el7.x86_64                                                                                                                                                                                              107/117 
  Installing : pygpgme-0.3-9.el7.x86_64                                                                                                                                                                                              108/117 
  Installing : libuser-0.60-7.el7_1.x86_64                                                                                                                                                                                           109/117 
  Installing : util-linux-2.23.2-26.el7.x86_64                                                                                                                                                                                       110/117 
  Installing : openssh-6.6.1p1-22.el7.x86_64                                                                                                                                                                                         111/117 
  Installing : python-pycurl-7.19.0-17.el7.x86_64                                                                                                                                                                                    112/117 
  Installing : python-urlgrabber-3.10-7.el7.noarch                                                                                                                                                                                   113/117 
  Installing : yum-3.4.3-132.el7.noarch                                                                                                                                                                                              114/117 
  Installing : yum-utils-1.1.31-34.el7.noarch                                                                                                                                                                                        115/117 
  Installing : openssh-clients-6.6.1p1-22.el7.x86_64                                                                                                                                                                                 116/117 
  Installing : procps-ng-3.3.10-3.el7.x86_64                                                                                                                                                                                         117/117 
  Verifying  : pygpgme-0.3-9.el7.x86_64                                                                                                                                                                                                1/117 
  Verifying  : 1:pkgconfig-0.27.1-4.el7.x86_64                                                                                                                                                                                         2/117 
  Verifying  : openssh-6.6.1p1-22.el7.x86_64                                                                                                                                                                                           3/117 
  Verifying  : glibc-common-2.17-105.el7.x86_64                                                                                                                                                                                        4/117 
  Verifying  : pth-2.0.7-23.el7.x86_64                                                                                                                                                                                                 5/117 
  Verifying  : pam-1.1.8-12.el7_1.1.x86_64                                                                                                                                                                                             6/117 
  Verifying  : 1:openssl-libs-1.0.1e-42.el7_1.9.x86_64                                                                                                                                                                                 7/117 
  Verifying  : systemd-libs-219-19.el7.x86_64                                                                                                                                                                                          8/117 
  Verifying  : libstdc++-4.8.5-4.el7.x86_64                                                                                                                                                                                            9/117 
  Verifying  : libblkid-2.23.2-26.el7.x86_64                                                                                                                                                                                          10/117 
  Verifying  : gawk-4.0.2-4.el7.x86_64                                                                                                                                                                                                11/117 
  Verifying  : 2:shadow-utils-4.1.5.1-18.el7.x86_64                                                                                                                                                                                   12/117 
  Verifying  : curl-7.29.0-25.el7.x86_64                                                                                                                                                                                              13/117 
  Verifying  : fipscheck-lib-1.4.1-5.el7.x86_64                                                                                                                                                                                       14/117 
  Verifying  : xz-libs-5.1.2-12alpha.el7.x86_64                                                                                                                                                                                       15/117 
  Verifying  : libuser-0.60-7.el7_1.x86_64                                                                                                                                                                                            16/117 
  Verifying  : shared-mime-info-1.1-9.el7.x86_64                                                                                                                                                                                      17/117 
  Verifying  : cracklib-dicts-2.9.0-11.el7.x86_64                                                                                                                                                                                     18/117 
  Verifying  : libacl-2.2.51-12.el7.x86_64                                                                                                                                                                                            19/117 
  Verifying  : readline-6.2-9.el7.x86_64                                                                                                                                                                                              20/117 
  Verifying  : libedit-3.0-12.20121213cvs.el7.x86_64                                                                                                                                                                                  21/117 
  Verifying  : libassuan-2.1.0-3.el7.x86_64                                                                                                                                                                                           22/117 
  Verifying  : ustr-1.0.4-16.el7.x86_64                                                                                                                                                                                               23/117 
  Verifying  : libselinux-2.2.2-6.el7.x86_64                                                                                                                                                                                          24/117 
  Verifying  : sqlite-3.7.17-8.el7.x86_64                                                                                                                                                                                             25/117 
  Verifying  : libutempter-1.1.6-4.el7.x86_64                                                                                                                                                                                         26/117 
  Verifying  : nss-util-3.19.1-4.el7_1.x86_64                                                                                                                                                                                         27/117 
  Verifying  : elfutils-libs-0.163-3.el7.x86_64                                                                                                                                                                                       28/117 
  Verifying  : libidn-1.28-4.el7.x86_64                                                                                                                                                                                               29/117 
  Verifying  : grep-2.20-2.el7.x86_64                                                                                                                                                                                                 30/117 
  Verifying  : nss-tools-3.19.1-18.el7.x86_64                                                                                                                                                                                         31/117 
  Verifying  : procps-ng-3.3.10-3.el7.x86_64                                                                                                                                                                                          32/117 
  Verifying  : nss-softokn-3.16.2.3-13.el7_1.x86_64                                                                                                                                                                                   33/117 
  Verifying  : elfutils-libelf-0.163-3.el7.x86_64                                                                                                                                                                                     34/117 
  Verifying  : python-iniparse-0.4-9.el7.noarch                                                                                                                                                                                       35/117 
  Verifying  : pinentry-0.8.1-14.el7.x86_64                                                                                                                                                                                           36/117 
  Verifying  : libuuid-2.23.2-26.el7.x86_64                                                                                                                                                                                           37/117 
  Verifying  : libsemanage-2.1.10-18.el7.x86_64                                                                                                                                                                                       38/117 
  Verifying  : tzdata-2015g-1.el7.noarch                                                                                                                                                                                              39/117 
  Verifying  : libverto-0.2.5-4.el7.x86_64                                                                                                                                                                                            40/117 
  Verifying  : nss-sysinit-3.19.1-18.el7.x86_64                                                                                                                                                                                       41/117 
  Verifying  : libpwquality-1.2.3-4.el7.x86_64                                                                                                                                                                                        42/117 
  Verifying  : 1:gmp-6.0.0-11.el7.x86_64                                                                                                                                                                                              43/117 
  Verifying  : bzip2-libs-1.0.6-13.el7.x86_64                                                                                                                                                                                         44/117 
  Verifying  : libcom_err-1.42.9-7.el7.x86_64                                                                                                                                                                                         45/117 
  Verifying  : glibc-2.17-105.el7.x86_64                                                                                                                                                                                              46/117 
  Verifying  : yum-metadata-parser-1.1.4-10.el7.x86_64                                                                                                                                                                                47/117 
  Verifying  : libcurl-7.29.0-25.el7.x86_64                                                                                                                                                                                           48/117 
  Verifying  : ca-certificates-2015.2.4-71.el7.noarch                                                                                                                                                                                 49/117 
  Verifying  : util-linux-2.23.2-26.el7.x86_64                                                                                                                                                                                        50/117 
  Verifying  : cracklib-2.9.0-11.el7.x86_64                                                                                                                                                                                           51/117 
  Verifying  : pyliblzma-0.5.3-11.el7.x86_64                                                                                                                                                                                          52/117 
  Verifying  : python-kitchen-1.1.1-5.el7.noarch                                                                                                                                                                                      53/117 
  Verifying  : glib2-2.42.2-5.el7.x86_64                                                                                                                                                                                              54/117 
  Verifying  : python-pycurl-7.19.0-17.el7.x86_64                                                                                                                                                                                     55/117 
  Verifying  : rpm-python-4.11.3-17.el7.x86_64                                                                                                                                                                                        56/117 
  Verifying  : yum-3.4.3-132.el7.noarch                                                                                                                                                                                               57/117 
  Verifying  : popt-1.13-16.el7.x86_64                                                                                                                                                                                                58/117 
  Verifying  : bash-4.2.46-19.el7.x86_64                                                                                                                                                                                              59/117 
  Verifying  : python-libs-2.7.5-34.el7.x86_64                                                                                                                                                                                        60/117 
  Verifying  : expat-2.1.0-8.el7.x86_64                                                                                                                                                                                               61/117 
  Verifying  : cyrus-sasl-lib-2.1.26-19.2.el7.x86_64                                                                                                                                                                                  62/117 
  Verifying  : pcre-8.32-15.el7.x86_64                                                                                                                                                                                                63/117 
  Verifying  : p11-kit-trust-0.20.7-3.el7.x86_64                                                                                                                                                                                      64/117 
  Verifying  : gzip-1.5-8.el7.x86_64                                                                                                                                                                                                  65/117 
  Verifying  : libgcrypt-1.5.3-12.el7_1.1.x86_64                                                                                                                                                                                      66/117 
  Verifying  : gdbm-1.10-8.el7.x86_64                                                                                                                                                                                                 67/117 
  Verifying  : libcap-2.22-8.el7.x86_64                                                                                                                                                                                               68/117 
  Verifying  : rpm-4.11.3-17.el7.x86_64                                                                                                                                                                                               69/117 
  Verifying  : libmount-2.23.2-26.el7.x86_64                                                                                                                                                                                          70/117 
  Verifying  : lua-5.1.4-14.el7.x86_64                                                                                                                                                                                                71/117 
  Verifying  : audit-libs-2.4.1-5.el7.x86_64                                                                                                                                                                                          72/117 
  Verifying  : yum-utils-1.1.31-34.el7.noarch                                                                                                                                                                                         73/117 
  Verifying  : python-2.7.5-34.el7.x86_64                                                                                                                                                                                             74/117 
  Verifying  : rpm-build-libs-4.11.3-17.el7.x86_64                                                                                                                                                                                    75/117 
  Verifying  : basesystem-10.0-7.el7.noarch                                                                                                                                                                                           76/117 
  Verifying  : fipscheck-1.4.1-5.el7.x86_64                                                                                                                                                                                           77/117 
  Verifying  : filesystem-3.2-20.el7.x86_64                                                                                                                                                                                           78/117 
  Verifying  : setup-2.8.71-6.el7.noarch                                                                                                                                                                                              79/117 
  Verifying  : gpgme-1.3.2-5.el7.x86_64                                                                                                                                                                                               80/117 
  Verifying  : coreutils-8.22-15.el7.x86_64                                                                                                                                                                                           81/117 
  Verifying  : p11-kit-0.20.7-3.el7.x86_64                                                                                                                                                                                            82/117 
  Verifying  : libdb-5.3.21-19.el7.x86_64                                                                                                                                                                                             83/117 
  Verifying  : libffi-3.0.13-16.el7.x86_64                                                                                                                                                                                            84/117 
  Verifying  : keyutils-libs-1.5.8-3.el7.x86_64                                                                                                                                                                                       85/117 
  Verifying  : nss-3.19.1-18.el7.x86_64                                                                                                                                                                                               86/117 
  Verifying  : file-libs-5.11-31.el7.x86_64                                                                                                                                                                                           87/117 
  Verifying  : ncurses-base-5.9-13.20130511.el7.noarch                                                                                                                                                                                88/117 
  Verifying  : openssh-clients-6.6.1p1-22.el7.x86_64                                                                                                                                                                                  89/117 
  Verifying  : libgpg-error-1.12-3.el7.x86_64                                                                                                                                                                                         90/117 
  Verifying  : cpio-2.11-24.el7.x86_64                                                                                                                                                                                                91/117 
  Verifying  : rpm-libs-4.11.3-17.el7.x86_64                                                                                                                                                                                          92/117 
  Verifying  : libattr-2.4.46-12.el7.x86_64                                                                                                                                                                                           93/117 
  Verifying  : zlib-1.2.7-15.el7.x86_64                                                                                                                                                                                               94/117 
  Verifying  : krb5-libs-1.13.2-10.el7.x86_64                                                                                                                                                                                         95/117 
  Verifying  : libcap-ng-0.7.5-4.el7.x86_64                                                                                                                                                                                           96/117 
  Verifying  : redhat-release-server-7.2-9.el7.x86_64                                                                                                                                                                                 97/117 
  Verifying  : libsepol-2.1.9-3.el7.x86_64                                                                                                                                                                                            98/117 
  Verifying  : libtasn1-3.8-2.el7.x86_64                                                                                                                                                                                              99/117 
  Verifying  : pyxattr-0.5.1-5.el7.x86_64                                                                                                                                                                                            100/117 
  Verifying  : libssh2-1.4.3-10.el7.x86_64                                                                                                                                                                                           101/117 
  Verifying  : sed-4.2.2-5.el7.x86_64                                                                                                                                                                                                102/117 
  Verifying  : chkconfig-1.3.61-5.el7.x86_64                                                                                                                                                                                         103/117 
  Verifying  : 1:findutils-4.5.11-5.el7.x86_64                                                                                                                                                                                       104/117 
  Verifying  : libxml2-2.9.1-5.el7_1.2.x86_64                                                                                                                                                                                        105/117 
  Verifying  : libgcc-4.8.5-4.el7.x86_64                                                                                                                                                                                             106/117 
  Verifying  : openldap-2.4.40-8.el7.x86_64                                                                                                                                                                                          107/117 
  Verifying  : ncurses-5.9-13.20130511.el7.x86_64                                                                                                                                                                                    108/117 
  Verifying  : python-chardet-2.2.1-1.el7_1.noarch                                                                                                                                                                                   109/117 
  Verifying  : libdb-utils-5.3.21-19.el7.x86_64                                                                                                                                                                                      110/117 
  Verifying  : nss-softokn-freebl-3.16.2.3-13.el7_1.x86_64                                                                                                                                                                           111/117 
  Verifying  : ncurses-libs-5.9-13.20130511.el7.x86_64                                                                                                                                                                               112/117 
  Verifying  : info-5.1-4.el7.x86_64                                                                                                                                                                                                 113/117 
  Verifying  : nspr-4.10.8-2.el7_1.x86_64                                                                                                                                                                                            114/117 
  Verifying  : diffutils-3.3-4.el7.x86_64                                                                                                                                                                                            115/117 
  Verifying  : gnupg2-2.0.22-3.el7.x86_64                                                                                                                                                                                            116/117 
  Verifying  : python-urlgrabber-3.10-7.el7.noarch                                                                                                                                                                                   117/117 

Installed:
  coreutils.x86_64 0:8.22-15.el7       openssh-clients.x86_64 0:6.6.1p1-22.el7     procps-ng.x86_64 0:3.3.10-3.el7     redhat-release-server.x86_64 0:7.2-9.el7     util-linux.x86_64 0:2.23.2-26.el7     yum.noarch 0:3.4.3-132.el7    
  yum-utils.noarch 0:1.1.31-34.el7    

Dependency Installed:
  audit-libs.x86_64 0:2.4.1-5.el7                 basesystem.noarch 0:10.0-7.el7              bash.x86_64 0:4.2.46-19.el7                 bzip2-libs.x86_64 0:1.0.6-13.el7                ca-certificates.noarch 0:2015.2.4-71.el7           
  chkconfig.x86_64 0:1.3.61-5.el7                 cpio.x86_64 0:2.11-24.el7                   cracklib.x86_64 0:2.9.0-11.el7              cracklib-dicts.x86_64 0:2.9.0-11.el7            curl.x86_64 0:7.29.0-25.el7                        
  cyrus-sasl-lib.x86_64 0:2.1.26-19.2.el7         diffutils.x86_64 0:3.3-4.el7                elfutils-libelf.x86_64 0:0.163-3.el7        elfutils-libs.x86_64 0:0.163-3.el7              expat.x86_64 0:2.1.0-8.el7                         
  file-libs.x86_64 0:5.11-31.el7                  filesystem.x86_64 0:3.2-20.el7              findutils.x86_64 1:4.5.11-5.el7             fipscheck.x86_64 0:1.4.1-5.el7                  fipscheck-lib.x86_64 0:1.4.1-5.el7                 
  gawk.x86_64 0:4.0.2-4.el7                       gdbm.x86_64 0:1.10-8.el7                    glib2.x86_64 0:2.42.2-5.el7                 glibc.x86_64 0:2.17-105.el7                     glibc-common.x86_64 0:2.17-105.el7                 
  gmp.x86_64 1:6.0.0-11.el7                       gnupg2.x86_64 0:2.0.22-3.el7                gpgme.x86_64 0:1.3.2-5.el7                  grep.x86_64 0:2.20-2.el7                        gzip.x86_64 0:1.5-8.el7                            
  info.x86_64 0:5.1-4.el7                         keyutils-libs.x86_64 0:1.5.8-3.el7          krb5-libs.x86_64 0:1.13.2-10.el7            libacl.x86_64 0:2.2.51-12.el7                   libassuan.x86_64 0:2.1.0-3.el7                     
  libattr.x86_64 0:2.4.46-12.el7                  libblkid.x86_64 0:2.23.2-26.el7             libcap.x86_64 0:2.22-8.el7                  libcap-ng.x86_64 0:0.7.5-4.el7                  libcom_err.x86_64 0:1.42.9-7.el7                   
  libcurl.x86_64 0:7.29.0-25.el7                  libdb.x86_64 0:5.3.21-19.el7                libdb-utils.x86_64 0:5.3.21-19.el7          libedit.x86_64 0:3.0-12.20121213cvs.el7         libffi.x86_64 0:3.0.13-16.el7                      
  libgcc.x86_64 0:4.8.5-4.el7                     libgcrypt.x86_64 0:1.5.3-12.el7_1.1         libgpg-error.x86_64 0:1.12-3.el7            libidn.x86_64 0:1.28-4.el7                      libmount.x86_64 0:2.23.2-26.el7                    
  libpwquality.x86_64 0:1.2.3-4.el7               libselinux.x86_64 0:2.2.2-6.el7             libsemanage.x86_64 0:2.1.10-18.el7          libsepol.x86_64 0:2.1.9-3.el7                   libssh2.x86_64 0:1.4.3-10.el7                      
  libstdc++.x86_64 0:4.8.5-4.el7                  libtasn1.x86_64 0:3.8-2.el7                 libuser.x86_64 0:0.60-7.el7_1               libutempter.x86_64 0:1.1.6-4.el7                libuuid.x86_64 0:2.23.2-26.el7                     
  libverto.x86_64 0:0.2.5-4.el7                   libxml2.x86_64 0:2.9.1-5.el7_1.2            lua.x86_64 0:5.1.4-14.el7                   ncurses.x86_64 0:5.9-13.20130511.el7            ncurses-base.noarch 0:5.9-13.20130511.el7          
  ncurses-libs.x86_64 0:5.9-13.20130511.el7       nspr.x86_64 0:4.10.8-2.el7_1                nss.x86_64 0:3.19.1-18.el7                  nss-softokn.x86_64 0:3.16.2.3-13.el7_1          nss-softokn-freebl.x86_64 0:3.16.2.3-13.el7_1      
  nss-sysinit.x86_64 0:3.19.1-18.el7              nss-tools.x86_64 0:3.19.1-18.el7            nss-util.x86_64 0:3.19.1-4.el7_1            openldap.x86_64 0:2.4.40-8.el7                  openssh.x86_64 0:6.6.1p1-22.el7                    
  openssl-libs.x86_64 1:1.0.1e-42.el7_1.9         p11-kit.x86_64 0:0.20.7-3.el7               p11-kit-trust.x86_64 0:0.20.7-3.el7         pam.x86_64 0:1.1.8-12.el7_1.1                   pcre.x86_64 0:8.32-15.el7                          
  pinentry.x86_64 0:0.8.1-14.el7                  pkgconfig.x86_64 1:0.27.1-4.el7             popt.x86_64 0:1.13-16.el7                   pth.x86_64 0:2.0.7-23.el7                       pygpgme.x86_64 0:0.3-9.el7                         
  pyliblzma.x86_64 0:0.5.3-11.el7                 python.x86_64 0:2.7.5-34.el7                python-chardet.noarch 0:2.2.1-1.el7_1       python-iniparse.noarch 0:0.4-9.el7              python-kitchen.noarch 0:1.1.1-5.el7                
  python-libs.x86_64 0:2.7.5-34.el7               python-pycurl.x86_64 0:7.19.0-17.el7        python-urlgrabber.noarch 0:3.10-7.el7       pyxattr.x86_64 0:0.5.1-5.el7                    readline.x86_64 0:6.2-9.el7                        
  rpm.x86_64 0:4.11.3-17.el7                      rpm-build-libs.x86_64 0:4.11.3-17.el7       rpm-libs.x86_64 0:4.11.3-17.el7             rpm-python.x86_64 0:4.11.3-17.el7               sed.x86_64 0:4.2.2-5.el7                           
  setup.noarch 0:2.8.71-6.el7                     shadow-utils.x86_64 2:4.1.5.1-18.el7        shared-mime-info.x86_64 0:1.1-9.el7         sqlite.x86_64 0:3.7.17-8.el7                    systemd-libs.x86_64 0:219-19.el7                   
  tzdata.noarch 0:2015g-1.el7                     ustr.x86_64 0:1.0.4-16.el7                  xz-libs.x86_64 0:5.1.2-12alpha.el7          yum-metadata-parser.x86_64 0:1.1.4-10.el7       zlib.x86_64 0:1.2.7-15.el7                         

Complete!
Executing Postbootstrap module
+ echo 'Looking in directory '\''/var/singularity/mnt/final'\'' for /bin/sh'
Looking in directory '/var/singularity/mnt/final' for /bin/sh
+ '[' '!' -x /var/singularity/mnt/final/bin/sh ']'
+ cp HPCGRHEL7.2.repo /var/singularity/mnt/final/etc/yum.repos.d/
+ mv /var/singularity/mnt/final/usr/bin/ssh /var/singularity/mnt/final/usr/bin/ssh_real
+ cp ssh-replacement.sh /var/singularity/mnt/final/usr/bin/ssh
+ chmod 755 /var/singularity/mnt/final/usr/bin/ssh
+ echo '* hard memlock unlimited'
+ echo '* soft memlock unlimited'
+ cd /var/singularity/mnt/final/usr/local/
+ echo 'Installing MPI and IB'
Installing MPI and IB
+ yum -y install libipathverbs mvapich2_intel_qlc hostname infinipath-libs libgfortran hwloc numactl numactl-devel
HPCGRHEL7.2                                                                                                                                                                                                           | 2.9 kB  00:00:00     
IB_REPO                                                                                                                                                                                                               | 2.9 kB  00:00:00     
(1/2): IB_REPO/primary_db                                                                                                                                                                                             |  71 kB  00:00:00     
(2/2): HPCGRHEL7.2/primary_db                                                                                                                                                                                         | 3.7 MB  00:00:00     
Resolving Dependencies
--> Running transaction check
---> Package hostname.x86_64 0:3.13-3.el7 will be installed
---> Package hwloc.x86_64 0:1.7-5.el7 will be installed
--> Processing Dependency: hwloc-libs = 1.7-5.el7 for package: hwloc-1.7-5.el7.x86_64
--> Processing Dependency: libpciaccess.so.0()(64bit) for package: hwloc-1.7-5.el7.x86_64
--> Processing Dependency: libnuma.so.1()(64bit) for package: hwloc-1.7-5.el7.x86_64
--> Processing Dependency: libhwloc.so.5()(64bit) for package: hwloc-1.7-5.el7.x86_64
---> Package infinipath-libs.x86_64 0:3.3-75029.1218_rhel7_qlc will be installed
---> Package libgfortran.x86_64 0:4.8.5-4.el7 will be installed
--> Processing Dependency: libquadmath = 4.8.5-4.el7 for package: libgfortran-4.8.5-4.el7.x86_64
--> Processing Dependency: libquadmath.so.0(QUADMATH_1.0)(64bit) for package: libgfortran-4.8.5-4.el7.x86_64
--> Processing Dependency: libquadmath.so.0()(64bit) for package: libgfortran-4.8.5-4.el7.x86_64
---> Package libipathverbs.x86_64 0:1.3-2.el7 will be installed
--> Processing Dependency: rdma for package: libipathverbs-1.3-2.el7.x86_64
--> Processing Dependency: libibverbs.so.1(IBVERBS_1.1)(64bit) for package: libipathverbs-1.3-2.el7.x86_64
--> Processing Dependency: libibverbs.so.1(IBVERBS_1.0)(64bit) for package: libipathverbs-1.3-2.el7.x86_64
--> Processing Dependency: libibverbs.so.1()(64bit) for package: libipathverbs-1.3-2.el7.x86_64
---> Package mvapich2_intel_qlc.x86_64 0:2.1-1 will be installed
--> Processing Dependency: librdmacm for package: mvapich2_intel_qlc-2.1-1.x86_64
--> Processing Dependency: libibumad for package: mvapich2_intel_qlc-2.1-1.x86_64
---> Package numactl.x86_64 0:2.0.9-5.el7_1 will be installed
---> Package numactl-devel.x86_64 0:2.0.9-5.el7_1 will be installed
--> Running transaction check
---> Package hwloc-libs.x86_64 0:1.7-5.el7 will be installed
---> Package libibumad.x86_64 0:1.3.10.2-1.3.10.2 will be installed
---> Package libibverbs.x86_64 0:1.1.8-8.el7 will be installed
--> Processing Dependency: libnl-route-3.so.200()(64bit) for package: libibverbs-1.1.8-8.el7.x86_64
--> Processing Dependency: libnl-3.so.200()(64bit) for package: libibverbs-1.1.8-8.el7.x86_64
---> Package libpciaccess.x86_64 0:0.13.4-2.el7 will be installed
--> Processing Dependency: hwdata for package: libpciaccess-0.13.4-2.el7.x86_64
---> Package libquadmath.x86_64 0:4.8.5-4.el7 will be installed
---> Package librdmacm.x86_64 0:1.0.21-1.el7 will be installed
---> Package numactl-libs.x86_64 0:2.0.9-5.el7_1 will be installed
---> Package rdma.noarch 0:7.2_4.1_rc6-1.el7 will be installed
--> Processing Dependency: udev >= 095 for package: rdma-7.2_4.1_rc6-1.el7.noarch
--> Processing Dependency: systemd-units for package: rdma-7.2_4.1_rc6-1.el7.noarch
--> Processing Dependency: systemd-units for package: rdma-7.2_4.1_rc6-1.el7.noarch
--> Processing Dependency: dracut for package: rdma-7.2_4.1_rc6-1.el7.noarch
--> Running transaction check
---> Package dracut.x86_64 0:033-359.el7 will be installed
--> Processing Dependency: xz for package: dracut-033-359.el7.x86_64
--> Processing Dependency: kpartx for package: dracut-033-359.el7.x86_64
--> Processing Dependency: kmod for package: dracut-033-359.el7.x86_64
--> Processing Dependency: hardlink for package: dracut-033-359.el7.x86_64
---> Package hwdata.x86_64 0:0.252-8.1.el7 will be installed
---> Package libnl3.x86_64 0:3.2.21-10.el7 will be installed
---> Package systemd.x86_64 0:219-19.el7 will be installed
--> Processing Dependency: libkmod.so.2(LIBKMOD_5)(64bit) for package: systemd-219-19.el7.x86_64
--> Processing Dependency: libcryptsetup.so.4(CRYPTSETUP_1.0)(64bit) for package: systemd-219-19.el7.x86_64
--> Processing Dependency: dbus for package: systemd-219-19.el7.x86_64
--> Processing Dependency: acl for package: systemd-219-19.el7.x86_64
--> Processing Dependency: libqrencode.so.3()(64bit) for package: systemd-219-19.el7.x86_64
--> Processing Dependency: libkmod.so.2()(64bit) for package: systemd-219-19.el7.x86_64
--> Processing Dependency: libcryptsetup.so.4()(64bit) for package: systemd-219-19.el7.x86_64
--> Running transaction check
---> Package acl.x86_64 0:2.2.51-12.el7 will be installed
---> Package cryptsetup-libs.x86_64 0:1.6.7-1.el7 will be installed
--> Processing Dependency: libdevmapper.so.1.02(Base)(64bit) for package: cryptsetup-libs-1.6.7-1.el7.x86_64
--> Processing Dependency: libdevmapper.so.1.02()(64bit) for package: cryptsetup-libs-1.6.7-1.el7.x86_64
---> Package dbus.x86_64 1:1.6.12-13.el7 will be installed
--> Processing Dependency: dbus-libs(x86-64) = 1:1.6.12-13.el7 for package: 1:dbus-1.6.12-13.el7.x86_64
--> Processing Dependency: libdbus-1.so.3()(64bit) for package: 1:dbus-1.6.12-13.el7.x86_64
---> Package hardlink.x86_64 1:1.0-19.el7 will be installed
---> Package kmod.x86_64 0:20-5.el7 will be installed
--> Processing Dependency: /usr/bin/nm for package: kmod-20-5.el7.x86_64
---> Package kmod-libs.x86_64 0:20-5.el7 will be installed
---> Package kpartx.x86_64 0:0.4.9-85.el7 will be installed
---> Package qrencode-libs.x86_64 0:3.4.1-3.el7 will be installed
---> Package xz.x86_64 0:5.1.2-12alpha.el7 will be installed
--> Running transaction check
---> Package binutils.x86_64 0:2.23.52.0.1-55.el7 will be installed
---> Package dbus-libs.x86_64 1:1.6.12-13.el7 will be installed
---> Package device-mapper-libs.x86_64 7:1.02.107-5.el7 will be installed
--> Processing Dependency: device-mapper = 7:1.02.107-5.el7 for package: 7:device-mapper-libs-1.02.107-5.el7.x86_64
--> Running transaction check
---> Package device-mapper.x86_64 7:1.02.107-5.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=============================================================================================================================================================================================================================================
 Package                                                     Arch                                            Version                                                              Repository                                            Size
=============================================================================================================================================================================================================================================
Installing:
 hostname                                                    x86_64                                          3.13-3.el7                                                           HPCGRHEL7.2                                           17 k
 hwloc                                                       x86_64                                          1.7-5.el7                                                            HPCGRHEL7.2                                          116 k
 infinipath-libs                                             x86_64                                          3.3-75029.1218_rhel7_qlc                                             IB_REPO                                              698 k
 libgfortran                                                 x86_64                                          4.8.5-4.el7                                                          HPCGRHEL7.2                                          293 k
 libipathverbs                                               x86_64                                          1.3-2.el7                                                            HPCGRHEL7.2                                           19 k
 mvapich2_intel_qlc                                          x86_64                                          2.1-1                                                                IB_REPO                                              4.6 M
 numactl                                                     x86_64                                          2.0.9-5.el7_1                                                        HPCGRHEL7.2                                           65 k
 numactl-devel                                               x86_64                                          2.0.9-5.el7_1                                                        HPCGRHEL7.2                                           23 k
Installing for dependencies:
 acl                                                         x86_64                                          2.2.51-12.el7                                                        HPCGRHEL7.2                                           81 k
 binutils                                                    x86_64                                          2.23.52.0.1-55.el7                                                   HPCGRHEL7.2                                          5.0 M
 cryptsetup-libs                                             x86_64                                          1.6.7-1.el7                                                          HPCGRHEL7.2                                          182 k
 dbus                                                        x86_64                                          1:1.6.12-13.el7                                                      HPCGRHEL7.2                                          307 k
 dbus-libs                                                   x86_64                                          1:1.6.12-13.el7                                                      HPCGRHEL7.2                                          151 k
 device-mapper                                               x86_64                                          7:1.02.107-5.el7                                                     HPCGRHEL7.2                                          251 k
 device-mapper-libs                                          x86_64                                          7:1.02.107-5.el7                                                     HPCGRHEL7.2                                          304 k
 dracut                                                      x86_64                                          033-359.el7                                                          HPCGRHEL7.2                                          311 k
 hardlink                                                    x86_64                                          1:1.0-19.el7                                                         HPCGRHEL7.2                                           14 k
 hwdata                                                      x86_64                                          0.252-8.1.el7                                                        HPCGRHEL7.2                                          2.1 M
 hwloc-libs                                                  x86_64                                          1.7-5.el7                                                            HPCGRHEL7.2                                          1.3 M
 kmod                                                        x86_64                                          20-5.el7                                                             HPCGRHEL7.2                                          114 k
 kmod-libs                                                   x86_64                                          20-5.el7                                                             HPCGRHEL7.2                                           47 k
 kpartx                                                      x86_64                                          0.4.9-85.el7                                                         HPCGRHEL7.2                                           59 k
 libibumad                                                   x86_64                                          1.3.10.2-1.3.10.2                                                    IB_REPO                                               63 k
 libibverbs                                                  x86_64                                          1.1.8-8.el7                                                          HPCGRHEL7.2                                           56 k
 libnl3                                                      x86_64                                          3.2.21-10.el7                                                        HPCGRHEL7.2                                          197 k
 libpciaccess                                                x86_64                                          0.13.4-2.el7                                                         HPCGRHEL7.2                                           26 k
 libquadmath                                                 x86_64                                          4.8.5-4.el7                                                          HPCGRHEL7.2                                          182 k
 librdmacm                                                   x86_64                                          1.0.21-1.el7                                                         HPCGRHEL7.2                                           64 k
 numactl-libs                                                x86_64                                          2.0.9-5.el7_1                                                        HPCGRHEL7.2                                           29 k
 qrencode-libs                                               x86_64                                          3.4.1-3.el7                                                          HPCGRHEL7.2                                           50 k
 rdma                                                        noarch                                          7.2_4.1_rc6-1.el7                                                    HPCGRHEL7.2                                           28 k
 systemd                                                     x86_64                                          219-19.el7                                                           HPCGRHEL7.2                                          5.1 M
 xz                                                          x86_64                                          5.1.2-12alpha.el7                                                    HPCGRHEL7.2                                          200 k

Transaction Summary
=============================================================================================================================================================================================================================================
Install  8 Packages (+25 Dependent packages)

Total download size: 22 M
Installed size: 92 M
Downloading packages:
(1/33): acl-2.2.51-12.el7.x86_64.rpm                                                                                                                                                                                  |  81 kB  00:00:00     
(2/33): cryptsetup-libs-1.6.7-1.el7.x86_64.rpm                                                                                                                                                                        | 182 kB  00:00:00     
(3/33): dbus-1.6.12-13.el7.x86_64.rpm                                                                                                                                                                                 | 307 kB  00:00:00     
(4/33): dbus-libs-1.6.12-13.el7.x86_64.rpm                                                                                                                                                                            | 151 kB  00:00:00     
(5/33): device-mapper-1.02.107-5.el7.x86_64.rpm                                                                                                                                                                       | 251 kB  00:00:00     
(6/33): binutils-2.23.52.0.1-55.el7.x86_64.rpm                                                                                                                                                                        | 5.0 MB  00:00:00     
(7/33): device-mapper-libs-1.02.107-5.el7.x86_64.rpm                                                                                                                                                                  | 304 kB  00:00:00     
(8/33): dracut-033-359.el7.x86_64.rpm                                                                                                                                                                                 | 311 kB  00:00:00     
(9/33): hardlink-1.0-19.el7.x86_64.rpm                                                                                                                                                                                |  14 kB  00:00:00     
(10/33): hostname-3.13-3.el7.x86_64.rpm                                                                                                                                                                               |  17 kB  00:00:00     
(11/33): hwloc-1.7-5.el7.x86_64.rpm                                                                                                                                                                                   | 116 kB  00:00:00     
(12/33): hwdata-0.252-8.1.el7.x86_64.rpm                                                                                                                                                                              | 2.1 MB  00:00:00     
(13/33): hwloc-libs-1.7-5.el7.x86_64.rpm                                                                                                                                                                              | 1.3 MB  00:00:00     
(14/33): kmod-20-5.el7.x86_64.rpm                                                                                                                                                                                     | 114 kB  00:00:00     
(15/33): kmod-libs-20-5.el7.x86_64.rpm                                                                                                                                                                                |  47 kB  00:00:00     
(16/33): kpartx-0.4.9-85.el7.x86_64.rpm                                                                                                                                                                               |  59 kB  00:00:00     
(17/33): libgfortran-4.8.5-4.el7.x86_64.rpm                                                                                                                                                                           | 293 kB  00:00:00     
(18/33): libibverbs-1.1.8-8.el7.x86_64.rpm                                                                                                                                                                            |  56 kB  00:00:00     
(19/33): infinipath-libs-3.3-75029.1218_rhel7_qlc.x86_64.rpm                                                                                                                                                          | 698 kB  00:00:00     
(20/33): libnl3-3.2.21-10.el7.x86_64.rpm                                                                                                                                                                              | 197 kB  00:00:00     
(21/33): libpciaccess-0.13.4-2.el7.x86_64.rpm                                                                                                                                                                         |  26 kB  00:00:00     
(22/33): libquadmath-4.8.5-4.el7.x86_64.rpm                                                                                                                                                                           | 182 kB  00:00:00     
(23/33): librdmacm-1.0.21-1.el7.x86_64.rpm                                                                                                                                                                            |  64 kB  00:00:00     
(24/33): libibumad-1.3.10.2-1.3.10.2.x86_64.rpm                                                                                                                                                                       |  63 kB  00:00:00     
(25/33): numactl-2.0.9-5.el7_1.x86_64.rpm                                                                                                                                                                             |  65 kB  00:00:00     
(26/33): libipathverbs-1.3-2.el7.x86_64.rpm                                                                                                                                                                           |  19 kB  00:00:00     
(27/33): numactl-devel-2.0.9-5.el7_1.x86_64.rpm                                                                                                                                                                       |  23 kB  00:00:00     
(28/33): numactl-libs-2.0.9-5.el7_1.x86_64.rpm                                                                                                                                                                        |  29 kB  00:00:00     
(29/33): qrencode-libs-3.4.1-3.el7.x86_64.rpm                                                                                                                                                                         |  50 kB  00:00:00     
(30/33): rdma-7.2_4.1_rc6-1.el7.noarch.rpm                                                                                                                                                                            |  28 kB  00:00:00     
(31/33): xz-5.1.2-12alpha.el7.x86_64.rpm                                                                                                                                                                              | 200 kB  00:00:00     
(32/33): systemd-219-19.el7.x86_64.rpm                                                                                                                                                                                | 5.1 MB  00:00:00     
(33/33): mvapich2_intel_qlc-2.1-1.x86_64.rpm                                                                                                                                                                          | 4.6 MB  00:00:00     
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                                                                         64 MB/s |  22 MB  00:00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : numactl-libs-2.0.9-5.el7_1.x86_64                                                                                                                                                                                        1/33 
  Installing : xz-5.1.2-12alpha.el7.x86_64                                                                                                                                                                                              2/33 
  Installing : libquadmath-4.8.5-4.el7.x86_64                                                                                                                                                                                           3/33 
  Installing : 1:hardlink-1.0-19.el7.x86_64                                                                                                                                                                                             4/33 
  Installing : libnl3-3.2.21-10.el7.x86_64                                                                                                                                                                                              5/33 
  Installing : libibumad-1.3.10.2-1.3.10.2.x86_64                                                                                                                                                                                       6/33 
  Installing : 1:dbus-libs-1.6.12-13.el7.x86_64                                                                                                                                                                                         7/33 
  Installing : acl-2.2.51-12.el7.x86_64                                                                                                                                                                                                 8/33 
  Installing : binutils-2.23.52.0.1-55.el7.x86_64                                                                                                                                                                                       9/33 
  Installing : qrencode-libs-3.4.1-3.el7.x86_64                                                                                                                                                                                        10/33 
  Installing : kmod-libs-20-5.el7.x86_64                                                                                                                                                                                               11/33 
  Installing : kpartx-0.4.9-85.el7.x86_64                                                                                                                                                                                              12/33 
  Installing : 7:device-mapper-1.02.107-5.el7.x86_64                                                                                                                                                                                   13/33 
  Installing : 7:device-mapper-libs-1.02.107-5.el7.x86_64                                                                                                                                                                              14/33 
  Installing : cryptsetup-libs-1.6.7-1.el7.x86_64                                                                                                                                                                                      15/33 
  Installing : dracut-033-359.el7.x86_64                                                                                                                                                                                               16/33 
  Installing : kmod-20-5.el7.x86_64                                                                                                                                                                                                    17/33 
  Installing : systemd-219-19.el7.x86_64                                                                                                                                                                                               18/33 
  Installing : 1:dbus-1.6.12-13.el7.x86_64                                                                                                                                                                                             19/33 
  Installing : rdma-7.2_4.1_rc6-1.el7.noarch                                                                                                                                                                                           20/33 
  Installing : libibverbs-1.1.8-8.el7.x86_64                                                                                                                                                                                           21/33 
  Installing : librdmacm-1.0.21-1.el7.x86_64                                                                                                                                                                                           22/33 
  Installing : hwdata-0.252-8.1.el7.x86_64                                                                                                                                                                                             23/33 
  Installing : libpciaccess-0.13.4-2.el7.x86_64                                                                                                                                                                                        24/33 
  Installing : hwloc-libs-1.7-5.el7.x86_64                                                                                                                                                                                             25/33 
  Installing : hwloc-1.7-5.el7.x86_64                                                                                                                                                                                                  26/33 
  Installing : mvapich2_intel_qlc-2.1-1.x86_64                                                                                                                                                                                         27/33 
/var/tmp/rpm-tmp.R1Tjut: line 2: /usr/bin/mpi-selector: No such file or directory
warning: %post(mvapich2_intel_qlc-2.1-1.x86_64) scriptlet failed, exit status 127
Non-fatal POSTIN scriptlet failure in rpm package mvapich2_intel_qlc-2.1-1.x86_64
  Installing : libipathverbs-1.3-2.el7.x86_64                                                                                                                                                                                          28/33 
  Installing : libgfortran-4.8.5-4.el7.x86_64                                                                                                                                                                                          29/33 
  Installing : numactl-2.0.9-5.el7_1.x86_64                                                                                                                                                                                            30/33 
  Installing : numactl-devel-2.0.9-5.el7_1.x86_64                                                                                                                                                                                      31/33 
  Installing : hostname-3.13-3.el7.x86_64                                                                                                                                                                                              32/33 
  Installing : infinipath-libs-3.3-75029.1218_rhel7_qlc.x86_64                                                                                                                                                                         33/33 
  Verifying  : numactl-2.0.9-5.el7_1.x86_64                                                                                                                                                                                             1/33 
  Verifying  : rdma-7.2_4.1_rc6-1.el7.noarch                                                                                                                                                                                            2/33 
  Verifying  : xz-5.1.2-12alpha.el7.x86_64                                                                                                                                                                                              3/33 
  Verifying  : infinipath-libs-3.3-75029.1218_rhel7_qlc.x86_64                                                                                                                                                                          4/33 
  Verifying  : 1:dbus-1.6.12-13.el7.x86_64                                                                                                                                                                                              5/33 
  Verifying  : librdmacm-1.0.21-1.el7.x86_64                                                                                                                                                                                            6/33 
  Verifying  : kmod-20-5.el7.x86_64                                                                                                                                                                                                     7/33 
  Verifying  : hwdata-0.252-8.1.el7.x86_64                                                                                                                                                                                              8/33 
  Verifying  : kmod-libs-20-5.el7.x86_64                                                                                                                                                                                                9/33 
  Verifying  : mvapich2_intel_qlc-2.1-1.x86_64                                                                                                                                                                                         10/33 
  Verifying  : qrencode-libs-3.4.1-3.el7.x86_64                                                                                                                                                                                        11/33 
  Verifying  : systemd-219-19.el7.x86_64                                                                                                                                                                                               12/33 
  Verifying  : libpciaccess-0.13.4-2.el7.x86_64                                                                                                                                                                                        13/33 
  Verifying  : dracut-033-359.el7.x86_64                                                                                                                                                                                               14/33 
  Verifying  : 7:device-mapper-libs-1.02.107-5.el7.x86_64                                                                                                                                                                              15/33 
  Verifying  : binutils-2.23.52.0.1-55.el7.x86_64                                                                                                                                                                                      16/33 
  Verifying  : hwloc-libs-1.7-5.el7.x86_64                                                                                                                                                                                             17/33 
  Verifying  : acl-2.2.51-12.el7.x86_64                                                                                                                                                                                                18/33 
  Verifying  : libipathverbs-1.3-2.el7.x86_64                                                                                                                                                                                          19/33 
  Verifying  : kpartx-0.4.9-85.el7.x86_64                                                                                                                                                                                              20/33 
  Verifying  : cryptsetup-libs-1.6.7-1.el7.x86_64                                                                                                                                                                                      21/33 
  Verifying  : libgfortran-4.8.5-4.el7.x86_64                                                                                                                                                                                          22/33 
  Verifying  : 1:dbus-libs-1.6.12-13.el7.x86_64                                                                                                                                                                                        23/33 
  Verifying  : hostname-3.13-3.el7.x86_64                                                                                                                                                                                              24/33 
  Verifying  : numactl-libs-2.0.9-5.el7_1.x86_64                                                                                                                                                                                       25/33 
  Verifying  : libibumad-1.3.10.2-1.3.10.2.x86_64                                                                                                                                                                                      26/33 
  Verifying  : libibverbs-1.1.8-8.el7.x86_64                                                                                                                                                                                           27/33 
  Verifying  : hwloc-1.7-5.el7.x86_64                                                                                                                                                                                                  28/33 
  Verifying  : numactl-devel-2.0.9-5.el7_1.x86_64                                                                                                                                                                                      29/33 
  Verifying  : libnl3-3.2.21-10.el7.x86_64                                                                                                                                                                                             30/33 
  Verifying  : 1:hardlink-1.0-19.el7.x86_64                                                                                                                                                                                            31/33 
  Verifying  : libquadmath-4.8.5-4.el7.x86_64                                                                                                                                                                                          32/33 
  Verifying  : 7:device-mapper-1.02.107-5.el7.x86_64                                                                                                                                                                                   33/33 

Installed:
  hostname.x86_64 0:3.13-3.el7      hwloc.x86_64 0:1.7-5.el7                infinipath-libs.x86_64 0:3.3-75029.1218_rhel7_qlc    libgfortran.x86_64 0:4.8.5-4.el7    libipathverbs.x86_64 0:1.3-2.el7    mvapich2_intel_qlc.x86_64 0:2.1-1   
  numactl.x86_64 0:2.0.9-5.el7_1    numactl-devel.x86_64 0:2.0.9-5.el7_1   

Dependency Installed:
  acl.x86_64 0:2.2.51-12.el7                   binutils.x86_64 0:2.23.52.0.1-55.el7   cryptsetup-libs.x86_64 0:1.6.7-1.el7   dbus.x86_64 1:1.6.12-13.el7          dbus-libs.x86_64 1:1.6.12-13.el7   device-mapper.x86_64 7:1.02.107-5.el7  
  device-mapper-libs.x86_64 7:1.02.107-5.el7   dracut.x86_64 0:033-359.el7            hardlink.x86_64 1:1.0-19.el7           hwdata.x86_64 0:0.252-8.1.el7        hwloc-libs.x86_64 0:1.7-5.el7      kmod.x86_64 0:20-5.el7                 
  kmod-libs.x86_64 0:20-5.el7                  kpartx.x86_64 0:0.4.9-85.el7           libibumad.x86_64 0:1.3.10.2-1.3.10.2   libibverbs.x86_64 0:1.1.8-8.el7      libnl3.x86_64 0:3.2.21-10.el7      libpciaccess.x86_64 0:0.13.4-2.el7     
  libquadmath.x86_64 0:4.8.5-4.el7             librdmacm.x86_64 0:1.0.21-1.el7        numactl-libs.x86_64 0:2.0.9-5.el7_1    qrencode-libs.x86_64 0:3.4.1-3.el7   rdma.noarch 0:7.2_4.1_rc6-1.el7    systemd.x86_64 0:219-19.el7            
  xz.x86_64 0:5.1.2-12alpha.el7               

Complete!
+ yum -y erase mvapich2_intel_qlc
Resolving Dependencies
--> Running transaction check
---> Package mvapich2_intel_qlc.x86_64 0:2.1-1 will be erased
--> Finished Dependency Resolution

Dependencies Resolved

=============================================================================================================================================================================================================================================
 Package                                                           Arch                                                  Version                                               Repository                                               Size
=============================================================================================================================================================================================================================================
Removing:
 mvapich2_intel_qlc                                                x86_64                                                2.1-1                                                 @IB_REPO                                                 27 M

Transaction Summary
=============================================================================================================================================================================================================================================
Remove  1 Package

Installed size: 27 M
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
/var/tmp/rpm-tmp.Le7MdH: line 1: /usr/bin/mpi-selector: No such file or directory
  Erasing    : mvapich2_intel_qlc-2.1-1.x86_64                                                                                                                                                                                           1/1 
  Verifying  : mvapich2_intel_qlc-2.1-1.x86_64                                                                                                                                                                                           1/1 

Removed:
  mvapich2_intel_qlc.x86_64 0:2.1-1                                                                                                                                                                                                          

Complete!
+ rm -rf '/usr/mpi/gcc/mvapich2_intel_qlc*'
+ yumdownloader mvapich2_intel_qlc
mvapich2_intel_qlc-2.1-1.x86_64.rpm                                                                                                                                                                                   | 4.6 MB  00:00:00     
+ rpm -ivh mvapich2_intel_qlc-2.1-1.x86_64.rpm
Preparing...                          ################################# [100%]
Updating / installing...
   1:mvapich2_intel_qlc-2.1-1         ################################# [100%]
/var/tmp/rpm-tmp.4vUco6: line 2: /usr/bin/mpi-selector: No such file or directory
warning: %post(mvapich2_intel_qlc-2.1-1.x86_64) scriptlet failed, exit status 127
+ yum -y install openssh-clients tar strace
Package openssh-clients-6.6.1p1-22.el7.x86_64 already installed and latest version
Resolving Dependencies
--> Running transaction check
---> Package strace.x86_64 0:4.8-11.el7 will be installed
---> Package tar.x86_64 2:1.26-29.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=============================================================================================================================================================================================================================================
 Package                                               Arch                                                  Version                                                        Repository                                                  Size
=============================================================================================================================================================================================================================================
Installing:
 strace                                                x86_64                                                4.8-11.el7                                                     HPCGRHEL7.2                                                265 k
 tar                                                   x86_64                                                2:1.26-29.el7                                                  HPCGRHEL7.2                                                843 k

Transaction Summary
=============================================================================================================================================================================================================================================
Install  2 Packages

Total download size: 1.1 M
Installed size: 3.6 M
Downloading packages:
(1/2): strace-4.8-11.el7.x86_64.rpm                                                                                                                                                                                   | 265 kB  00:00:00     
(2/2): tar-1.26-29.el7.x86_64.rpm                                                                                                                                                                                     | 843 kB  00:00:00     
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                                                                         14 MB/s | 1.1 MB  00:00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
Warning: RPMDB altered outside of yum.
  Installing : 2:tar-1.26-29.el7.x86_64                                                                                                                                                                                                  1/2 
  Installing : strace-4.8-11.el7.x86_64                                                                                                                                                                                                  2/2 
  Verifying  : strace-4.8-11.el7.x86_64                                                                                                                                                                                                  1/2 
  Verifying  : 2:tar-1.26-29.el7.x86_64                                                                                                                                                                                                  2/2 

Installed:
  strace.x86_64 0:4.8-11.el7                                                                                             tar.x86_64 2:1.26-29.el7                                                                                            

Complete!
+ ls /enod/hpc
Done.

</pre>

<pre>
&lt;shell&gt;$ ./singularity-mvapich2-mpirun -np 64 -hostfile &lt;hostfile&gt; IMB-MPI2 -npmin 32 sendrecv exchange allgather 

#---------------------------------------------------
#---------------------------------------------------
# Date                  : Thu Mar  9 10:26:52 2017
# Machine               : x86_64
# System                : Linux
# Release               : 3.10.0-327.el7.x86_64
# Version               : #1 SMP Thu Oct 29 17:29:29 EDT 2015
# MPI Version           : 3.0
# MPI Thread Environment: MPI_THREAD_MULTIPLE


# New default behavior from Version 3.2 on:

# the number of iterations per message size is cut down
# dynamically when a certain run time (per message size sample)
# is expected to be exceeded. Time limit is defined by variable
# "SECS_PER_SAMPLE" (=> IMB_settings.h)
# or through the flag => -time



# Calling sequence was:

# IMB-MPI2 -npmin 32 sendrecv exchange allgather

# Minimum message length in bytes:   0
# Maximum message length in bytes:   4194304
#
# MPI_Datatype                   :   MPI_BYTE
# MPI_Datatype for reductions    :   MPI_FLOAT
# MPI_Op                         :   MPI_SUM
#
#

# List of Benchmarks to run:

# Sendrecv
# Exchange
# Allgather

#-----------------------------------------------------------------------------
# Benchmarking Sendrecv
# #processes = 32
# ( 32 additional processes waiting in MPI_Barrier)
#-----------------------------------------------------------------------------
       #bytes #repetitions  t_min[usec]  t_max[usec]  t_avg[usec]   Mbytes/sec
            0         1000         0.91         0.92         0.91         0.00
            1         1000         0.83         0.83         0.83         2.30
            2         1000         0.85         0.85         0.85         4.49
            4         1000         0.91         0.91         0.91         8.38
            8         1000         0.86         0.86         0.86        17.74
           16         1000         0.93         0.93         0.93        32.81
           32         1000         0.92         0.92         0.92        66.10
           64         1000         1.27         1.27         1.27        96.15
          128         1000         1.29         1.30         1.30       187.86
          256         1000         1.37         1.37         1.37       355.49
          512         1000         1.43         1.43         1.43       680.62
         1024         1000         1.79         1.80         1.80      1084.75
         2048         1000         2.16         2.17         2.17      1799.45
         4096         1000         5.92         6.03         5.97      1296.66
         8192         1000        10.85        11.06        10.96      1412.63
        16384         1000        13.64        13.75        13.69      2272.28
        32768         1000        20.50        20.67        20.58      3023.47
        65536          640        46.51        47.27        46.88      2644.31
       131072          320        78.51        80.45        79.35      3107.58
       262144          160       138.43       143.22       141.19      3491.25
       524288           80       267.61       289.53       280.73      3453.88
      1048576           40       838.38       950.84       902.65      2103.40
      2097152           20      2068.21      2383.10      2237.95      1678.49
      4194304           10      4017.69      4691.91      4418.00      1705.06

#-----------------------------------------------------------------------------
# Benchmarking Sendrecv
# #processes = 64
#-----------------------------------------------------------------------------
       #bytes #repetitions  t_min[usec]  t_max[usec]  t_avg[usec]   Mbytes/sec
            0         1000         0.86         0.87         0.87         0.00
            1         1000         0.88         0.88         0.88         2.17
            2         1000         0.88         0.89         0.89         4.30
            4         1000         0.89         0.89         0.89         8.55
            8         1000         0.96         0.96         0.96        15.83
           16         1000         0.96         0.96         0.96        31.72
           32         1000         0.95         0.95         0.95        64.13
           64         1000         1.30         1.30         1.30        93.64
          128         1000         1.31         1.32         1.32       185.00
          256         1000         1.37         1.38         1.37       354.63
          512         1000         1.47         1.48         1.47       659.79
         1024         1000         1.72         1.73         1.72      1127.91
         2048         1000         2.25         2.28         2.27      1716.68
         4096         1000         5.86         5.98         5.92      1306.96
         8192         1000        10.94        11.20        11.08      1395.48
        16384         1000        13.88        14.04        13.96      2226.28
        32768         1000        20.67        20.88        20.78      2992.85
        65536          640        46.52        47.68        47.11      2621.73
       131072          320        82.01        87.33        84.75      2862.71
       262144          160       144.36       152.16       148.97      3286.07
       524288           80       279.46       341.05       311.30      2932.08
      1048576           40       800.67      1265.65      1063.24      1580.21
      2097152           20      1868.83      2951.74      2432.82      1355.13
      4194304           10      3663.21      5771.76      4784.25      1386.06

#-----------------------------------------------------------------------------
# Benchmarking Exchange
# #processes = 32
# ( 32 additional processes waiting in MPI_Barrier)
#-----------------------------------------------------------------------------
       #bytes #repetitions  t_min[usec]  t_max[usec]  t_avg[usec]   Mbytes/sec
            0         1000         2.39         2.41         2.40         0.00
            1         1000         2.41         2.42         2.41         1.58
            2         1000         2.41         2.43         2.42         3.14
            4         1000         2.46         2.46         2.46         6.19
            8         1000         2.44         2.46         2.45        12.42
           16         1000         2.67         2.68         2.68        22.74
           32         1000         2.78         2.79         2.79        43.72
           64         1000         3.01         3.01         3.01        80.98
          128         1000         3.04         3.05         3.05       160.05
          256         1000         3.08         3.09         3.09       316.00
          512         1000         3.44         3.45         3.45       565.59
         1024         1000         4.34         4.36         4.35       895.10
         2048         1000         5.08         5.11         5.09      1530.14
         4096         1000         8.72         8.78         8.76      1779.18
         8192         1000        16.48        16.60        16.54      1882.27
        16384         1000        25.67        25.81        25.75      2421.30
        32768         1000        44.82        45.06        44.95      2773.95
        65536          640       101.19       102.16       101.70      2447.14
       131072          320       177.38       179.38       178.38      2787.39
       262144          160       353.80       367.27       361.30      2722.82
       524288           80       661.26       716.57       692.42      2791.07
      1048576           40      1797.63      1963.59      1904.32      2037.09
      2097152           20      3882.30      4453.33      4219.29      1796.41
      4194304           10      7798.39      9182.05      8677.03      1742.53

#-----------------------------------------------------------------------------
# Benchmarking Exchange
# #processes = 64
#-----------------------------------------------------------------------------
       #bytes #repetitions  t_min[usec]  t_max[usec]  t_avg[usec]   Mbytes/sec
            0         1000         2.49         2.52         2.50         0.00
            1         1000         2.60         2.63         2.61         1.45
            2         1000         2.51         2.54         2.52         3.01
            4         1000         2.54         2.57         2.56         5.94
            8         1000         2.53         2.56         2.54        11.92
           16         1000         2.83         2.87         2.85        21.29
           32         1000         2.86         2.90         2.88        42.13
           64         1000         3.06         3.09         3.08        78.93
          128         1000         3.06         3.09         3.08       157.87
          256         1000         3.24         3.27         3.25       298.67
          512         1000         3.60         3.63         3.61       537.50
         1024         1000         4.30         4.36         4.33       896.53
         2048         1000         5.58         5.66         5.62      1380.98
         4096         1000         9.83        10.00         9.91      1562.99
         8192         1000        18.30        18.67        18.49      1673.57
        16384         1000        26.19        26.47        26.33      2361.36
        32768         1000        45.36        45.90        45.64      2723.32
        65536          640       110.48       112.92       111.70      2214.03
       131072          320       194.86       200.24       197.72      2496.99
       262144          160       349.92       379.18       365.88      2637.24
       524288           80       659.13       768.30       714.57      2603.16
      1048576           40      1865.42      2547.63      2253.23      1570.08
      2097152           20      3838.92      5461.48      4724.78      1464.80
      4194304           10      7891.15     11023.14      9612.66      1451.49

#----------------------------------------------------------------
# Benchmarking Allgather
# #processes = 32
# ( 32 additional processes waiting in MPI_Barrier)
#----------------------------------------------------------------
       #bytes #repetitions  t_min[usec]  t_max[usec]  t_avg[usec]
            0         1000         7.41         7.41         7.41
            1         1000         8.51         8.51         8.51
            2         1000         8.58         8.58         8.58
            4         1000         8.73         8.73         8.73
            8         1000         8.22         8.23         8.22
           16         1000         9.02         9.02         9.02
           32         1000        11.12        11.13        11.12
           64         1000        16.43        16.44        16.43
          128         1000        26.53        26.54        26.53
          256         1000        14.72        14.72        14.72
          512         1000        70.62        70.65        70.64
         1024         1000       106.61       106.67       106.65
         2048         1000       200.51       200.62       200.59
         4096         1000       397.34       397.62       397.54
         8192         1000       774.81       775.41       775.31
        16384         1000      1483.46      1484.34      1484.17
        32768         1000      2923.70      2924.14      2924.04
        65536          640      5771.29      5772.57      5772.29
       131072          320     12059.73     12062.62     12061.84
       262144          160      6444.46      6453.89      6449.36
       524288           80     12509.90     12547.59     12530.23
      1048576           40     33248.16     33395.09     33333.91
      2097152           20     69397.12     70215.03     69884.13
      4194304           10    139260.58    142329.05    141136.73

#----------------------------------------------------------------
# Benchmarking Allgather
# #processes = 64
#----------------------------------------------------------------
       #bytes #repetitions  t_min[usec]  t_max[usec]  t_avg[usec]
            0         1000         9.49         9.50         9.49
            1         1000        11.00        11.00        11.00
            2         1000        11.22        11.23        11.23
            4         1000        11.41        11.41        11.41
            8         1000        11.35        11.36        11.36
           16         1000        13.30        13.30        13.30
           32         1000        19.65        19.66        19.66
           64         1000        32.69        32.71        32.70
          128         1000        55.62        55.63        55.63
          256         1000        24.56        24.57        24.56
          512         1000       156.83       156.91       156.88
         1024         1000       277.38       277.52       277.48
         2048         1000       545.60       545.91       545.82
         4096         1000      1095.89      1096.48      1096.36
         8192         1000      2143.29      2144.26      2144.02
        16384         1000      4528.70      4531.52      4531.04
        32768         1000      8255.19      8256.55      8256.32
        65536          608     16484.32     16485.69     16485.26
       131072          299     33429.64     33436.16     33432.65
       262144          160     12836.32     12855.13     12846.90
       524288           80     25924.97     25997.99     25966.34
      1048576           40     77733.41     78271.51     78039.95
      2097152           20    155834.90    158187.26    157153.98
      4194304           10    317781.28    328006.60    323618.12


# All processes entering MPI_Finalize

</pre>

