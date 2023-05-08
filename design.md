# Backend - Audio Server

The brains of the operation. While being named as a server, it really just lives as a library built with/into the game. The engine bindings will make raw FFI calls to this server to communicate what sounds to play, how to play them (position, parameter pointers, etc.) The Audio Server **must** never write data to any parameter pointer passed. Additionally, this will communicate any required visualization data with the Live-Reload server, and will accept changes to basic values in the project from the Live-Reload server. Most likely (but not definitely) limited to changing numerical values. Adding new sounds, or writing over sounds would not be supported in an initial version, at the very least.

Additionally, this server is responsible for playing audio, and performing signal processing operations to make it sound like it's good and fun and 3D. It will also be responsible for enumerating audio devices, communicating said devices to the engine bindings so that an audio device may be selected, and adjusting it's own capabilities on the fly with the audio device selected. Audio device should also be able to be changed on the fly without issues.

The engine bindings must communicate 1 project file to the audio server per instance started (not sure if multiple parallel instances will ever be officially supported from the same executable (nor should it be needed), but i don't see any reason to make it flat out impossible?). The audio server will verify that the project was signed by the DAW frontend, and will crash in the event that the data passed cannot be verified to be from a frontend. This keypair will be public, open, and will never be changed! (except for in the event of breaking changes to the file spec, maybe.) In the event that you'd like to ensure that any project file passed into the audio engine from engine bindings are from *your* team specifically, you may generate a keypair in the DAW to sign the project files with, and rebuild this server with your generated public key. Reccomend against this, however! The possibility of modding audio is nice :)

# Debug-Only Backend - Live-Reload server

This is a component of the main audio server only built into lib\[name\]-devel. release versions of the library will not include this feature in any capacity.

In practice, this is just a websocket server. It will recieve requests to edit parameter data from a DAW frontend, and will pipe back information, alongside information about what reflective objects exist in the scene for an audio visualizer on the client side. 

Because this is a server where many clients can be connected, every single parameterized value must be treated as a mutex. Clicking into a given value will lock it for anyone else editing, and a lock must be released before another editor can take control of that same value. As an example - the low and high bounds of a value being read from the game for example, are two separate values. One editor can change the low bound while another changes the high bound, but you cannot have two editors act on just the low or high bounds.

Spatial audio data is in the form of very primitive objects. Planes, extruded planes, cyllinders, capsules, spheres is the limit of these primitive shapes, most likely. audio sources are single points. Visualization will show all reflective objects in the scene, and a visual representation of how sound is being propagated throughout a scene.

# Sound Designer's Frontend - DAW Client
See DAW design doc, I have nothing high level to say here.

# Progammer's Frontend - Engine bindings
Engine bindings are what translate raw function calls into lib[name]\(-devel), and translate lib[name]\(-devel) datatypes into whatever datatypes the engine might prefer to use. This layer is very implementation specific. A lower-level engine such as bevy might choose to just expose raw datatypes and functions, or maybe even just include [name] as a library itself (given [name] will be written in rust.) Bindings for higher-level engines, such as your Unity, your Unreal, your Godot may choose to abstract some lower-level features of the engine behind easier to use methods. At the end of the day, this is what translates engine datatypes into [name]-native datatypes.
