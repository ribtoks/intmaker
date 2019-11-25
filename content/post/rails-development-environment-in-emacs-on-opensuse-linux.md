---
title: Rails development environment in Emacs on OpenSuSE Linux
date: 2013-10-29T23:12:15+00:00
author: "Taras Kushnir"
permalink: /rails-development-environment-in-emacs-on-opensuse-linux/
categories:
  - Linux
keywords:
  - config.el
  - Emacs
  - list
  - rails
  - ruby
---
Today we'll set up a complete Ruby On Rails development environment on Linux. For an IDE we'll use Emacs and for host system - OpenSUSE.

Let's install latest Ruby+Rails bundle before configuring Emacs. You can refer to <a title="Rails on OpenSUSE 12.1" href="http://alphacluster.wordpress.com/2012/03/29/rails-on-opensuse-12-1/" target="_blank">nice article</a> on that. In short, you need get RVM and follow through script steps in terminal:

<pre><span style="color: #0000ff;"><span style="color: #000000;">></span> bash</span> -s stable < <(<span style="color: #0000ff;">curl</span> -s <span style="color: #808000;">https://raw.github.com/wayneeseguin/rvm/master/binscripts/rvm-installer</span>)</pre>

Then load path to installed RVM in current bash session:

<pre><span style="color: #000000;">></span> . ~/.bash_profile</pre>

It's a bit frustrating that if bash finds a _.bash_profile_ file, it ignores _.bashrc_, so be careful. Usually, I just move this one line from _.bash_profile_ to _.profile_ or _.bashrc_.

Then check requirements and install everything what is needed

<pre>> rvm requirements</pre>

Now let's set ruby version to 2.0 and install rails (4):

<pre>> rvm install 2.0
> rvm use 2.0
> gem install rails
> gem install sqlite3</pre>

Now, when everything is ready, we can configure Emacs installation. I assume you've already installed emacs package via your favorite package manager.
  
Then lets add some custom configuration file for ruby configs in the _.emacs.d_ directory, say _ruby-configuration.el_ and add it to your .emacs file using

<pre>(<span style="color: #0000ff;">load</span> <span style="color: #339966;">"~/.emacs.d/ruby-configuration.el"</span>)</pre>

<!--more-->

I use next packages for Emacs + Ruby + Rails development:

  * rvm.el
  * rsense + ri + rurema
  * flymake
  * rinari
  * inf-ruby
  * web-mode
  * rhtml
  * ruby-compilation
  * yaml-mode

You can easily google how to install and use them. I can only point at some interesting moments. First, you can set some git submodules, as I did:

<pre>[submodule <span style="color: #800000;">".emacs.d/web-mode"</span>]
    path = .emacs.d/web-mode
    url = <span style="color: #008000;">https://github.com/fxbois/web-mode</span>
[submodule <span style="color: #800000;">".emacs.d/rvm"</span>]
    path = .emacs.d/rvm
    url = <span style="color: #008000;">http://github.com/djwhitt/rvm.el.git</span>
[submodule <span style="color: #800000;">".emacs.d/rhtml"</span>]
    path = .emacs.d/rhtml
    url = <span style="color: #008000;">https://github.com/eschulte/rhtml.git</span>
[submodule <span style="color: #800000;">".emacs.d/yaml-mode"</span>]
    path = .emacs.d/yaml-mode
    url = <span style="color: #008000;">http://github.com/yoshiki/yaml-mode.git</span>
[submodule <span style="color: #800000;">".emacs.d/inf-ruby"</span>]
    path = .emacs.d/inf-ruby
    url = <span style="color: #008000;">https://github.com/nonsequitur/inf-ruby</span>
[submodule <span style="color: #800000;">".emacs.d/rinari"</span>]
    path = .emacs.d/rinari
    url = <span style="color: #008000;">https://github.com/eschulte/rinari.git</span>
[submodule <span style="color: #800000;">".emacs.d/fold-dwim"</span>]
    path = .emacs.d/fold-dwim
    url = <span style="color: #008000;">https://github.com/emacsmirror/fold-dwim</span></pre>

Do not forget to add appropriate directories to your load path using function

<pre>(<span style="color: #0000ff;">add-to-list</span> <span style="color: #993366;">'load-path</span> <span style="color: #339966;">"your/directory/with/submodule"</span>)</pre>

My other configuration are mostly copy-and-pasted from EmacsWiki for those packages I mentioned in a list of my Ruby packages. You can find <a href="https://github.com/Ribtoks/configs/blob/master/.emacs.d/elisp/ruby-config.el" target="_blank">my configuration at GitHub</a>.

Links:

  * <a href="http://viget.com/extend/emacs-24-rails-development-environment-from-scratch-to-productive-in-5-minu" target="_blank">Emacs 24 Rails Development Environment - From *scratch* to Productive in 5 Minutes</a>
  * <a href="http://www.emacswiki.org/emacs/RubyOnRails" target="_blank">EmacsWiki: RubyOnRails</a>
