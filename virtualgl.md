# Remote Rendering with VirtualGL

In order to use our [rendering and visualization toolkit](https://www.github.com/BerkeleyAutomation/meshrender) with hardware acceleration over SSH, we use [VirtualGL](https://www.virtualgl.org/).
This guide explains why VirtualGL is necessary and how to install and configure it properly.

## Why is remote rendering difficult?

Our rendering and 3D visualization toolkit relies on OpenGL 3+, a standard API for rasterization-based graphics rendering.
OpenGL is not a software package -- it's just an API, and many implementations exist.
This means that OpenGL can be implemented entirely in software on the CPU, or it can
take advantage of GPU resources to accelerate rendering.

Ideally, we'd like to be able to take advantage of the significant GPU resources available on either our lab's machines or on EC2 GPU instances to accelerate rendering.
However, this is difficult to without specialized software.
When OpenGL commands are issued in a program run over SSH, the default behavior on most systems is to send the OpenGL commands back to the client machine and have the client's OpenGL implementation actually perform rendering.
This is called indirect rendering, and it's undesirable for a few important reasons.
First, this behavior prevents us from taking advantage of our remote server's GPU resources and negates the usefulness of having a remote server in the first place.
Second, only OpenGL 1.4 commands are supported via SSH, which removes the ability to use modern OpenGL features such as shaders.

In order to take advantage of both modern OpenGL and remote hardware acceleration, we use [VirtualGL](https://www.virtualgl.org/), an open-source toolkit for enabling direct hardware rendering over SSH.
VirtualGL allows us to run programs on the server that will use the server's X11 display, OpenGL implementation, and hardware resources for rendering and only sends rendered images to the client machine (if needed).
For more information, see [here](https://cdn.rawgit.com/VirtualGL/virtualgl/2.5.2/doc/index.html#hd003).

## Installing VirtualGL on the server

Full installation instructions for VirtualGL are provided [here](https://cdn.rawgit.com/VirtualGL/virtualgl/2.5.2/doc/index.html#hd005).
If your server is running Ubuntu 14.04 or 16.04, installation is fairly easy.
Download a .deb from SourceForge [here](https://sourceforge.net/projects/virtualgl/files/2.5.2/virtualgl_2.5.2_amd64.deb/download) or via this [direct link](https://downloads.sourceforge.net/project/virtualgl/2.5.2/virtualgl_2.5.2_amd64.deb?r=https%3A%2F%2Fsourceforge.net%2Fprojects%2Fvirtualgl%2Ffiles%2F2.5.2%2F&ts=1513547244&use_mirror=astuteinternet).
Go to the directory where you downloaded the .deb file and run the following command:

```
dpkg -i virtualgl*.deb
```

Next, set up and configure the VirtualGL server.
If your Ubuntu instance doesn't have a display manager, install one (either GDM or LightDM, usually).
Then, stop the display manager.
For GDM, run `/etc/init.d/gdm stop`, and for LightDM, run `service lightdm stop`.
Then, run the following command:
```
/opt/VirtualGL/bin/vglserver_config
```
Select option one, then respond no to all of the following propmpts. Finally, restart the display manager with `/etc/init.d/gdm start` or `service lightdm start`.

Run `xdpyinfo -display :0` -- it should run with no errors.

## Installing VirtualGL on the client

Follow the [instructions](https://cdn.rawgit.com/VirtualGL/virtualgl/2.5.2/doc/index.html#hd005) for installing VirtualGL on your client machine.

If your remote machine is being accessed via port forwarding,
you will need to edit the `vglconnect` script.
On Mac OS, this is located at `/opt/VirtualGL/bin/vglconnect`.
On any line that starts with `$SSHCMD`, you'll find a string that looks like this:
```
${1+"$@"}
```
Immediately after that string, add `-p PORTNUM`, with `PORTNUM` replaced with your target port.
For example, if the target port was 8028, the new string would look like this: 
```
${1+"$@"} -p 8028
```

Now, instead of directly SSHing into your target machine, use `vglconnect`:
```
/opt/VirtualGL/bin/vglconnect -s {user}@{server}
```

On the server, preface any application that you want to run with `vglrun`, i.e.
```
vglrun python mygraphicalscript.py
```

As a test, run
```
vglrun glxgears
```

This should pop a window containing rotating 3D gears on your local machine.

## Drawbacks and Alternatives

One major drawback of this setup is that the client must remain connected via SSH for the server's application to continue to run -- disconnecting can cause the application to crash.
This isn't really a problem when using live visualization, but it can be annoying for long-term offscreen rendering.

As an alternative to using VirtualGL, a hack is to hijack the server's local X screen.
This won't send any OpenGL output back to the client and is therefore unsuitable for visualization, but it allows offscreen rendering to be done in `screen`, even when the session disconnects.
In order to do this, simply run the following commands:
```
export DISPLAY=:0.0
sudo xhost + local:
```

Then, run whatever application you want normally (i.e. no `vglrun`). You won't be able to see the output of the application, but that's fine for offscreen rendering.

