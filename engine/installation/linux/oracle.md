---
description: Instructions for installing Docker on Oracle Linux
keywords: Docker, Docker documentation, requirements, installation, oracle, ol, rpm, install, uninstall, upgrade, update
redirect_from:
- /engine/installation/oracle/
title: Get Docker for Oracle Linux
---

To get started with Docker on Oracle Linux, make sure you
[meet the prerequisites](#prerequisites), then
[install Docker](#install-docker).

## Prerequisites

### OS requirements

To install Docker, you need the 64-bit version of Oracle Linux 6 or 7.

To use `btrfs`, you need to install the Unbreakable Enterprise Kernel (UEK)
version 4.1.12 or higher. For Oracle Linux 6, you need to enable extra repositories
to install UEK4. See
[Obtaining and installing the UEK packages](https://docs.oracle.com/cd/E37670_01/E37355/html/ol_obtain_uek.html){: target="_blank" class="_" }.

### Remove unofficial Docker packages

Oracle's repositories contain an older version of Docker, with the package name
`docker` instead of `docker-engine`. If you installed this version of Docker,
remove it using the following command:

```bash
$ sudo yum -y remove docker
```

You may also have to remove the package `docker-engine-selinux` which conflicts with
the official `docker-engine` package.  Remove it with the following command:

```bash
$ sudo yum -y remove docker-engine-selinux
```

The contents of `/var/lib/docker` are not removed, so any images, containers,
or volumes you created using the older version of Docker are preserved.

## Install Docker

You can install Docker in different ways, depending on your needs:

- Most users
  [set up the official Docker repositories](#install-using-the-repository) and
  install from them, for ease of installation and upgrade tasks. This is the
  recommended approach.

- Some users download the RPM package and install it manually and manage
  upgrades completely manually.

- Some users cannot use third-party repositories, and must rely on
  the version of Docker in the Oracle repositories. This version of Docker may
  be out of date. Those users should consult the Oracle documentation and not
  follow these procedures.

### Install using the repository

Before you install Docker for the first time on a new host machine, you need to
set up the Docker repository. Afterward, you can install, update, or downgrade
Docker from the repository.

#### Set up the repository

1.  Install the `yum-utils` plugin, which provides the `yum-config-manager`
    plugin.

    ```bash
    $ sudo yum install -y yum-utils
    ```

2.  Use one of the following commands to set up the **stable** repository,
    depending on your version of Oracle Linux:

    **Oracle Linux 7**:

    ```bash
    $ sudo yum-config-manager \
        --add-repo \
        https://docs.docker.com/engine/installation/linux/repo_files/oracle/docker-ol7.repo
    ```

    **Oracle Linux 6**:

    ```bash
    $ sudo yum-config-manager \
        --add-repo \
        https://docs.docker.com/engine/installation/linux/repo_files/oracle/docker-ol6.repo
    ```

3.  **Optional**: Enable the **testing** repository. This repository is included
    in the `docker.repo` file above but is disabled by default. You can enable
    it alongside the stable repository. Do not use unstable repositories on
    on production systems or for non-testing workloads.

    > **Warning**: If you have both stable and unstable repositories enabled,
    > installing or updating without specifying a version in the `yum install`
    > or `yum update` command will always install the highest possible version,
    > which will almost certainly be an unstable one.

    ```bash
    $ sudo yum-config-manager --enablerepo docker-testing
    ```

    You can disable the `testing` repository by running the `yum-config-manager`
    command with the `--disablerepo` flag. To re-enable it, use the
    `--set-enabled` flag. The following command disables the `testing`
    repository.

    ```bash
    $ sudo yum-config-manager --disablerepo docker-testing
    ```

#### Install Docker

1.  Update the `yum` package index.

    ```bash
    $ sudo yum makecache fast
    ```

    If this is the first time you have refreshed the package index since adding
    the Docker repositories, you will be prompted to accept the GPG key, and
    the key's fingerprint will be shown. Verify that the fingerprint matches
    `58118E89F3A912897C070ADBF76221572C52609D` and if so, accept the key.

2.  Install the latest version of Docker, or go to the next step to install a
    specific version.

    ```bash
    $ sudo yum -y install docker-engine
    ```

    > **Warning**: If you have both stable and unstable repositories enabled,
    > installing or updating Docker without specifying a version in the
    > `yum install` or `yum upgrade` command will always install the highest
    > available version, which will almost certainly be an unstable one.

3.  On production systems, you should install a specific version of Docker
    instead of always using the latest. List the available versions.
    This example uses the `sort -r` command to sort the results by version
    number, highest to lowest. The output is truncated.

    > **Note**: This `yum list` command only shows binary packages. To show
    > source packages as well, omit the `.x86_64` from the package name.

    ```bash
    $ yum list docker-engine.x86_64  --showduplicates |sort -nr

    docker-engine.x86_64  1.13.0-1.el6                                docker-main   
    docker-engine.x86_64  1.12.3-1.el6                                docker-main   
    docker-engine.x86_64  1.12.2-1.el6                                docker-main   
    docker-engine.x86_64  1.12.1-1.el6                                docker-main    
    ```

    The contents of the list depend upon which repositories you have enabled,
    and will be specific to your version of Oracle Linux (indicated by the
    `.el7` suffix on the version, in this example). Choose a specific version to
    install. The second column is the version string. The third column is the
    repository name, which indicates which repository the package is from and by
    extension extension its stability level. To install a specific version,
    append the version string to the package name and separate them by a hyphen
    (`-`):

    ```bash
    $ sudo yum -y install docker-engine-<VERSION_STRING>
    ```

    The Docker daemon does not start automatically.

4.  Start the Docker daemon. Use `systemctl` on Oracle Linux 7 or `service` on
    Oracle Linux 6.

    **Oracle Linux 7**:

    ```bash
    $ sudo systemctl start docker
    ```

    **Oracle Linux 6**:

    ```bash
    $ sudo service docker start
    ```

5.  Verify that `docker` is installed correctly by running the `hello-world`
    image.

    ```bash
    $ sudo docker run hello-world
    ```

    This command downloads a test image and runs it in a container. When the
    container runs, it prints an informational message and exits.

Docker is installed and running. You need to use `sudo` to run Docker commands.
Continue to [Linux postinstall](linux-postinstall.md) to allow non-privileged
users to run Docker commands and for other optional configuration steps.

#### Upgrade Docker

To upgrade Docker, first run `sudo yum makecache fast`, then follow the
[installation instructions](#install-docker), choosing the new version you want
to install.

### Install from a package

If you cannot use Docker's repository to install Docker, you can download the
`.rpm` file for your release and install it manually. You will need to download
a new file each time you want to upgrade Docker.

1.  Go to [https://yum.dockerproject.org/repo/main/oraclelinux/](https://yum.dockerproject.org/repo/main/oraclelinux/)
    and choose the subdirectory for your Oracle Linux version. Download the
    `.rpm` file for the Docker version you want to install.

    > **Note**: To install a testing version, change the word `main` in the
    > URL to `testing`. Do not use unstable versions of Docker in production
    > or for non-testing workloads.

2.  Install Docker, changing the path below to the path where you downloaded
    the Docker package.

    ```bash
    $ sudo yum install /path/to/package.rpm
    ```

    The Docker daemon does not start automatically.

4.  Start the Docker daemon. Use `systemctl` on Oracle Linux 7 or `service` on
    Oracle Linux 6.

    **Oracle Linux 7**:

    ```bash
    $ sudo systemctl start docker
    ```

    **Oracle Linux 6**:

    ```bash
    $ sudo service docker start
    ```

5.  Verify that `docker` is installed correctly by running the `hello-world`
    image.

    ```bash
    $ sudo docker run hello-world
    ```

    This command downloads a test image and runs it in a container. When the
    container runs, it prints an informational message and exits.

Docker is installed and running. You need to use `sudo` to run Docker commands.
Continue to [Post-installation steps for Linux](linux-postinstall.md) to allow
non-privileged users to run Docker commands and for other optional configuration
steps.

#### Upgrade Docker

To upgrade Docker, download the newer package file and repeat the
[installation procedure](#install-from-a-package), using `yum -y upgrade`
instead of `yum -y install`, and pointing to the new file.

## Uninstall Docker

1.  Uninstall the Docker package:

    ```bash
    $ sudo yum remove docker-engine
    ```

2.  Images, containers, volumes, or customized configuration files on your host
    are not automatically removed. To delete all images, containers, and
    volumes:

    ```bash
    $ sudo rm -rf /var/lib/docker
    ```

    > **Note**: This won't work when the `btrfs` graph driver has been used,
    > because the `rm -rf` command cannot remove the subvolumes that Docker
    > creates. See the output of `man btrfs-subvolume` for information on
    > removing `btrfs` subvolumes.

You must delete any edited configuration files manually.

## Next steps

- Continue to [Post-installation steps for Linux](linux-postinstall.md)

- Continue with the [User Guide](../../userguide/index.md).
