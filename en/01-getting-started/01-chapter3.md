# Technical overview

## Architecture

Rundeck is an application hosted on a server you designate an administrative control point.

![Architecture diagram](http://rundeck.org/docs/figures/fig0001.png)

### Node roles
Two node roles in a rundeck environment:

*   Server Node: The node where Rundeck runs
     -  Deployment: Server installation
     -  Java: It is a servlets based webapp so requires java runtime
     -  Network access: SSH
     -  Relational database
     -  File system
     -  External providers
*   Remote Nodes: The nodes where commands are executed
     -  Required: Allow SSH access from server node
     -  Optional: HTTP access to server node

### User access

*   User interfaces
     - Graphical HTML
     - Shell tool for command line access
 *  Login and access control
     - Authentication
     - Authorization

### Installation methods

* RPM
* Launcher

A more extensive description up can be found in the user manual's 
[architecture section](http://rundeck.org/docs/RunDeck-Guide.html#rundeck-architecture).

## Concepts

Rundeck uses the following terminology

* Node: a host Resource
    - tags: a label describing a class, role, group, etc
    - Resource: a generic data type of which Node is an example
* Command: a system command or script 
* Job: a sequence of Command steps
     - options: user specified parameters to a Job
* Dispatch: an execution of a Command  on a Node
* History: previous dispatches
* Filter: a user specified query that can be named
    - node filter
    - job filter
    - history filter
* Project: a collection of Nodes and Jobs
    - Has its own resource model provider
