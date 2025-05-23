---
layout: default
title: Feature container for bootstrap
---

## Container image for bootstrap

In past, we had some incompatibilities between host and build target. They were, in fact, small. Like using a different package manager. Some were big. Like, the introduction of Weak and Rich dependencies. For this reason, we introduced [bootstrap](Feature-bootstrap). But then comes [zstd payload](https://fedoraproject.org/wiki/Changes/Switch_RPMs_to_zstd_compression). This is a new type of payload. And to install packages with this payload, you need an `rpm` binary, which supports this payload. This is true for all current Fedoras. Unfortunately, neither RHEL 8 nor RHEL 7 supports this payload. So even bootstrap will not help you to build Fedora packages on RHEL 8.

We come up with a nice feature. Mock will not install bootstrap chroot itself. Instead, it will download the container image, extract the image, and use this extracted directory as a bootstrap chroot. And from this bootstrapped chroot install the final one.

Using this feature, **any** incompatible feature in either RPM or DNF can be used in the target chroot. Now or in future. And you will be able to install the final chroot. You do not even need to have `rpm` on a host. So this should work on any system, even Debian-based ones. The only requirement for this feature is [Podman](https://podman.io/). Do not forget to install the `podman` package.

This feature is available since 1.4.20 and enabled by default from 5.0. You can disable it using:

    config_opts['use_bootstrap_image'] = False

It can be enabled or disabled on the command line using `--use-bootstrap-image` or `--no-bootstrap-image` options.

Note however that also this is prerequisite:

    config_opts['use_bootstrap_container'] = True # or --bootstrap-chroot option

To specify which image should be used for bootstrap container you can put in config:

    config_opts['bootstrap_image'] = 'registry.fedoraproject.org/fedora:latest'

This is a general config. Each config has specified its own image specified. E.g. CentOS 7 has `config_opts['bootstrap_image'] = 'centos:7'` in config. So unless you use your own config, you can enable this feature, and the right image will be used.

The image contents are typically suboptimal for Mock's use-case.  In particular,
Mock needs to have a correct package manager (as specified in the
`package_manager` configuration option) installed inside the image, along with
the `builddep` functionality (typically provided by `python3-dnf-plugins-core`).
This is why Mock still has to 'update the downloaded bootstrap' somehow.  If you
happen to have an image with `builddep` pre-installed, you can set
`bootstrap_image_ready` to 'True':

    config_opts['bootstrap_image_ready'] = True

This option will significantly reduce the bootstrap preparation time, as no
package management actions need to be performed for the bootstrap (no need to
download and initialize package manager caches).

There is one known issue:

 * Neither Mageia 6 nor 7 works correctly now with this feature.

Technically, you can use any container, as long as there is the required package manager (DNF or YUM). The rest of the needed packages will be installed by mock.
