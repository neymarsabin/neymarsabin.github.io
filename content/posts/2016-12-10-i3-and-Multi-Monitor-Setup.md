+++
layout = 'post'
type = 'blog'
title = 'Multi-Monitor and i3 window manager'
date = 2016-12-10T12:00:00+05:45
+++

**i3** is a tiling window manager.

> A window manager is a system software that controls the placement and apperarance of windows within 
> a windowing system in a graphical user interface(GUI). It can be part of a desktop environment or be used standalone.

<!--more-->
And, 

> Tiling window manager is a window manager with an organization of the screen into manually non-overlapping frames.

Confusing?? Try i3! Give it a shot and u will find out.

Various operations in i3wm are performed using the keybindings and it's fast.One of the greatest feature is multi-monitor 
support.I was always inspired by the english movies ( especially those hacker movies) where a guy sits in a basement 
with multiple monitors in front of him. Those scenes were really cool but i always wondered how do they do it!
Of course, they might not be using **i3wm** but that can be done using i3 window manager.
Other linux desktop environments also provides the feature of multiple monitors but i think they are not as awesome as of 
i3wm.
I have been using i3wm since some months and i find it really awesome.Especially with it's multiple monitor support.
Connect a new monitor and that's it.It provides you with a new empty workspace.
Use your keybinding,jump to any workspaces and monitors, do some task, open some windows, check some outputs, copy paste something,use IRC or Browse,presentations.
If you are a command line lover you may wanna use **xrandr** to configure those multiple monitors/displays.There are some GUI 
applications too like **lxrandr** and **arandr**.
After connecting the display, use this command to find out the information to configure.

    xrandr --query

You may get output like this:

    Screen 0: minimum 8 x 8, current 2732 x 768, maximum 32767 x 32767
    eDP1 connected 1366x768+1366+0 (normal left inverted right x axis y axis) 340mm x 190mm
    	 1366x768      60.10*+
    	 1024x768      60.00
    	 1024x576      60.00
    	 960x540       60.00
    	 800x600       60.32    56.25
    	 864x486       60.00
    	 640x480       59.94
    	 720x405       60.00
    	 680x384       60.00
    	 640x360       60.00
    DP1 connected 1366x768+0+0 (normal left inverted right x axis y axis) 340mm x 190mm
    	 1366x768      59.79*+
    	 1280x720      60.00
    	 1024x768      60.00
    	 800x600       60.32
    	 640x480       59.94
    HDMI1 disconnected (normal left inverted right x axis y axis)
    HDMI2 disconnected (normal left inverted right x axis y axis)
    VIRTUAL1 disconnected (normal left inverted right x axis y axis)

Note the names of the connected ones and their supported/used resolutions.In my case those are **eDP1** and **DP1**.
Now if I want to place **eDP1** to the right of **DP1** then i would run the following command: 

    xrandr --output DP1 --mode 1366x768 --output eDP1 --mode 1366x768 --right-of DP1 

If you want to mirror both of these displays then replace &#x2013;right-of in above command with &#x2013;same-as 

    xrandr --output DP1 --mode 1366x768 --output eDP1 --mode 1366x768 --same-as DP1 

GUI applications also can be used to perform these applications easily.


# 

> i3 was specifically developed with support for multiple monitors in mind.

So, when connected with multiple monitors, each monitors get's an initial workspaces as i previously mentioned.
Monitor 1, monitor 2 and monitor 3 gets workspace 1 , workspace 2 and workspace 3 respectively if they are connected in that order.
As,i3wm has special keybindings to switch between workspaces, you are automatically switching between monitors also.That'
easy isn't it.

So, one last thing I am going to mention is how to move workspaces to other screen. 
In you i3 config file generally **~/.config/i3/config** 

    bindsym $mod+n move workspace to output right

So, now i can easily move the workspace to another screen using the **$mod+n** keybinding as I have placed my monitor to the right.


# Sources

-   [i3wm-Docs-for-Multiple-Monitors](http://i3wm.org/docs/userguide.html#multi_monitor)
-   [Arch wiki guide for xrandr and configuration- ](https://wiki.archlinux.org/index.php/xrandr)
-   If you want to know more about Tiling window managers [Wikipedia guide for tiling window manager](https://en.wikipedia.org/wiki/Tiling_window_manager)
-   [Arch Wiki guide to i3 window manager](https://wiki.archlinux.org/index.php/i3)
-   [Best documentation is available at i3wm's own site](https://i3wm.org/)
