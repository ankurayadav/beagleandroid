This contains steps for building [AOSP](https://source.android.com/) android kitkat port for beaglebone.
=================

Reference: [http://bbbandroid.sourceforge.net/build.html](http://bbbandroid.sourceforge.net/build.html)

##Step 1:
* First we have to [initialize a build environment](http://source.android.com/source/initializing.html)
	1. Install latest version of android sdk
		<pre><code>
			sudo add-apt-repository ppa:webupd8team/java
			sudo apt-get update
			sudo apt-get install oracle-java6-installer
		</pre></code>
	2. Install required packages for (Ubuntu 14.04)
		<pre><code>
			sudo apt-get install bison g++-multilib git gperf libxml2-utils
		</pre></code>
	3. Download Repo Tool
		<pre><code>
			mkdir ~/bin
			PATH=~/bin:$PATH
			curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
			chmod a+x ~/bin/repo
		</pre></code>

##Step 2
Fetch android source code
		<pre><code>
			cd bbbandroid
			repo init -u http://github.com/hendersa/bbbandroid-manifest
			repo sync -c 
		</pre></code>
This will take several hours depending upon internet speed.

***************pending*********************