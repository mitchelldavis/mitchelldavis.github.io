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

{% highlight bash %}
ruby dk.rb init
{% endhighlight %}

This command will generate a *config.yml* file within the working directory.  This file is used to determine which installation of ruby you want to install the DevKit into.  Just in case you have multiple versions of ruby installed on your machine.  If you open the file, you'll see some instructions on how to make sure that the version of ruby you installed in the previous step is listed in the *config.yml* file.  Make sure that is the case then execute the following command:

{% highlight bash %}
ruby dk.rb install
{% endhighlight %}

**Install Jekyll**

Once we have Ruby and it's DevKit installed, installing Jekyll is easy.  Simply type the following into the console:

{% highlight bash %}
gem install jekyll
{% endhighlight %}

This will install a lot of prerequisites before it gets to the actual Jekyll gem.

**The Surgery**

This step is where I kept tripping up.  The *Jekyll* gem relies on the *Pygments.rb* gem to do syntax highlighting in code blocks.  However, the version of *Pygments.rb* that gets installed is incompatible with Windows.  (As far as I can tell anyways.)  So, we simply need to uninstall the version that is on your machine, and install the correct version.

Type the following into your console:

{% highlight bash %}
gem list --local
{% endhighlight %}

This command will list out all of the gems that you have installed on your machine.  Find and note the version of the *Pygments.rb* gem.  You'll need it in the next command.

{% highlight bash %}
gem uninstall Pygments.rb --version "=(version)"
{% endhighlight %}

So, if the version you have installed is 0.5.4, then the command would become:

{% highlight bash %}
gem uninstall Pygments.rb --version "=0.5.4"
{% endhighlight %}

The version of *Pygments.rb* that will work with Windows is version 0.5.0.  So, let's install it with the next command.

{% highlight bash %}
gem install Pygments.rb --version "=0.5.0"
{% endhighlight %}

Once it's installed you should be able to start using Jekyll.  Refer to the [Jekyll](http://jekyllrb.com/) documentation on how to start you're own Jekyll static website.

**UPDATE**

Well, I was wrong.  I thought I covered everything but it turns out we need [Python](http://www.python.org) as well.

In order to do any syntax highlighting *Pygments.rb* calls *Python's Pygments* which does the actual highlighting.

Download and install [Python Version 2.7.6](http://www.python.org/download/releases/2.7.6/).  Once installed, make sure that the path to the python executable has been added to your machine's *Path* environment variable.

Copy the contents of [this (ez_install.py)](https://bitbucket.org/pypa/setuptools/raw/bootstrap/ez_setup.py) file onto you file system. If you name it *ez_install.py* then run:

{% highlight bash %}
python ez_install.py
{% endhighlight %}

That command will install python's [Setup Tools](https://pypi.python.org/pypi/setuptools) and place some executable's within the *Scripts* directory in your python root directory.  Navigate to that directory now.  Once there run the following command:

{% highlight bash %}
easy_install Pygments
{% endhighlight %}

That command installs python's version of *Pygments* which, as stated earlier, is what *Pygments.rb* calls to do syntax highlighting.

My bad about forgetting the python parts of this walkthrough.  As far as I can tell, you can still use Jekyll without *Python*.  However, once you start using syntax highlighting, you'll start seeing errors.

**Note:**

I noticed that if you try and use the *--watch* switch on the jekyll command line, it will error.  Hmmm.... If I figure that one out, I'll post the resolution here.

Happy Blogging.

