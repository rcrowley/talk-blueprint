!SLIDE bullets

# Blueprint

## Configuration management for busy people

* [rcrowley.org/talks/sf-python-2011-09-14/](http://rcrowley.org/talks/sf-python-2011-09-14/)

!SLIDE bullets

# Hi, I&#8217;m Richard Crowley

* Equal opportunity technology hater.
* DevStructure&#8217;s operator and UNIX hacker.

!SLIDE bullets

# Blueprint

* [github.com/devstructure/blueprint](https://github.com/devstructure/blueprint)
* Debian- or RPM-based Linux.
* Python 2.6 or better.
* BSD licensed.

!SLIDE bullets

# Blueprint

* Figures out what you did.
* Puts it in JSON in Git.
* Generates code.
* Deploys it.

!SLIDE bullets

# Philosophy

!SLIDE bullets

# Philosophy

## Realistic development environments

* Linux.
* Real web servers and databases.

!SLIDE bullets

# Philosophy

## Familiar interactive tools

* `apt-get install gcc`
* Not `package { "gcc": ensure => installed }`

!SLIDE bullets

# Philosophy

## Whole-stack configuration management

* APT, Yum, RubyGems, PyPI, and PEAR.
* `/etc` and `/usr/local`.

!SLIDE bullets

# Philosophy

## Idempotent everything 

* No diffing/patching.
* &#8220;Compliance.&#8221;

!SLIDE bullets

# Philosophy

## Gently encourage The Right Way

* The UNIX Way.
* The Debian Way or The RedHat Way.

!SLIDE bullets

# Philosophy

## Never having to say you&#8217;re sorry

* Reverse-engineer absolute state.
* It doesn&#8217;t matter when you install Blueprint.

!SLIDE bullets

# Philosophy

## Simple file formats

* JSON.
* Tarballs.

!SLIDE bullets

# Philosophy

## UNIX man pages

* [`blueprint-create`(1)](http://devstructure.github.com/blueprint/blueprint-create.1.html)
* [`blueprint`(5)](http://devstructure.github.com/blueprint/blueprint.5.html)

!SLIDE bullets

# Philosophy

## Interoperability

* Puppet and Chef.
* EC2 via user-data.

!SLIDE bullets

# Workflow

!SLIDE bullets

# Workflow

## Start with your production OS

* Vagrant, raw VirtualBox, or the cloud.
* You have `root` access, so use it.

!SLIDE bullets smaller

<pre><strong>$</strong> echo "deb http://packages.devstructure.com lucid main" |
  sudo tee /etc/apt/sources.list.d/devstructure.list
<strong>$</strong> sudo wget -O /etc/apt/trusted.gpg.d/devstructure.gpg \
  http://packages.devstructure.com/keyring.gpg
<strong>$</strong> sudo apt-get update
...
<strong>$</strong> sudo apt-get -y install blueprint
...
<strong>$</strong> blueprint
Usage: blueprint &lt;command&gt; [...]
Common commands: list, create, show, diff, apply, destroy, git
<strong>$</strong></pre>

!SLIDE bullets

# Workflow

## Blueprints

* `blueprint create foo`
* Reverse-engineers the server.

!SLIDE bullets small

<pre><strong>$</strong> sudo blueprint create foo
# [blueprint] searching for APT packages to exclude
# [blueprint] caching excluded APT packages
# [blueprint] searching for Yum packages to exclude
# [blueprint] parsing ~/.blueprintignore
# [blueprint] searching for Yum packages
# [blueprint] searching for APT packages
# [blueprint] searching for Python packages
# [blueprint] searching for PEAR/PECL packages
# [blueprint] searching for Ruby gems
# [blueprint] searching for configuration files
# [blueprint] searching for software built from source
# [blueprint] searching for service dependencies
<strong>$</strong></pre>

!SLIDE bullets

# Workflow

## Packages

* APT, Yum, RubyGems, PyPI, PEAR, NPM.
* Ignores the &#8220;base system&#8221; by default.

!SLIDE bullets

# Workflow

## Files

* Configuration files from `/etc`.
* Ignores unchanged files from packages.

!SLIDE bullets

# Workflow

## Source installs

* `./configure && make && make install`
* Junk drawer.

!SLIDE bullets

# Workflow

## Services

* System V init and Upstart.  Runit soon.
* When should each service be restarted?

!SLIDE bullets

# Workflow

## Inspecting blueprints

* `blueprint show foo | less`
* Show the JSON representation.

!SLIDE bullets smaller

<pre>{
 "files": {
  "/etc/apt/sources.list.d/devstructure.list": {
   "content": "deb http://packages.devstructure.com natty main\n",
   "encoding": "plain",
   "group": "root",
   "mode": "100644",
   "owner": "root"
  }
 },
 "packages": {
  "apt": {
   "openssh-server": ["1:5.8p1-1ubuntu3"],
  }
 },
 "services": {
  "sysvinit": {
   "ssh": {
    "packages": {
     "apt": ["openssh-server"]
    }
   }
  }
 }
}</pre>

!SLIDE bullets

# Workflow

## Blueprint diet plan

* `~/.blueprintignore`
* Inspired by `gitignore`(5).

!SLIDE bullets

<pre><strong>$</strong> cat ~/.blueprintignore
/etc/apt
!/etc/apt/sources.list.d
!/etc/apt/trusted.gpg.d
*.pyc
/etc/hosts
/etc/nginx/sites-enabled/default
/etc/sudoers

:package:apt/build-essential
!:package:apt/build-essential
:package:apt/libopts25
:package:apt/libtspi1
<strong>$</strong> sudo blueprint create foo
...
<strong>$</strong></pre>

!SLIDE bullets

# Workflow

## Push and pull

* `blueprint push foo`
* `blueprint pull foo`

!SLIDE bullets

# Workflow

## Push and pull

* Unguessable URLs.
* Stored in S3.

!SLIDE bullets

# Workflow

## Bootstrap new environments

* `blueprint apply foo`
* New guy&#8217;s devbox?  No problem.

!SLIDE bullets

# Deployment

!SLIDE bullets

# Deployment

## Generate shell code

* `blueprint show -S foo`
* Run `foo.sh` without installing Blueprint.

!SLIDE bullets

# Deployment

## Puppet and Chef can play, too

* `blueprint show -P foo`
* `blueprint show -C foo`

!SLIDE bullets smaller

<pre class="sh_Puppet sh_sourceCode">class foo {
	Class['files'] -> Class['packages']
	class files {
		file {
			'/etc/apt/sources.list.d/devstructure.list':
				content => template('foo/etc/apt/sources.list.d/devstructure.list'),
				ensure  => file,
				group   => root,
				mode    => 0644,
				owner   => root;
		}
	}
	include files
	class packages {
		class apt {
			package {
				'openssh-server':
					ensure => '1:5.8p1-1ubuntu3';
			}
		}
		include apt
	}
	include packages
	class services {
		class sysvinit {
			service { 'ssh':
				enable    => true,
				ensure    => running,
				subscribe => Package['openssh-server'],
			}
		}
		include sysvinit
	}
	include services
}</pre>

!SLIDE bullets smaller

<pre class="sh_Ruby sh_sourceCode">cookbook_file('/etc/apt/sources.list.d/devstructure.list') do
  backup false
  group 'root'
  mode '0644'
  owner 'root'
  source 'etc/apt/sources.list.d/devstructure.list'
end
package('openssh-server') { version '1:5.8p1-1ubuntu3' }
service('ssh') do
  action [:enable, :start]
  subscribes :restart, resources('package[openssh-server]')
end</pre>

!SLIDE bullets

# Deployment

## Remotely generated shell code

* Fetch without installing Blueprint.
* Plays nicely with `cron`(8).

!SLIDE bullets smaller

<pre>0 * * * * curl https://devstructure.com/<em>secret</em>/foo/foo.sh | sh</pre>

!SLIDE bullets

# Deployment

## Bootstrap on EC2

* `user-data.sh`
* [`python-cloudformation`](https://github.com/devstructure/python-cloudformation)

!SLIDE bullets small

<pre class="sh_Sh sh_sourceCode">#!/bin/sh

set -e

TMPDIR="$(mktemp -d)"
cd "$TMPDIR"
trap "rm -rf \"$TMPDIR\"" EXIT

wget "http://devstructure.com/<em>secret</em>/foo/foo.sh"

sh "$(ls)"</pre>

!SLIDE bullets

# Future work

* Modularity and inheritance.
* Service supervisors: Runit, Monit, `systemd`.

!SLIDE bullets

# Thank you

* [rcrowley.org/talks/sf-python-2011-09-14/](http://rcrowley.org/talks/sf-python-2011-09-14/)
* <richard@devstructure.com> or [@rcrowley](https://twitter.com/rcrowley)
