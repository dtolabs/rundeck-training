# Lab: Installation

## Install the software

1.   Get the distribution

    - Go to http://rundeck.org/downloads.html
    - For RPM:  Install the repo package for a yum install
    - For launcher: Download rundeck-laucher-1.1.0.jar to a directory (e.g, $HOME/rundeck)

1.    Run the installer

    - For [RPM](http://rundeck.org/docs/RunDeck-Guide.html#installing-with-rpm): ``yum install rundeck``
    - For [launcher](http://rundeck.org/docs/RunDeck-Guide.html#installing-with-launcher): ``java -jar rundeck-launcher-1.1.0.jar``

1.    Edit [configuration files](http://rundeck.org/docs/RunDeck-Guide.html#configuration-files) 
    - Located under the ``etc/`` directory
         - RPM: ``/etc/rundeck``
         - Launcher: ``$RDECK_BASE/etc``
    - ``framework.properties``: Where you can change the SSH key path
    - ``rundeck-config.properties``: Where you can change the listen port
    - ``realm.properties``: Flat file is default [logins](http://rundeck.org/docs/RunDeck-Guide.html#logins) but LDAP is optional 

1.    Configuring [SSH](http://rundeck.org/docs/RunDeck-Guide.html#ssh)

    - [Key setup](http://rundeck.org/docs/RunDeck-Guide.html#ssh-key-generation): Specifies the identity access remote hosts
        - ``framework.ssh.keypath``: Specifies the server identity file
    - [Remote machine setup](http://rundeck.org/docs/RunDeck-Guide.html#configuring-remote-machine-for-ssh): Add the server's identity to the authorized_key

## Project setup

[Tutorial: Create a project](RunDeck-Training.html##create-a-project)

* A  Project provides a space to manage related management activities.
* You can create a project with the GUI or command line.

For more see [RunDeck guide: Project setup](http://rundeck.org/docs/RunDeck-Guide.html#project-setup)

## Resource model

     "A resource model is a representation of your hosts"

* A specification of host metadata
* Each host where you want to execute a command is defined as a node
* Can be defined via XML or YAML. RunDeck will load the correct parser depending on file name extension

[Tutorial: Add remote hosts to the resource model](RunDeck-Tutorial.html#add-remote-hosts-to-the-resource-model)
