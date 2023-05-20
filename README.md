# Cruster

This project aims to create an application that makes it easy to set up clusters for rendering Blender files. For the sake of learning the language, I'm going to implement it in Rust for now, although in the long term a reimplementation in Python is probably a better choice (as Blender offers a Python API and using Python would also open the door to creating a Blender plugin).  


## Plan

The basic plan for the moment is to have a client application that is used to set up the cluster and feed work to the server applications on the worker nodes, which then pass on their workload to Blender by executing Blender's command line commands.  

### Client

A user will mostly interact with the client application to distribute the work to the cluster. At the beginning it is going to be run via the command line, but of course a GUI would also be nice eventually. The user will need to specify the .blend file that is to be rendered, some amount of options (like the frame range, render engine and where the files are to be stored locally) and the list of the worker nodes (IP addresses / names and ports). Then the client application sends the .blend file to the worker nodes and balances the load between them by telling them which frames to render (so you will only be able to distribute load among n nodes if you have at a minimum n frames, at least until some advanced functionality of the Python API is used). Once worker nodes are done rendering their respective frames they are received by the client and saved to the specified location. Of course idle nodes get fed more work if there is any. 

### Server

The role of the server is basically to receive requests from the client and execute Blender CLI commands accordingly and send the image file back to the client.  


## Potential problems

Some problems of choosing this approach are already apparent:  

	- Limited amount of configurability using Blender via the CLI. Therefore the user will need to set the options that are not offered via CLI in the .blend file itself. This may make using a standalone GUI somewhat annoying, as one would still need to jump back and forth between Blender and the client UI (and Blender also needs to be installed on the client, which it probably is, but otherwise wouldn't need to be).
		- This also makes it necessary for the .blend file to be copied over to the worker nodes again for every single setting change. Depending on the size of the .blend file and connection speed to the nodes, this may make experimenting with settings very cumbersome.
	- Rendering using mix of CPU and GPU rendering. It's not clear to me at the moment what the behaviour is going to be if the .blend file is set up to use some method of GPU rendering and the worker nodes only support another method of GPU rendering or only CPU rendering. Maybe it won't be possible for the worker nodes to set their optimal rendering method if it doesn't match the .blend file.
	- Only animations with multiple frames can be distributed. Without tapping into more advanced functionality that the Python API may offer, it's not possible to split a single frame into fragments to render in a distributed way.
