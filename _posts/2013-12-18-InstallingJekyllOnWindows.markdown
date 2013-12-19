---
layout: post
title:  "Installing Jekyll On Windows"
date:   2013-12-18
categories: Ruby Jekyll Windows 
---

Installing [Jekyll](http://jekyllrb.com/) on windows is a little weird.  When I first tried it, I kept stubbing my toes and it took a little too long for my liking.  I wanted to use this post as an outline of the steps needed to get Jekyll up and running.

At the time of this writing:

- Ruby Version: 1.9.3-p484
- Jekyll Version: 1.3.1

**Install Ruby**

First, we need to get ruby on our machine.  This is actually really easy thanks to [RubyInstaller.org](http://rubyinstaller.org/).  An organization dedicated to creating windows versions of ruby that you can easily install with one of the [installers](http://rubyinstaller.org/downloads/).

I chose to install version [1.9.3-p484](http://dl.bintray.com/oneclick/rubyinstaller/rubyinstaller-1.9.3-p484.exe?direct).

*Note:* Make sure that you install Ruby in a location without spaces in the path.  For example:

- C:\Ruby (Good)
- C:\Test Location\Ruby (Bad)

*Note* There is an option within the installer to add Ruby to the *path* environment variable.  Do this.  Otherwise, make sure you add the location of *ruby.exe* to your *path* environment variable.

**Install Ruby DevKit**

*This portion of this walk through takes pieces from the walk through on [GitHub](https://github.com/oneclick/rubyinstaller/wiki/Development-Kit)*

Second, we need the Ruby Development Kit, that you can get from the same download page as ruby.

You need to make sure that you get the appropriate kit for your particular version of ruby.  If you installed version 1.9.3-p484, then you need to install DevKit Version [tdm-32-4.5.2](https://github.com/downloads/oneclick/rubyinstaller/DevKit-tdm-32-4.5.2-20111229-1559-sfx.exe)

*Note:* Make sure that you install the DevKit in a location without spaces in the path.  For example:

- C:\DevKit (Good)
- C:\Dev Kit (Bad)

Once installed, navigate to the location the DevKit was installed.

Within there, execute the following into the console:

```console
ruby dk.rb init
```

This command will generate a *config.yml* file within the working directory.  This file is used to determine which installation of ruby you want to install the DevKit into.  Just in case you have multiple versions of ruby installed on your machine.  If you open the file, you'll see some instructions on how to make sure that the version of ruby you installed in the previous step is listed in the *config.yml* file.  Make sure that is the case then execute the following command:

```console
ruby dk.rb install
```

**Install Jekyll**

Once we have Ruby and it's DevKit installed, installing Jekyll is easy.  Simply type the following into the console:

```console
gem install jekyll
```

This will install a lot of prerequisites before it gets to the actual Jekyll gem.

**The Surgery**

This step is where I kept tripping up.  The *Jekyll* gem relies on the *Pygments.rb* gem to do syntax highlighting in code blocks.  However, the version of *Pygments.rb* that gets installed is incompatible with Windows.  (As far as I can tell anyways.)  So, we simply need to uninstall the version that is on your machine, and install the correct version.

Type the following into your console:

```console
gem list --local
```

This command will list out all of the gems that you have installed on your machine.  Find and note the version of the *Pygments.rb* gem.  You'll need it in the next command.

```console
gem uninstall Pygments.rb --version "=(version)"
```

So, if the version you have installed is 0.5.4, then the command would become:

```console
gem uninstall Pygments.rb --version "=0.5.4"
```

The version of *Pygments.rb* that will work with Windows is version 0.5.0.  So, let's install it with the next command.

```console
gem install Pygments.rb --version "=0.5.0"
```

Once it's installed you should be able to start using Jekyll.  Refer to the [Jekyll](http://jekyllrb.com/) documentation on how to start you're own Jekyll static website.
