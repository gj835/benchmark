# TestOn Enviroment Setup

## Overview

**TestOn** is hte test frame work made by ON.Lab, designed for ONOS *BenchMark* and *User Case* testing. **TestOn** can use for test on **Cluster** or **Single Node** scenario. 

### Hardware Environment

#### Node Arrangement

                  	  -------------------------------
                     |           node 1:             |
                     |       TestOn FrameWork        |
                     |            Mininet            |
                     |            Java 8             |
                     |            Maven              |
                     |            Karaf              |
                     |                               |
	                  -------------------------------
                                    |
                                    | 
                                    |  Internal Subnet (192.168.204.X e.g)
                                    |
                                    |
        -------------------------------
       |         node 2 - 4:           |             1. Keyless ssh through node 1 to 4 (including themselvies)
       |      Java 8 Environment       |-----------        2. Keyless access sudo command
       |                               |           |
       |                               |           |-----------
        -------------------------------            |           |
                     |                             |           |
                      -----------------------------            |
                                 |      Up to 7 nodes(odd)     |
                                  -----------------------------


Node 1: run as main test machine and run Mininet.

Node 2 - 3: run as test machine, after setting up basic enviroment, 
Node 1 will inject ONOS code to these nodes and remotely send command to nodes to performance the test behaivor.

**By setting up cell file in node 1, the address setting will populate to test nodes**

#### Dependencies

##### For all machines

**sudoer** 

	# change sudoer file to allow the currnet use (sdn e.g) to use sudo command without password
	$ visudo
	# insert the following setting:
	# sdn     ALL=NOPASSWD: ALL

**keyless ssh access**
	
	# set up public key for remotely access all nodes without password
	$ ssh-keygen -t rsa
	$ ssh-copy-id 192.168.204.X  # for example, X indicate a certain number, 
	#including the node itself

**Java**

	# set up java enviromnet
	$ sudo apt-get install software-properties-common -y
	$ sudo add-apt-repository ppa:webupd8team/java -y
	$ sudo apt-get update
	$ sudo apt-get install oracle-java8-installer oracle-java8-set-default -y
	$ export JAVA_HOME=/usr/lib/jvm/java-8-oracle
	
#####Node 1 (onosbench machine)

**Essential packages**
	
	$ sudo apt-get install -y git
	$ sudo apt-get install -y build-essential python-dev
	$ sudo pip install configObj
	$ sudo pip install pexpect==3.2
	$ sudo pip install configObj
	$ sudo pip install numpy

**Mininet**

	# install Mininet
	$ git clone http://github.com/mininet/mininet
	$ mininet/util/install.sh -nvfw

**TestON packages**

	# download TestOn code
	$ git clone https://gerrit.onosproject.org/OnosSystemTest
	$ ln -s OnosSystemTest/TestON TestON
	$ export PYTHONPATH=/home/sdn/TestON/

**ONOS packages**

	# download ONOS code and set up environment
	$ git clone https://gerrit.onosproject.org/onos
	$ export ONOS_ROOT=/home/xxx/onos
	$ vi ~/.bashrc
	# insert the fllowing code to the end of bashrc file
	# . ~/onos/tools/dev/bash_profile
	$ . ~/.bashrc
	
	# change java's heap memory size
	$ vi ~/onos/tools/package/bin/onos-service
	# insert the following part
	# export JAVA_OPTS="${JAVA_OPTS:--Xms8G -Xmx8G}"
	
**Maven & Karaf**

	$ cd; mkdir Downloads Applications
	$ cd Downloads
	$ wget http://archive.apache.org/dist/karaf/3.0.5/apache-karaf-3.0.5.tar.gz
	$ wget http://archive.apache.org/dist/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz
	$ tar -zxvf apache-karaf-3.0.5.tar.gz -C ../Applications/
	$ tar -zxvf apache-maven-3.3.9-bin.tar.gz -C ../Applications/
	
	$ vi ~/Applications/apache-karaf-3.0.5/etc/org.apache.karaf.features.cfg
	# insert the fllowing code to the setting file
	# mvn:org.onosproject/onos-features/1.5.0-SNAPSHOT/xml/features

