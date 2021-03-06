---
layout: documentation
title: Documentation - Provisioners - Puppet
---
# Puppet Provisioning

[Puppet](http://www.puppetlabs.com/puppet) allows you to provision your virtual machines with Puppet manifests.  It
runs in a stand-alone mode that doesn't require a Puppet server.

This page will not go into the details of creating Puppet manifests, since that
is covered in detail around the web, but a good place to start is the [Puppet Labs Docs site](http://docs.puppetlabs.com).

## Setting the Manifests Path

First, Vagrant needs to know where the Puppet manifests are located. By default, Vagrant will
look in the `manifests` directory relative to the root of the project directory (where
the project's Vagrantfile is located). The `manifests` should contain a file for each box to be configured,
the default assumes a file named after the value of config.vm.box option:

Both the file and path can be configured using the `manifest_file` and `manifests_path` options in your
Vagrantfile configuration file, for example:

{% highlight ruby %}
Vagrant::Config.run do |config|
  config.vm.provision :puppet do |puppet|
    puppet.manifests_path = "puppetmanifests"
    puppet.manifest_file = "newbox.pp"
  end
end
{% endhighlight %}

## Configuring the manifests

Creating a basic manifest is simple, for example:

{% highlight ruby %}
# Basic Puppet manifests

class lucid32 {
  package { "apache2":
    ensure => present,
  }

  service { "apache2":
    ensure => running,
    require => Package["apache2"],
  }
}

include lucid32
{% endhighlight %}

## Modules

[Modules](http://docs.puppetlabs.com/guides/modules.html) provide a way to encapsulate
Puppet files (resources, definitions, classes, etc.) into a single redistributable
package.

The Vagrant Puppet provisioner allows you to mount a local folder of modules
onto the VM, and will configure Puppet to be aware of them, automatically.

For example, if you have a folder named "my_modules" with a bunch of modules
in the same folder as your Vagrantfile, you can mount them like so:

{% highlight ruby %}
Vagrant::Config.run do |config|
  config.vm.provision :puppet, :module_path => "my_modules"
end
{% endhighlight %}

The module path is expanded relative to the folder containing the Vagrantfile.
You may of course also put absolute paths in place. You may also specify an array
of module folders.

Now, let's say there is an "apache" module in your "my_modules" folder. You can
now use it in your manifest file:

{% highlight ruby %}
include apache
{% endhighlight %}

When the provisioner runs, it will automatically mount your modules folder and
configure Puppet to know where they are so they can be loaded.

## Configuring the Puppet Provisioning Path

In order to run Puppet, Vagrant has to mount the specified manifests directory as a
shared folder on the virtual machine. By default, this is set to be `/tmp/vagrant-puppet`,
and this should be fine for most users. But in the case that you need to customize
the location, you can do so in the Vagrantfile:

{% highlight ruby %}
Vagrant::Config.run do |config|
  config.vm.provision :puppet, :pp_path = "/tmp/vagrant-puppet"
end
{% endhighlight %}

This folder is created for provisioning purposes and destroyed once provisioning
is complete.

## Setting Additional Options

You can also specify additional options to be passed to Puppet using the `options` variable.

{% highlight ruby %}
Vagrant::Config.run do |config|
  config.vm.provision :puppet, :options => ["--modulepath","modules"]
end
{% endhighlight %}

You can also pass options as strings:

{% highlight ruby %}
  config.vm.provision :puppet, :options => "--verbose --debug"
{% endhighlight %}

## Enabling and Executing

If you are building a VM from scratch, run `vagrant up` and provisioning
will automatically occur. If you already have a running VM and don't want to rebuild
everything from scratch, run `vagrant reload` and it will restart the VM, without completely
destroying the environment first, allowing the import step to be skipped.
