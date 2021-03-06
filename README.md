# Packstack

Utility to install **OpenStack** on **Red Hat** based operating system. See
other branches for older **OpenStack** versions. Details on how to
contribute to **Packstack** may be found in the **Packstack** wiki at
<https://wiki.openstack.org/wiki/Packstack> Additional information
about involvement in the community around **Packstack** can be found at
<https://openstack.redhat.com/Get_involved>


This utility can be used to install **OpenStack** on a single or group of
hosts (over `ssh`).

This utility is still in the early stages, a lot of the configuration
options have yet to be added.

## Installation of packstack:

    $ yum install -y git
    $ git clone git://github.com/stackforge/packstack.git
    $ cd packstack && sudo python setup.py install

## Installation of openstack-puppet-modules (REQUIRED if running packstack from source):

    $ sudo python setup.py install_puppet_modules

### Option 1 (using answer file)

    $ packstack --gen-answer-file=ans.txt

Then edit `ans.txt` as appropriate e.g.

- set `CONFIG_SSH_KEY` to a public ssh key to be installed to remote machines
- Edit the IP address to anywhere you want to install a piece of OpenStack on another server
- Edit the 3 network interfaces to whatever makes sense in your setup

you'll need to use a icehouse repository for example for RHEL

    $ CONFIG_REPO=http://repos.fedorapeople.org/repos/openstack/openstack-icehouse/epel-6/

    $ packstack --answer-file=ans.txt

### Option 2 (prompts for configuration options)

    $ packstack

that's it, if everything went well you can now start using OpenStack

    $ cd
    $ . keystonerc_admin
    $ nova list
    $ swift list  # if you have installed swift

## Config options

- `CONFIG_NOVA_COMPUTE_HOSTS` :
  * A comma separated list of ip addresses on which to install nova compute
- `CONFIG_SWIFT_STORAGE_HOSTS` :
  * A comma separated list of swift storage devices
    * `1.1.1.1`: create a testing loopback device and use this for storage
    * `1.1.1.1/sdb`: use `/dev/sdb` on `1.1.1.1` as a storage device

## Logging

The location of the log files and generated puppet manifests are in the
`/var/tmp/packstack` directory under a directory named by the date in which
**Packstack** was run and a random string (e.g. `/var/tmp/packstack/20131022-204316-Bf3Ek2`).
Inside, we find a manifest directory and the `openstack-setup.log` file; puppet
manifests and a log file for each one are found inside the manifest directory.

## Debugging

To make **Packstack** write more detailed information into the log file you can use the `-d` switch:

    $ packstack -d --allinone

## Developing

**Warning:**
this procedure installs **openstack-puppet-modules** containing code that has
not been upstreamed and fully tested yet and as such will not be as robust as
the other install procedures.  It is recommended to install from **RPM**
instead.

To ease development of **Packstack** and **openstack-puppet-modules**, it can be
useful to install from *git* such that updates to the git repositories are
immediately effective without reinstallation of packstack and
**openstack-puppet-modules**.

To do this, start with a minimal **Fedora 21** installation.  Then remove any
relevant packages that might conflict:

    $ yum -y erase openstack-{packstack*,puppet-modules}

Disable **SELinux** by changing "`enforcing`" to "`permissive`" in
`/etc/sysconfig/selinux`, then reboot to allow service changes to take effect
and swap over networking.  Then install packages:

    $ yum -y install git python-setuptools

And install **RDO**:

    $ yum -y install https://rdo.fedorapeople.org/rdo-release.rpm
    $ yum -y update

Now we get **openstack-puppet-modules**.  Because `python setup.py
install_puppet_modules` from **Packstack** copies rather than linking, this is not
entirely straightforward:

    $ git clone https://github.com/redhat-openstack/openstack-puppet-modules
    $ cd openstack-puppet-modules
    $ git checkout master-patches
    $ mkdir /usr/share/openstack-puppet
    $ ln -sv /root/openstack-puppet-modules /usr/share/openstack-puppet/modules

Then we get **Packstack**, and perform a similar dance:

    $ git clone https://github.com/stackforge/packstack
    $ cd packstack
    $ python setup.py develop
    $ cd /usr/share/openstack-puppet/modules
    $ ln -sv /root/packstack/packstack/puppet/modules/packstack
    $ ln -sv /root/packstack/packstack/puppet/modules/remote

And we're done.  Changes to the contents of **Packstack** and
**openstack-puppet-modules** repositories are picked up by the **Packstack**
executable without further intervention, and **Packstack** is ready to install.

## Puppet Style Guide

**IMPORTANT** <https://docs.puppetlabs.com/guides/style_guide.html>

Please, respect the Puppet Style Guide as much as possible !

## Running local Puppet-lint tests

It assumes that both `bundler` as well as `rubygems` (and `ruby`) are already
installed on the system. If not, run this command:

    $ sudo yum install rubygems rubygem-bundler ruby ruby-devel -y

Go into the **Packstack** root directory.

    $ cd packstack/

A `Rakefile` contains all you need to run puppet-lint task automatically over
all the puppet manifests included in the **Packstack** project.

    $ ls -l packstack/puppet/templates/

and

    $ ls -l packstack/puppet/modules/

The default puppet-lint pattern for `.pp` files is `**/*.pp`. So there is no
need to go inside those directories to run puppet-lint !

    $ mkdir vendor
    $ export GEM_HOME=vendor
    $ bundle install
    $ bundle exec rake lint