**Build onos package for testing**

	$ cd ~/onos
	$ mvn install -DskipTests -Dcheckstyle.skip
	$ op

**Set up cell environment variable**

	$ vi ~/onos/tools/test/cells/testCell
	# OC1 - OC3: corespinding to the test machine
	# OCN	   : address of Mininet (default to be localhost)
	# ONOS_NIC = 192.168.204.*
	$ cell testCell
	
**Cell file example**

	export ONOS_NIC=192.168.204.*
	export OC1="192.168.204.15"
	export OCI=$OC1
	export OCN="localhost"
	export ONOS_APPS=drivers
	export ONOS_GROUP=sdn
	export ONOS_USER=sdn
	
Add the following code to the end of ~/.bashrc (on node1), for removing previous test environment

	killTestONall(){
		sudo kill -9 `ps -ef | grep "./cli.py" | grep -v grep | grep -v vim | awk '{print $2}'`
		sudo kill -9 `ps -ef | grep "ssh -X" | grep -v grep | awk '{print $2}'`
		sudo pkill bgpd
		sudo pkill zebra
		sudo kill -9 `ps -ef | grep "bird" | grep -v grep | awk '{print $2}'`
		echo "uninstall onos on all nodes"
		NUM="$(cell | egrep OC[0-9] |wc -l)"
		#for i in {1..7}; do onos-uninstall 10.128.21.$i& done
		stc teardown
		echo "cleaning on all nodes"
		for i in $(seq 1 $NUM); do onos-ssh $OC$i "sudo pkill -9 java"; done
		ssh sdn@$OCN "sudo mn -c &"
	}
	
	
#### Starting Sample Test
	$ cd ~/TestON/bin
	$ ./cli.py run SAMPstartTemplate2_3node


All test case script is locate at ~/TestON/tests/. For example the above test case:
~/TestON/tests/SAMP/SAMPstartTemplate2_3node

You will see three files:

**SAMPstartTemplate2_3node.params**  
test case summarys

**SAMPstartTemplate2_3node.topo**    
nodes information

**SAMPstartTemplate2_3node.py**      
test case code


#### Start SCPFbatchFlowResp (OpenFlow Northbound Test)
	
###### Prepare for test parameters (exp. Switch number)
	$ cd /TestON/tests/MISC/SCPFbatchFlowResp
	$ vi SCPFbatchFlowResp.params
	# change <GLOBAL><numSw> to the switch number you need

##### Start test

	$ cd ~/TestON/bin
	$ ./cli.py run SCPFbatchFlowResp

<!--




An email <example@example.com> link.

Simple inline link <http://chenluois.com>, another inline link [Smaller](http://25.io/smaller/), one more inline link with title [Resize](http://resizesafari.com "a Safari extension").

A [reference style][id] link. Input id, then anywhere in the doc, define the link with corresponding id:

[id]: http://25.io/mou/ "Markdown editor on Mac OS X"

Titles ( or called tool tips ) in the links are optional.

#### Images

