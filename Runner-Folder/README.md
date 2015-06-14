Here you can learn how to create External Runners, also known as Runner Folders. With these you can execute universAAL platform without using Eclipse, in a folder in your file system, and can then pack them, move them to wherever you want, and finally run universAAL in a computer without Maven, Eclipse and all those things you had to install in order to develop. You just need Java.

An example that is ready to run
-------------------------------

You can find a template runner folder that is ready to run in https://github.com/universAAL/distro.pax or you can use the Archetype *org.universAAL.runners/template-archetype*.

You can run uAAL from a runner folder in many ways, but in order to get the example up and running you just need to execute the **Run.bat** batch file in the root folder ( or “mvn pax:run”).

That´s to let you see what you´ll get in the end. Now you´ll see the details of this runner folder and learn how to configure it to run what you want.

The Runner folder in detail
---------------------------

This is a list of everything you can find in a runner folder like the one from the example along with an explanation of what every item does:

-   **Run.bat**: This is a batch file that only contains the command “mvn pax:run”. It reads the POM file and, basically, does all the work of setting up the runner and executing it. It´s the first thing you must run when you create a runner folder. Due to its purpose, you can only run this in a computer where you have installed all the development environment of uAAL (basically, Maven). In that computer you must have access to all the bundles you have listed in the POM, with through Nexus or because you have installed them locally.
-   **pom.xml**: This is a Maven POM that lists all the bundles you want to run in this folder, and it also identifies the runner itself.
-   **pax.configuration**: A configuration file that holds properties of 2 types: properties for the pax:run plugin and properties for the JVM once OSGi is started. They match those defined in an Eclipse run configuration, if you are familiar with it.
-   **eclipse.launchers**: *(Optional)* It has a collection of .launch files that you can use if you import the runner folder into Eclipse. **More info coming soon**.
-   **runner/**: The folder where all files needed during runtime are stored.
    -   **run.bat**: This batch file runs the platform in the, let´s say, “offline mode”: You just need Java to run it, not Maven nor anything else. But to do that, first you must have run the “outside” Run.bat, the one with “mvn pax:run”. It doesn´t matter if it was executed in another computer.
    -   **remoteDebug.bat**: You can use this instead of run.bat if you want to debug the platform from a remote Eclipse. *more info about this coming soon*'.
    -   **bundles/**: *(Auto-generated afterwards)* In this folder all bundles jar files are stored once you have run the platform with the “outside” Run.bat.
    -   **log/**: *(Auto-generated afterwards)* The logs of the system are stored here (the folder may vary if you change the log config).
    -   **felix/**: This folder contains the cached Felix OSGi execution
        -   **config.ini**: It´s like a translation of the POM file to the format that Felix OSGi understands.
        -   **system.properties**: The collection of properties for the JVM (the ones you put in pax.configuration) so that Felix can read them. You can get rid of this file if you put all its properties inside the “inner” run.bat as arguments.
    -   **configurations/**: A folder that contains exactly that, configuration files for the bundles you are going to run.
        -   **factories/**: Empty for now.
        -   **services/**: Here you have a file in which you can define the log level for each class. Find more information about this [here], or look at the example in the SVN.
        -   **X.X.X/**: There are other folders in configurations/. Each of those is used by its respective bundle to find any file data it uses.
		
There are some other items that may be present or automatically generated afterwards, but you don´t usually need to deal with them.

How to create your own Runner, step by step
-------------------------------------------

First thing you need to know: You can only *create* Runner folders in uAAL-development computers, that have at least Maven installed. So make sure you have it before starting.

1.  First, it is always recommended that you copy/paste a Runner folder that you already know is working, and then modify it. You can just clone this [repository](https://github.com/universAAL/distro.pax). Another option it to use the Maven Archetype for this purpose.
2.  Edit the **pom.xml**
    1.  Change GroupId if you want, but you *must* change the ArtifactId (set a coherent name for your runner, according to the run configuration you wish to set up) *or* the version number.
    2.  Put the bundles you want into the provision list. Put them in the order you want them to start. If you use for instance the example folder, you´ll see the bundles are ordered in sections, for better reading. Which bundles you need or want to run is out of scope in this tutorial, but the example provides a good starting point.
    3.  To put a bundle on your own you follow this format: 
	```
	<provision>[wrap:]mvn:{groupId}/{artifactId}/{version} [@level] [@nostart] </provision>
	```
	where:
	- **wrap:** is only for non-OSGi bundles i.e: libraries
	- **{groupId}/{artifactId}/{version}** correspond to the Maven coordinates of the bundle|library you intend to add to the run configuration
	- **@level** optionally set the run level for the bundle|library, used to make sure they are started in the specidied order (1 first then 2,3,...)
	- **@nostart** optionally set felix not to start the bundle, it installs it but does not start it

3.  Edit the **pax.configuration** file
    1.  Usually, if you have copied the folder from one that already worked, you don´t need to change the properties.
    2.  Make sure universAAL URLs are the latest good ones.
    3.  The **--sp** property is important and may change from one Runner to another. It should usually work with **--sp=sun.reflect,sun.misc,sun.audio**, but it depends on which bundles you want to run.
    4.  The **--vmOptions** is where you put a space-separated list of VM arguments, like you would in a normal Java execution. Make sure you have all the properties you need to run the platform and your bundles. You can also redirect to the **system.properties** files with **--vmOptions=-Dfelix.system.properties=<file:felix/system.properties>**
    5.  It is a good idea to use properties from Eclipse run configurations (launches). You just need to copy the values from “run configurations/arguments”. The “Program Arguments” go to the normal "--" entries in **pax.configuration**, and the “VM arguments” go to **--vmOptions**.

4.  Make sure that in the **runner/** folder you have exactly what you need for your bundles and apps to work. Focus specially in the **configurations/** folder, but each bundle or app has its own requisites.
5.  Now you can execute **Run.bat** (in the “outer” folder) or directly type “mvn pax:run”. This will start downloading all required bundles in the POM and then will start the platform. It is recommended to do this from a previously open command line console, so that it doesn´t close if there is any error in the process.
6.  Once Felix is running, you can handle it through the console. How to use it is out of scope of this tutorial.

### Running with “outer/inner” run.bat

We call “outer” run.bat the **Run.bat** batch file that executes “mvn pax:run”, and “inner” the one inside the **runner/** folder. You can change their name if you don´t want to mistake them. Or even remove the outer one, as you can type “mvn pax:run” on your own.

The differences among these two types of running the platform is that for the first one, you must have Maven installed, an internet connection to download bundles (from Nexus or elsewhere) and, if necessary, the bundles you want installed in your .m2 local repository (if they are not deployed in uAAL Nexus yet). 

Once you have executed that one, the bundles jar files are copied to **runner/bundles** and you are able to use the “inner” run.bat to execute the platform directly. But take into account that if there is any change to the bundles (e.g. you have made changes to a bundle and mvn-installed again) you must use the “outer” run to *refresh* the bundles jar files. The “inner” one does not update anything, that is why you can use it “offline”.

### Packing up the Runner and moving to other computers

Once you have made the initial “outer” run, and you can use the “inner” run, you can do it in any computer. You just need to take the **runner** folder and move it wherever you want (where there is Java installed). Just make sure that in the **runner/felix/config.ini** file there are no absolute paths. If there are, you´ll have to change them to point to the new location.

Note that each bundle may have its own requisites. Some require MySQL to be installed when they run; others require files in specific computer folders. Make sure you have all of those in the new computer you are moving it to.

Advanced Topics
---------------

Use these if you really know what you are doing and there is no other way.

### Running from Eclipse

Of course it sounds contradictory to run from Eclipse something that was created precisely to run outside Eclipse. But you can. Just import Maven project, then point to where the POM file is and select it. When you have the folder in your Eclipse explorer, right click and select Maven build... Type “pax:run” in the “goal” field and you are good to go.

There are also some .launch files in the example folder that you can use to launch the Folder by right-clicking and choosing “Run as...”

### Versions of Felix

If you put the tag **<runner>0.17.1</runner>** (or other versions) in the POM file, just below the line **<args>pax.configuration</args>**, you will specify to use precisely that version of the pax:run plugin, which will limit to a specific version of Felix to be run. In this case, to Felix 1.4.1. However, of course, it is recommended to use the latest version, which is done by not putting that tag.

There is one issue with this, as versions may change from time to time. This may produce an error when trying to run the “inner” way. If so, make sure that the Apache Felix jar file mentioned in the “inner” run.bat has the same version as that jar file in the **runner/bundles** folder.

### Changing and updating bundles in “offline” mode

We´ve said that you must use the “outer” run to refresh the bundles whenever there is a change in the code. If you can´t or don´t want to do this, for whatever reason, you can still use the “inner”, offline run.

If you just want to update one bundle, just replace the old jar with the new one in the **runner/bundles** folder. Make sure to rename the new one to the same name than the old one.

If you want to add, remove, or change bundles you can do that as well without modifying the POM and running the “outer” way. You will have to first put the new bundles jar files in the **runner/bundles** folder. Then modify the **runner/felix/config.ini** file to include/edit/remove the file names of the jars you just added. Notice how that file as an equivalent to the “outer” POM file.

Take into account that this is not recommended, as you may end up with non-synchronized POM and .ini files. And if you execute the “outer” run the .ini file will be overwritten.

### Debugging

The best practice to Debug is using remote debugging. This way you can debug local or remote uAAL nodes.

The way to do this is to run your node just as “inner” run, instead use remoteDebug.bat (the prerequisites are the same, make sure you have runned the “outer” run first, so the latest bundles are imported).

To connect your eclipse to the node:

1.  click on Run-&gt;Debugging configurations... and add a new Remote Java Application (if you don't already have one, or if you don't want to reuse the any of the already existent)
2.  select connection type as “Standard (Socket Attach)”
3.  select the host, Typically “localhost” but if you are debugging remote nodes, insert the remote node's IP address.
4.  select the port, “remoteDebug.bat” by default uses port 5005. if you are willing to debug simultaneously several nodes on the same machine, or for any other reason you need to change the port, you must edit the port for each node in each “remoteDebug.bat” script.
5.  optionally, select Allow termination of remote VM. This will allow to kill the remote process.
6.  click on Debug, ensure your node is running before clicking.

Of course this configuration will be saved, so if you wish to relaunch you don't need to go through all the above steps.

#### Manual Configuration

Just like you ran the folder in the “Running in Eclipse” above with “Run as...”, you can use “Debug as...”. In case these don't work we will explain how to configure your own launchers.

### Packing up the folder

When you want to move the Runner to execute in other computer, you may want to zip it or directly copy it to some removable device. Runner folders can get very big, so here is a tip: Go to the folder **runner/felix/** and there remove everything *but* the **config.ini** file and the **system.properties** (if there is one).

You will probably have removed a lot of Mb of cached bundles. What you must *never* remove is the **runner/bundles** folder!!!