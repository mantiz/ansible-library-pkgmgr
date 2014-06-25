# Ansible Library: PkgMgr

This library aims to unify package management tasks.

## Motivation

I wrote the plugin because I don't like to have a task list like the following:

    - name: install apache ant (pacman)
      pacman: pkg=apache-ant state=present
      when: ansible_pkg_mgr == 'pacman'

    - name: install apache ant (apt)
      apt: pkg=ant state=present
      when: ansible_pkg_mgr == 'apt'

The result of this is that I have a skipping task on each run and it gets worse if more package managers are involved.

I couldn't find a solution how to handle such things properly in Ansible, so I decided to write my own little package manager.

Currently only `pacman` and `apt` are supported because these are the package managers used on the systems I work with but more package managers will be added over time. Feel free to pull request support for more package managers and/or features.

## Usage

This plugin allows the following:

    - name: install apache ant
      pkgmgr: pkg=apache-ant state=present

And we are done.

Well, not really, you may have noticed that the name of the package differs across different distributions (or even versions of the same distro).
To solve this issue, I wrote a [lookup plugin](https://github.com/mantiz/ansible-lookup-plugin-os-specific).
Put these pieces together and you will end up with this:

    - name: install apache ant
      pkgmgr: pkg={{ item }} state=present
      with_os_specific:
          - default: ant
            Archlinux: apache-ant

Now, the package `ant` will be installed on each distribution but for `Archlinux` `apache-ant` will be taken for the name of the package.

You may also use this technique to install some common base packages:

    - name: install common base packages
      pkgmgr: pkg={{ item }} state=present
      with_os_specific:
          - bash-completion
          - default: ant
            Archlinux: apache-ant
          - Archlinux: yaourt
          - Debian: put-what-ever-you-want-here

This will install `bash-completion` on each system, `apache-ant` on `Archlinux` and `ant` on all other systems, `yaourt` on `Archlinux` and `put-what-ever-you-want-here` on `Debian`.

So, I really recommend when using this library then use it together with the [os specific lookup plugin](https://github.com/mantiz/ansible-lookup-plugin-os-specific) to get the full potential out of it.

## Installation

Simply clone this repository and create a symlink inside a directory `library` in the same directory of your playbook.
If you are using git for your playbook, for example, you can add this as a submodule:

    git submodule add https://github.com/mantiz/ansible-library-pkgmgr external/library-pkgmgr
    mkdir library
    ln -s ../external/library-pkgmgr library
