# Getting Started with Rundeck

## What you will learn 

This training program describes the overall Rundeck concept. 

*   Learn To
      - Understand the core concepts that drive Rundeck
      - Use the GUI and command line interfaces
      - Encapsulate your procedures so they can be executed via Rundeck
      - Introduce you to command sequencing using Jobs
*   Audience
      - System Administrators
      - Build Engineers
      - System Engineers
*   Prerequisites
      - Shell scripting knowledge
      - Linux host environment
      
# What is RunDeck?

## What problem does RunDeck solve?

Have you ever been in this situation? 

*  You have a procedure you need to run on a bunch of nodes
*  Need to find the current {spreadsheet,wiki,email,whiteboard,etc} describing nodes and their roles
*  Choose some command that will execute the necessary command(s) across the nodes
*  Execute the command everywhere it needs to run, maybe in parallel with or without ignoring errors
*  Collect the results and find nodes where it failed
*  Report the success or failure of the job run to my {boss,customer,team mate}

![figure](haha)

Rundeck helps solve this problem by providing:

* Access and use information about your nodes
* A command dispatching feature that will manage remote execution
* Store history about command executions

## What do you do with RunDeck ?

1.   Ad hoc command execution

      RunDeck dispatches commands to your hosts transparently and securely 
       by collecting node data into an easy to navigate tool.  

2.   Job workflow execution
       
      As commands become entrenched, RunDeck provides a management 
       primitives to govern those commands for use with NOC, or any group 
       where you want to provide a self service interface.

3.   Integrate your existing tools and procedures
      
      