An inline image ![Smaller icon](http://25.io/smaller/favicon.ico "Title here"), title is optional.

A ![Resize icon][2] reference style image.

[2]: http://resizesafari.com/favicon.ico "Title"

#### Inline code and Block code

Inline code are surround by `backtick` key. To create a block code:

	Indent each line by at least 1 tab, or 4 spaces.
    var Mou = exactlyTheAppIwant; 
    asdf
    
####  Ordered Lists

Ordered lists are created using "1." + Space:

1. Ordered list item
2. Ordered list item
3. Ordered list item

#### Unordered Lists

Unordered list are created using "*" + Space:

* Unordered list item
* Unordered list item
* Unordered list item 

Or using "-" + Space:

- Unordered list item
- Unordered list item
- Unordered list item

#### Hard Linebreak

End a line with two or more spaces will create a hard linebreak, called `<br />` in HTML. ( Control + Return )  
Above line ended with 2 spaces.

#### Horizontal Rules

Three or more asterisks or dashes:

***

---

- - - -

#### Headers

Setext-style:

This is H1
==========

This is H2
----------

atx-style:

# This is H1
## This is H2
### This is H3
#### This is H4
##### This is H5
###### This is H6


### Extra Syntax

#### Footnotes

Footnotes work mostly like reference-style links. A footnote is made of two things: a marker in the text that will become a superscript number; a footnote definition that will be placed in a list of footnotes at the end of the document. A footnote looks like this:

That's some text with a footnote.[^1]

[^1]: And that's the footnote.


#### Strikethrough

Wrap with 2 tilde characters:

~~Strikethrough~~


#### Fenced Code Blocks

Start with a line containing 3 or more backticks, and ends with the first line with the same number of backticks:

```
Fenced code blocks are like Stardard Markdown’s regular code
blocks, except that they’re not indented and instead rely on
a start and end fence lines to delimit the code block.
```

#### Tables

A simple table looks like this:

First Header | Second Header | Third Header
------------ | ------------- | ------------
Content Cell | Content Cell  | Content Cell
Content Cell | Content Cell  | Content Cell

If you wish, you can add a leading and tailing pipe to each line of the table:

| First Header | Second Header | Third Header |
| ------------ | ------------- | ------------ |
| Content Cell | Content Cell  | Content Cell |
| Content Cell | Content Cell  | Content Cell |

Specify alignment for each column by adding colons to separator lines:

First Header | Second Header | Third Header
:----------- | :-----------: | -----------:
Left         | Center        | Right
Left         | Center        | Right


### Shortcuts

#### View

* Toggle live preview: Shift + Cmd + I
* Toggle Words Counter: Shift + Cmd + W
* Toggle Transparent: Shift + Cmd + T
* Toggle Floating: Shift + Cmd + F
* Left/Right = 1/1: Cmd + 0
* Left/Right = 3/1: Cmd + +
* Left/Right = 1/3: Cmd + -
* Toggle Writing orientation: Cmd + L
* Toggle fullscreen: Control + Cmd + F

#### Actions

* Copy HTML: Option + Cmd + C
* Strong: Select text, Cmd + B
* Emphasize: Select text, Cmd + I
* Inline Code: Select text, Cmd + K
* Strikethrough: Select text, Cmd + U
* Link: Select text, Control + Shift + L
* Image: Select text, Control + Shift + I
* Select Word: Control + Option + W
* Select Line: Shift + Cmd + L
* Select All: Cmd + A
* Deselect All: Cmd + D
* Convert to Uppercase: Select text, Control + U
* Convert to Lowercase: Select text, Control + Shift + U
* Convert to Titlecase: Select text, Control + Option + U
* Convert to List: Select lines, Control + L
* Convert to Blockquote: Select lines, Control + Q
* Convert to H1: Cmd + 1
* Convert to H2: Cmd + 2
* Convert to H3: Cmd + 3
* Convert to H4: Cmd + 4
* Convert to H5: Cmd + 5
* Convert to H6: Cmd + 6
* Convert Spaces to Tabs: Control + [
* Convert Tabs to Spaces: Control + ]
* Insert Current Date: Control + Shift + 1
* Insert Current Time: Control + Shift + 2
* Insert entity <: Control + Shift + ,
* Insert entity >: Control + Shift + .
* Insert entity &: Control + Shift + 7
* Insert entity Space: Control + Shift + Space
* Insert Scriptogr.am Header: Control + Shift + G
* Shift Line Left: Select lines, Cmd + [
* Shift Line Right: Select lines, Cmd + ]
* New Line: Cmd + Return
* Comment: Cmd + /
* Hard Linebreak: Control + Return

#### Edit

* Auto complete current word: Esc
* Find: Cmd + F
* Close find bar: Esc

#### Post

* Post on Scriptogr.am: Control + Shift + S
* Post on Tumblr: Control + Shift + T

#### Export

* Export HTML: Option + Cmd + E
* Export PDF:  Option + Cmd + P


### And more?

Don't forget to check Preferences, lots of useful options are there.

Follow [@Mou](https://twitter.com/mou) on Twitter for the latest news.

For feedback, use the menu `Help` - `Send Feedback`-->