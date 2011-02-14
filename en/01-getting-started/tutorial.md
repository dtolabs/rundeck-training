# Introductory Tutorial

After reading this tutorial, you should have the general
pattern to using RunDeck.
You'll begin with the installation of the RunDeck server and
an SSH configuration that allows command execution
on the remote hosts. You will create a project where you will manage your
administration of the remote hosts. 

Each step of automation is specified in a new or existing scripts. 
Automation steps are combined into a workflow. Any script 
can be executed across the network via the
command dispatcher via the GUI or CLI. Dispatching flags control how
the commands are executed across the network.

## Download and Install

* Download the desired package type (e.g., Launcher jar or RPM)
* Install the RunDeck package on the chosen host
* Login to Rundeck. The default account is "admin" with password "admin".

## Create a project

A RunDeck Project provides a space to manage related automation
activities. The ``rd-project`` shell tool can be used to create the
workspace for each project from command line. There is also a GUI
for the same step which is used here.

On the RunDeck server, select the "Create new project"choice from the
project menu. Enter the project name at the prompt

![Create project prompt](http://rundeck.org/docs/figures/fig0203-a.png)

This will create the Project in the server, and registering the server
host as a Node in the "examples" project's Resource Model.

![Run page after new project](http://rundeck.org/docs/figures/fig0204.png)

The node "strongbad" is shown in the screenshot but it might be
"localhost" depending on your installation method and host configuration.

### Node resources

During project setup, a  bit of metadata about the RunDeck server host in the
form of a Node resource in the project's resource model is created. The resource
model is stored in a file and accessed during command execution.

Node resource data can be maintained via an external resource
provider, with an XML or YAML definition.

#### Add remote hosts to the resource model

With the RunDeck server Node configured and defined, register
remote nodes to the project.

For the sake of this document, imagine two remote hosts:
centos and ubuntu. 

Project resource data is stored under ``RDECK_BASE`. 
RPM installs store the data under ``/var/rundeck/projects`` while Launcher
installs will be set during installation and should be saved as an environment variable.

Add the following content to the RDECK_BASE/projects/examples/etc/resources.xml:

    <node name="ubuntu" type="Node" description="Ubuntu server node" 
       tags="simple" hostname="172.16.61.170"  username="demo" />
    <node name="centos" type="Node" description="Centos server node" 
       tags="simple" hostname="172.16.61.171"  username="alexh" />

Node resources have standard property called "tags".
A tag is a text label that you give to the Node, perhaps
denoting a classification, a role the node plays in the environment,
or group membership. 

The definitions for the remote nodes declare "simple" as the value for tags.

#### Setup SSH configuration

The Rundeck server configuration file, 
[framework.properties](http://rundeck.org/docs/RunDeck-Guide.html#framework.properties)
contains the identity used to connect to the remote hosts. This public key
must be added to the desired user's authorized_keys file on the remote hosts.

1.   Check the value of the ``framework.ssh.keypath`` setting. This file
      will be used as the identity when connecting to remote hosts:

          $ awk '/framework.ssh.keypath/ {print $3}' framework.properties
          /Users/alexh/.ssh/id_dsa

2.   Add the public key to the ``authorized_keys`` file on the remote hosts.

3.   Test a connection using this identity:

          ssh -i $(awk '/framework.ssh.keypath/ {print $3}' framework.properties) demo@ubuntu pwd

The command should execute successfully.

## Ad hoc command execution

An *ad hoc command* is a shell script or system executable that you
might run at an interactive terminal. Ad hoc commands can be executed
via the [dispatch](http://rundeck.org/docs/dispatch.html)
shell command or the "Run" page command prompt.

Here the  Unix ``uptime`` command is run to print system status:

![Command prompt](figures/fig0002.png)

Output from the command is displayed below the command prompt.

![Command output](figures/fig0003.png)

The server node is selected by default.  To execute the command on
other hosts, modify the Node filter.

### Node filtering

The command dispatcher supports a number of flags that can control where
commands are executed across nodes. The ``dispatch`` shell tool
and the RunDeck GUI support specifying these filtering options.

#### Include/exclude filters

Include and exclude flags filter the set of nodes the command will
execute. 

To execute a command across all Nodes specify wildcard pattern for 
the *name* property ``.*``:

![Node wildcard filter](figures/fig0004.png)

After pressing "Filter" you'll see the matches.

![Node listing](figures/fig0005.png)

Change the filter to include nodes with *tag* "simple".

![Node tag filter](figures/fig0006.png)

Here's an example that runs the ``whoami`` unix command on all hosts that tagged "simple":

![Whoami command](figures/fig0007.png)

Press the "Annotated" button to organize the output by node:

![Annotated view](figures/fig0008.png)

## Saved Jobs

Ad hoc commands occasionally become routine procedures. Rundeck 
provides a method to declare and save routine procedures as Jobs. 
Jobs can also be defined in groups to help organize procedures. 

### Info job

Create a Job that calls some commands that prints some system info.

Begin by pressing the "Save as Job..." 
button from the last ad hoc command run to reveal the new job form. 

![Save as Job...](figures/fig0009.png)

Select "Yes" in the "Save this job?" prompt. 

![Info job form](figures/fig0010.png)

For "Job Name", enter "info" and for the "Group", enter
"adm". Providing a description will be come helpful to other
users to understand the intent and purpose for the Job.

Review the form for "Dispatch to Nodes". It will already be
filled out from the  ad hoc command execution. 

![Info job nodes](figures/fig0011.png)

You'll notice in the Workflow section of the form there is a single step containing the
whoami command.

![Workflow definition](figures/fig0015.png)

Pressing the "Add a step" link opens up a dialog to create
additional command steps.

Once saved to the Workflow, press the "Create" button.

Once the the job is created, the browser will be directed to the
Jobs page. The folder structure reflecting the group naming will show
one Job. 

![The adm jobs](figures/fig0012.png)

Press the green arrow icon to run the Job.

![Info Run Job Now](figures/fig0013.png)

After the command executed, it is added to the history view.

![Info history](figures/fig0014.png)

You can see the job output by clicking the "View Output >>" link.

### Deployment job example

This next example shows how to use Rundeck to construct a multi-step
workflow that manages deployment for a simple webapp called "simple"
along with its servlet container, apache-tomcat.

![Deployment job overview](figures/fig0016.png)

The workflow will be organized into four steps:

1.   Shutdown: Stop tomcat container
      -  Check if tomcat is running and if so, run the tomcat ``shutdown.sh`` script
2.   Installation: Install the software artifacts
      -  Download and extract the Tomcat Zip and Simple WAR files
3.   Configuration: Configure the tomcat instance
      -  Customize the tomcat ``server.xml`` to run on different ports. 
         Change the file permissions for the tomcat stop/start scripts to be executable
4.   Startup: Start tomcat
      -  Run the tomcat ``startup.sh`` script. Verify it is running by checking the listening port.

Each of these steps will be defined in a separate job, with a fifth job managing the cycle.

To make this deployment example more realistic, the Zip and War files will be retrieved
from a web repository. This web repository plays the role of artificat distribution point.

#### Deployment job prerequisites

Before we can begin a few prerequisites must be met.

* Obtain example's artifacts: Download the apache-tomcat.zip and simple.war files
   -  Download [apache-tomcat](http://tomcat.apache.org/download-55.cgi)
   -  Download [simple.war](https://github.com/downloads/dtolabs/rundeck-training/simple-1.0.0.war)
* Copy artifacts to web repository: This is the web server where the artifacts are retrieved by the remote hosts
   - Create a directory called "simpleapp" inside the web docroot to store the example's artifacts. Substitute "$DOCROOT" for your own.
   - A simple README.txt file is added to test repository access
       
          mkdir $DOCROOT/simpleapp
          cp apache-tomcat*.zip simple*.war $DOCROOT/simpleapp
          echo "Simple app artifacts" > $DOCROOT/simpleapp/README.txt
	
   - Ensure the artifacts are accessible using the ``curl`` command to retrieve the README.txt file:
       
        curl http://strongbad/simpleapp/README.txt

Be sure the file and directory permissions are correct by making the files accessible 
to the webserver process.
	
#### The deployment scripts

Each of the four steps is implemented with a separate script.

To make the scripts flexible across environements they are parameterized
through command line options. Jobs can pass parameters to scripts using options.

These scripts approximate what you can plug in to manage your own processes.

The first script checks if the tomcat server is listening and if so
calls the ``$CATALINA_HOME/bin/shutdown.sh`` script to stop tomcat.
It expects a single argument that specifies the tomcat installation directory.

Script listing: "stop"

    #!/bin/bash
    # File: stop.sh <catalina-home>
    die() { echo "ERROR: $@" ; exit 1 ; }
    [ $# = 1 ] || { die "Usage: stop.sh <catalina-home>" ; }     
    export CATALINA_HOME=$1
    export PORT=28080     
    [ ! -d "$1" ] && {  echo "CATALINA_HOME not found: $1" ; }     
    export CATALINA_BASE=$CATALINA_HOME;
    netstat -an|grep ${PORT}|grep -q LISTEN && {
        echo "Tomcat listening (port=$PORT). Running stop script ..."
        $CATALINA_HOME/bin/shutdown.sh || die "Script failed: stop.sh"
        exit $?
    } 
    echo "Tomcat stopped"
    exit 0

The start script is a basic wrapper around tomcat's ``$CATALINA_HOME/bin/startup.sh``
script.

Script listing: "start"

    #!/bin/bash
    # File: start.sh
    USAGE="$0 <catalinahome>"
    die() { echo "ERROR: $@" ; exit 1 ; }
    [ $# = 1 ] || { die "Usage: start.sh <catalinahome>" ; }
    export CATALINA_HOME=$1
    [ -d "$CATALINA_HOME" ] || { die "CATALINA_HOME not found: $CATALINA_HOME" ; }     
    export CATALINA_BASE=$CATALINA_HOME;
    echo "Invoking $CATALINA_HOME/bin/startup.sh"
    $CATALINA_HOME/bin/startup.sh;
    exit $?

The install script does the work of downloading the archives from the specified URLs
and extracts them to the speciifed catalinahome directory:

Script listing: "install"

    #!/bin/bash
    # File: install.sh 
    USAGE="$0 <catalinahome> <tomcat-zip-url> <simple-war-url>"     
    die() { echo "ERROR: $@" ; exit 1 ; }
    [ $# = 3 ] || { die "$USAGE" ; }     
    CATALINA_HOME=$1; 
    TOMCAT_PKG_URL=$2;
    SIMPLE_PKG_URL=$3;
     
    catalina_parentdir=`dirname $CATALINA_HOME`  ;# parent dir
    mkdir -p $catalina_parentdir || {die "failed creating dir: ${catalina_parentdir}"}
    tomcat_filename=${TOMCAT_PKG_URL##*/} ;# parse file names
    tomcat_base=${tomcat_filename%.zip}   ;#
    simple_filename=${SIMPLE_PKG_URL##*/} ;#
    simple_base=${simple_filename%-*}     ;#
     
    [ -d ${CATALINA_HOME} ] && {
        rm -r ${CATALINA_HOME} || die "failed cleaning existing tomcat deployment"
    }
     
    fetch() {
        srcurl=$1; dstpath=$2; overwrite=$3
        curl -s -S  ${srcurl} -o ${dstpath} || die "Download failed: ${srcurl}"
        echo "Downloaded ${srcurl}" 
    } 
     
    # retrieve the ZIP
    fetch  ${TOMCAT_PKG_URL}  $catalina_parentdir/${tomcat_filename}
     
    # extract the ZIP
    unzip -q $catalina_parentdir/${tomcat_filename} -d $catalina_parentdir || {
      die "Extract failed: $catalina_parentdir/${tomcat_filename}"
    }
     
    # move the extracted dir to the catalina_home basename
    mv $catalina_parentdir/${tomcat_base} ${CATALINA_HOME} || die "failed to create ${CATALINA_HOME}"
    echo "Extracted ${tomcat_filename} to $CATALINA_HOME"
     
    # retrieve the WAR
    fetch ${SIMPLE_PKG_URL} $catalina_parentdir/${simple_filename}
     
    # extract the WAR
    mkdir -p ${CATALINA_HOME}/webapps/${simple_base} || {
        die "failed making subdirectory directory: ${simple_base}"
    }
    unzip -q $catalina_parentdir/${simple_filename} -d ${CATALINA_HOME}/webapps/${simple_base} || {
        die "Extract failed: $catalina_parentdir/$simple_filename"
    }
    echo "install done. extracted ${simple_filename} to ${CATALINA_HOME}/webapps/${simple_base}"     

The configure script re-writes the ``server.xml`` file to make tomcat
run on a different set of ports.

Script listing: "configure"

    #!/bin/sh
    # File: configure.sh
    USAGE="$0 configure.sh <catalina-home>"
    die() { echo "ERROR: $@" ; exit 1 ; }
    [ $# = 1 ] || { die "$USAGE" ; }     
    CATALINA_HOME=$1     
    [ -d "$CATALINA_HOME" ] || { die "CATALINA_HOME not found: $1" ; }     
    echo "Chmod'ing files in $CATALINA_HOME/bin/*.sh"
    chmod +x $CATALINA_HOME/bin/*.sh || die "chmod step failed"     
    echo "Customizing $CATALINA_HOME/conf/server.xml ..."
    server_xml_tmp=/tmp/server.xml.$$ ;# define a temporary file     
    sed -e 's/8080/28080/g' -e 's/8005/28005/g' \
        -e 's/8009/28009/g' $CATALINA_HOME/conf/server.xml > ${server_xml_tmp}
    mv ${server_xml_tmp} $CATALINA_HOME/conf/server.xml || {
        die "Text replace failed: $CATALINA_HOME/conf/server.xml"
    }     
    echo "$CATALINA_HOME/conf/server.xml customized"


### Job options

Each of the deployment scripts takes one or more command line options. 
Values for these options can be specified when the job runs.

Here is a snippet of XML that defines three job options: 
catalinahome, simple, tomcat:

     <options> 
        <option name="catalinahome" enforcedvalues="true" required="true" 
           values="/var/rundeck/simpleapp/apache-tomcat" 
           description="the tomcat install directory"/>  
        <option name="simple" enforcedvalues="true" required="true" 
           valuesUrl="http://localhost/simpleapp/simple.json" description="the simple war"/>  
        <option name="tomcat" enforcedvalues="true" required="true" 
           valuesUrl="http://localhost/simpleapp/tomcat.json" 
           description="tomcat zip"/> 
      </options> 

The "catalinahome" option has a default value, ``/var/rundeck/simpleapp``.

Default choices can be specified as a comma separated list but
can also be requested from an external source using the ``valuesUrl``
attribute.

It's often advantageous to define options in an external source since
this information often varies independently from the job definitions.

We'll store the URLs to the tomcat Zip and simple War archives stored
in the webserver using JSON files.
Whenever the Job is run, this JSON data will be used to populate a menu
for the option.

The JSON format is simple:

     {key:value, key:value, ...}

For this example, we'll make the artifact URL the value and a short mnemonic for the key.

      
File listing: tomcat.json

     {"tomcat-5-1.31":"http://strongbad/simpleapp/apache-tomcat-5.1.31.zip"}
   
File listing: simple.json

     {"simple-1.0.0":"http://strongbad/simpleapp/simple-1.0.0.war"}
     
Copy the content for simple.json and tomcat.json to to their own
files and move the files to $DOCROOT/simpleapp.
We'll use the URLs to the JSON files in the ``valuesUrl`` option attributes
when the Jobs are defined.

### The deployment jobs

While each job can be defined graphically in Rundeck as we did for the "info" job, 
each job can be defined succinctly with in an XML document. This 
document contains a set of tags corresponding to the choices seen in
the Rundeck graphical job editor.

XML definitions for each of the jobs is included below. Copy the content for
each definition to a file on the Rundeck server host.

The first job definition describes the job named "Deploy". This job
does nothing except call other jobs using the ``jobref`` tag.
Inside the ``jobref`` element, an``arg`` 
element is used to pass Deploy job option choices to the subordinate jobs. 

Here's the basic usage of the  jobref and arg tags:

      <jobref name="<NAME>" group="<GROUP>"> 
          <arg line="-param1 paramValue"/> 
      </jobref> 
	    

File listing: Deploy.xml

    <joblist> 
      <job> 
        <name>Deploy</name>  
        <description>deploy the simple app</description>  
        <loglevel>INFO</loglevel>  
        <group>simpleapp</group>  
        <context> 
          <project>examples</project>  
          <options> 
            <option name="catalinahome" enforcedvalues="true" required="true" 
     	      values="/var/rundeck/simpleapp/apache-tomcat" description="the tomcat install directory"/>  
            <option name="simple" enforcedvalues="true" required="true" 
     	      valuesUrl="http://localhost/simpleapp/simple.json" description="the simple war"/>  
            <option name="tomcat" enforcedvalues="true" required="true" 
     	      valuesUrl="http://localhost/simpleapp/tomcat.json" description="tomcat zip"/> 
          </options> 
        </context>  
        <sequence threadcount="1" keepgoing="false" strategy="node-first"> 
          <command> 
            <jobref name="stop" group="simpleapp"> 
              <arg line="-catalinahome ${option.catalinahome}"/> 
            </jobref> 
          </command>  
          <command> 
            <jobref name="install" group="simpleapp"> 
              <arg line="-catalinahome ${option.catalinahome} -simple ${option.simple} -tomcat ${option.tomcat}"/> 
            </jobref> 
          </command>  
          <command> 
            <jobref name="configure" group="simpleapp"> 
              <arg line="-catalinahome ${option.catalinahome}"/> 
            </jobref> 
          </command>  
          <command> 
            <jobref name="start" group="simpleapp"> 
              <arg line="-catalinahome ${option.catalinahome}"/> 
            </jobref> 
          </command> 
        </sequence>  
        <dispatch> 
          <threadcount>1</threadcount>  
          <keepgoing>false</keepgoing> 
        </dispatch> 
      </job>  
    </joblist> 
    
Jobs that do not specify node filters default to the Rundeck
host. That is how we want Deploy to run since it does nothing
but call the other "worker" jobs.
    
Now, on to the "worker" jobs that perform each of the actual steps.    

The basic declaration for a script step uses the following tag structure:

    <command>
       <script>
          ... your script content here ...
       </script>
       <scriptargs> you script args here </scriptargs>
    </command>

Each step is defined using a ``<command>`` tag. Script content
is embedded inside the ``<script>`` tag and arguments are passed
using the ``<scriptargs>`` tag. CDATA sections can be used to
preserve formatting and white space.

    
The "stop" job embeds the "stop" script described earlier.
It also defines an option named "catalinahome".  
The option.catalinahome
value is passed to the script inside the ``<scriptargs>`` XML tag.

File listing: stop.xml

    <joblist>  
      <job> 
        <name>stop</name>  
        <description>stop the simple app</description>  
        <loglevel>INFO</loglevel>  
        <group>simpleapp</group>  
        <context> 
          <project>examples</project>  
          <options> 
            <option name="catalinahome" enforcedvalues="false" required="true" 
           values="/var/rundeck/simpleapp/apache-tomcat" 
     	   description="the tomcat install directory"/> 
          </options> 
        </context>  
        <sequence threadcount="1" keepgoing="false" strategy="node-first"> 
          <command> 
            <script><![CDATA[#!/bin/bash
    # File: stop.sh <catalina-home>
    die() { echo "ERROR: $@" ; exit 1 ; }
    [ $# = 1 ] || { die "Usage: stop.sh <catalina-home>" ; }
     
    export CATALINA_HOME=$1
    export PORT=28080
     
    [ ! -d "$1" ] && {  echo "CATALINA_HOME not found: $1" ; }
     
    export CATALINA_BASE=$CATALINA_HOME;
    netstat -an|grep ${PORT}|grep -q LISTEN && {
        echo "Tomcat listening (port=$PORT). Running stop script ..."
        $CATALINA_HOME/bin/shutdown.sh || die "Script failed: stop.sh"
        exit $?
    } 
    echo "Tomcat stopped"
    exit 0
    ]]></script>  
            <scriptargs>${option.catalinahome}</scriptargs> 
          </command> 
        </sequence>  
        <nodefilters excludeprecedence="true"> 
          <include> 
            <tags>simpleapp</tags> 
          </include> 
        </nodefilters>  
        <dispatch> 
          <threadcount>1</threadcount>  
          <keepgoing>false</keepgoing> 
        </dispatch> 
      </job> 
    </joblist>
    
The "stop" job along with the remaining jobs execute on nodes tagged "simple".

The following XML elements are used to define that node filter:

        <nodefilters excludeprecedence="true"> 
          <include> 
            <tags>simpleapp</tags> 
          </include> 
        </nodefilters>  

The "install" job embeds the ``install.sh`` script content described earlier.
It also defines all three options: catalinahome, simple, tomcat.

File listing: install.xml
    
    <joblist> 
      <job> 
        <name>install</name>  
        <description>installs the simple app server and war</description>  
        <loglevel>INFO</loglevel>  
        <group>simpleapp</group>  
        <context> 
          <project>examples</project>  
          <options> 
            <option name="catalinahome" enforcedvalues="false" required="true" 
     	     values="/var/rundeck/simpleapp/apache-tomcat" 
     	     description="the tomcat install directory"/>  
            <option name="simple" enforcedvalues="false" required="true" 
     	     valuesUrl="file:/var/rundeck/simpleapp/simple.json" 
     	     description="the simple war url"/>  
            <option name="tomcat" enforcedvalues="false" required="true" 
     	      valuesUrl="file:/var/rundeck/simpleapp/tomcat.json" 
     	      description="tomcat zip url"/> 
          </options> 
        </context>  
        <sequence threadcount="1" keepgoing="false" strategy="node-first"> 
          <command> 
            <script><![CDATA[#!/bin/bash
    # File: install.sh 
    USAGE="$0 <catalinahome> <tomcat-pkg-url> <simple-pkg-url>"
     
    die() { echo "ERROR: $@" ; exit 1 ; }
    [ $# = 3 ] || { die "$USAGE" ; }
     
    CATALINA_HOME=$1; 
    TOMCAT_PKG_URL=$2;
    SIMPLE_PKG_URL=$3;
     
    catalina_parentdir=`dirname $CATALINA_HOME`  ;# parent dir
    mkdir -p $catalina_parentdir || {die "failed creating dir: ${catalina_parentdir}"}
    tomcat_filename=${TOMCAT_PKG_URL##*/} ;# parse file names
    tomcat_base=${tomcat_filename%.zip}   ;#
    simple_filename=${SIMPLE_PKG_URL##*/} ;#
    simple_base=${simple_filename%-*}     ;#
     
    [ -d ${CATALINA_HOME} ] && {
        rm -r ${CATALINA_HOME} || die "failed cleaning existing tomcat deployment"
    }
     
    fetch() {
        srcurl=$1; dstpath=$2; overwrite=$3
        curl -s -S  ${srcurl} -o ${dstpath} || die "Download failed: ${srcurl}"
        echo "Downloaded ${srcurl}" 
    } 
     
    # retrieve the ZIP
    fetch  ${TOMCAT_PKG_URL}  $catalina_parentdir/${tomcat_filename}
     
    # extract the ZIP
    unzip -q $catalina_parentdir/${tomcat_filename} -d $catalina_parentdir || {
      die "Extract failed: $catalina_parentdir/${tomcat_filename}"
    }
     
    # move the extracted dir to the catalina_home basename
    mv $catalina_parentdir/${tomcat_base} ${CATALINA_HOME} || die "failed to create ${CATALINA_HOME}"
    echo "Extracted ${tomcat_filename} to $CATALINA_HOME"
     
    # retrieve the WAR
    fetch ${SIMPLE_PKG_URL} $catalina_parentdir/${simple_filename}
     
    # extract the WAR
    mkdir -p ${CATALINA_HOME}/webapps/${simple_base} || {
        die "failed making subdirectory directory: ${simple_base}"
    }
    unzip -q $catalina_parentdir/${simple_filename} -d ${CATALINA_HOME}/webapps/${simple_base} || {
        die "Extract failed: $catalina_parentdir/$simple_filename"
    }
    echo "extracted ${simple_filename} to ${CATALINA_HOME}/webapps/${simple_base}"
     
    echo "install done"
    ]]></script>  
            <scriptargs>${option.catalinahome} ${option.tomcat} ${option.simple}</scriptargs> 
          </command> 
        </sequence>  
        <nodefilters excludeprecedence="true"> 
          <include> 
            <tags>simpleapp</tags> 
          </include> 
        </nodefilters>  
        <dispatch> 
          <threadcount>1</threadcount>  
          <keepgoing>false</keepgoing> 
        </dispatch> 
      </job>  
    </joblist>   

The "configure" job embeds the ``configure.sh`` script content described earlier.
It also defines an option named "catalinahome".
    
File listing: configure.xml
     
    <joblist> 
      <job> 
        <name>configure</name>  
        <description>configure the simple app</description>  
        <loglevel>INFO</loglevel>  
        <group>simpleapp</group>  
        <context> 
          <project>examples</project>  
          <options> 
            <option name="catalinahome" enforcedvalues="false" required="true" 
     	      description="the tomcat install directory"/> 
          </options> 
        </context>  
        <sequence threadcount="1" keepgoing="false" strategy="node-first"> 
          <command> 
            <script><![CDATA[#!/bin/sh
    # File: configure.sh
    USAGE="$0 configure.sh <catalina-home>"
    die() { echo "ERROR: $@" ; exit 1 ; }
    [ $# = 1 ] || { die "$USAGE" ; }
     
    CATALINA_HOME=$1
     
    [ -d "$CATALINA_HOME" ] || { die "CATALINA_HOME not found: $1" ; }
     
    echo "Chmod'ing files in $CATALINA_HOME/bin/*.sh"
    chmod +x $CATALINA_HOME/bin/*.sh || die "chmod step failed"
     
    echo "Customizing $CATALINA_HOME/conf/server.xml ..."
    server_xml_tmp=/tmp/server.xml.$$ ;# define a temporary file
     
    sed -e 's/8080/28080/g' -e 's/8005/28005/g' \
        -e 's/8009/28009/g' $CATALINA_HOME/conf/server.xml > ${server_xml_tmp}
    mv ${server_xml_tmp} $CATALINA_HOME/conf/server.xml || {
        die "Text replace failed: $CATALINA_HOME/conf/server.xml"
    }
     
    echo "$CATALINA_HOME/conf/server.xml customized"
    ]]></script>  
            <scriptargs>${option.catalinahome}</scriptargs> 
          </command> 
        </sequence>  
        <nodefilters excludeprecedence="true"> 
          <include> 
            <tags>simpleapp</tags> 
          </include> 
        </nodefilters>  
        <dispatch> 
          <threadcount>1</threadcount>  
          <keepgoing>false</keepgoing> 
        </dispatch> 
      </job>  
    </joblist>  

The "start" job embeds the ``start.sh`` script content described earlier.
It also defines an option named "catalinahome".

File listing: start.xml
     
    <joblist> 
      <job> 
        <name>start</name>  
        <description>start the simple app</description>  
        <loglevel>INFO</loglevel>  
        <group>simpleapp</group>  
        <context> 
          <project>examples</project>  
          <options> 
            <option name="catalinahome" enforcedvalues="false" required="true" 
     	   values="/var/rundeck/simpleapp/apache-tomcat" 
     	   description="the tomcat install directory"/> 
          </options> 
        </context>  
        <sequence threadcount="1" keepgoing="false" strategy="node-first"> 
          <command> 
            <script><![CDATA[#!/bin/bash
    # File: start.sh
    USAGE="$0 <catalinahome>"
    die() { echo "ERROR: $@" ; exit 1 ; }
    [ $# = 1 ] || { die "Usage: start.sh <catalinahome>" ; }
    export CATALINA_HOME=$1
    [ -d "$CATALINA_HOME" ] || { die "CATALINA_HOME not found: $CATALINA_HOME" ; }
     
    export CATALINA_BASE=$CATALINA_HOME;
    echo "Invoking $CATALINA_HOME/bin/startup.sh"
    $CATALINA_HOME/bin/startup.sh;
    exit $?]]></script>  
            <scriptargs>${option.catalinahome}</scriptargs> 
          </command> 
        </sequence>  
        <nodefilters excludeprecedence="true"> 
          <include> 
            <tags>simpleapp</tags> 
          </include> 
        </nodefilters>  
        <dispatch> 
          <threadcount>1</threadcount>  
          <keepgoing>false</keepgoing> 
        </dispatch> 
      </job>  
    </joblist>


After saving the XML documents to files located on the Rundeck server, 
you can load them using the ``rd-jobs`` command. 

Run the ``rd-jobs load -f <file>`` command for each job definition file:

    rd-jobs load -f stop.xml
    rd-jobs load -f install.xml    
    rd-jobs load -f configure.xml
    rd-jobs load -f start.xml    
    rd-jobs load -f Deploy.xml
    
Go to the Rundeck Jobs page to see the jobs listed.

![Simpleapp job list](figures/fig0017.png)

Moving the mouse over the Deploy job shows how the whole process hangs together.

![Deploy job](figures/fig0018.png)

### Run the deployment job

Press the "run" button to start the Deploy job.

![Run now](figures/fig0019.png)

You will be prompted for the options:

![Choose options](figures/fig0020.png)

Make the selections and then press "Run Job Now" to execute it.
The execution will become visible in the "Now Running" area.

![Now running Deploy](figures/fig0021.png)

### Command line Job control

Shell utilities exist to provide command line Job control.

The [run](http://rundeck.org/docs/run.html)
command is used to start the execution of a Job defined in Rundeck. 

Here's an example that runs the "info" job defined in the section above:

    $ run -j adm/info
    Job execution started:
    [51] info <http://strongbad:4440/execution/follow/51>

Jobs as well as commands executed with the -Q option can be listed
with the [rd-queue](http://rundeck.org/docs/rd-queue.html)
command. The ``rd-queue`` command lists jobs
currently running in Rundeck execution queue:

    $ rd-queue
    Queue: 1 items
    [5] workflow: Workflow:(threadcount:1){ [command( scriptfile: /Users/alexh/bin/checkagain.sh)] } <http://strongbad:4440/execution/follow/5>

Running jobs can also be killed via the ``rd-queue`` "kill"
command. The ``rd-queue`` command includes the execution ID for each
running job. Specify execution ID using the "-e" option:

    $ rd-queue kill -e 5
    rd-queue kill: success. [5] Job status: killed

