EmulationStation
================

A graphical front-end for emulators with controller navigation. Developed both on and for the Raspbery Pi. Intended for use with RetroArch, but it can easily be used with other emulators.

RetroArch for the Raspberry Pi can be found here: https://github.com/ToadKing/RetroArch-Rpi
I'm not associated with RetroArch in any way!

A cool guy named petrockblog made a script which automatically installs RetroArch, its cores, and ES. It also includes options for configuring your RPi and setting it up to boot directly into ES. You can find it here: https://github.com/petrockblog/RetroPie-Setup

Building
========

**On the Raspberry Pi:**

EmulationStation has a few dependencies. For building, you'll need SDL 1.2, SDL_mixer, FreeImage, FreeType, and Boost.Filesystem, which can easily be obtained with apt-get:
```
sudo apt-get install libsdl1.2-dev libboost-filesystem-dev libfreeimage-dev libfreetype6-dev libsdl-mixer1.2-dev
```

There are also a few libraries already on the RPi (located in /opt/vc/, like the Broadcom libraries, EGL, and GLES). You can build EmulationStation by simply running `make`.


**On something else (desktop):**

EmulationStation can also be built on a "normal" Linux system. You'll need the same libraries listed above:
```
sudo apt-get install libsdl1.2-dev libboost-filesystem-dev libfreeimage-dev libfreetype6-dev libsdl-mixer1.2-dev
```

You'll also need OpenGL. I don't know the proper package name, but you'll need `/usr/include/GL/gl.h` and `libGL`. You probably already have it. You can build with `make -f Makefile.x86` (badly named Makefile, I know).

Configuring
===========

**~/.emulationstation/es_systems.cfg:**
When first run, an example systems configuration file will be created at $HOME/.emulationstation/es_systems.cfg. This example has some comments explaining how to write the configuration file, and an example RetroArch launch command. Keep in mind you can define more than one system! Just use all the variables again. Also, you can use multiple extensions - just separate them with a space, e.g.: ".nes .NES .bin".

If an SDL Joystick is detected at startup, and $HOME/.emulationstation/es_input.cfg doesn't exist, an Input Configuration screen will appear instead of the game list.
This should be pretty self-explanatory. If you want to reconfigure, just delete $HOME/.emulationstation/es_input.cfg.

Mappings will be applied to the first joystick with the same name as the joystick you configured.
An Xbox 360 controller with the xboxdrv driver was used for testing. POV hats are automatically mapped to directions (so if you're not using an analog stick, you'll need to skip mapping Up/Down/Left/Right by pressing a keyboard key).

**Keep in mind you'll have to set up your emulator separately from EmulationStation.**
If you're using RetroArch, a handy input config generation tool can be found in the tools/ subdirectory - you can use it with `retroarch-joyconfig -o ~/.retroarch.cfg` or something similar. You may need to tell RetroArch to load this config file with `-c ~/.retroarch.cfg` in your RetroArch launch commands.

EmulationStation will return once your system's command terminates (i.e. your emulator closes).


**Keyboard mappings:**
```
Up - Scroll up

Down - Scroll down

PageUp or ] - Page up

PageDown or [ - Page down

Left - Previous system (if it exists)

Right - Next system (if it exists)

Enter - Select

Escape - Back out of a folder

F1 - Open the restart/shutdown system menu

F2 - Open the fast select dialog

F4 - Close EmulationStation (should work as long as ES hasn't frozen)
```

Unfortunately, there is no built-in way to change keyboard mappings - if you need to, check out `src/InputManager.cpp`. There's a switch statement with a list of keys; it should be pretty simple to change them.

You can use `--help` to view a list of command-line options. Briefly outlined here:
```
-w [width]		- specify resolution width.
-h [height]		- specify resolution height.
--gamelist-only		- only display games defined in a gamelist.xml file.
--ignore-gamelist	- do not parse any gamelist.xml files.
--draw-framerate	- draw the framerate.
--no-exit		- do not display 'exit' in the ES menu.
--debug			- print additional output to the console, primarily about input.
--dimtime [seconds]	- delay before dimming the screen and entering sleep mode. Default is 30, use 0 for never.
```

Writing an es_systems.cfg
=========================
The file `~/.emulationstation/es_systems.cfg` contains the system configuration data for EmulationStation. A system is a NAME, DESCNAME, PATH, EXTENSION, and COMMAND. You can define any number of systems. You can switch between them by pressing left and right. They will cycle in the order they are defined.

The NAME is what ES will use to internally identify the system. Theme.xml and gamelist.xml files will also be searched for in `~/.emulationstation/NAME/` if not found at the root of PATH. It is recommended that you abbreviate here if necessary, e.g. "nes".

The DESCNAME is a "pretty" name for the system - it show up in a header if one is displayed. It is optional; if not supplied, it will copy NAME (note: DESCNAME must also *not* be the last tag you define for a system! This is due to the nature of how optional tags are implemented.).

The PATH is where ES will start the search for ROMs. All subdirectories (and links!) will be included.

**NOTE:** A system *must* have at least one game present in its PATH directory, or ES will ignore it.

The EXTENSION is a list of extensions ES will consider valid and add to the list when searching. Each extension *must* start with a period. The list is delimited by a space.

The COMMAND is the shell command ES will execute to start your emulator. As it is evaluated by the shell (i.e. bash), you can do some clever tricks if need be.

The following "tags" are replaced by ES in COMMANDs:

`%ROM%`		- Replaced with absolute path to the selected ROM.

`%BASENAME%`	- Replaced with the "base" name of the path to the selected ROM. For example, a path of "/foo/bar.rom", this tag would be "bar". This tag is useful for setting up AdvanceMAME.

gamelist.xml
============

If a file named gamelist.xml is found in the root of a system's search directory OR within `~/.emulationstation/%NAME%/`, it will be parsed and the detailed GuiGameList will be used. This means you can define images, descriptions, and different names for files. Note that only standard ASCII characters are supported (if you see a weird [X] symbol, you're probably using unicode!).
Images will be automatically resized to fit within the left column of the screen. Smaller images will load faster, so try to keep your resolution low.
An example gamelist.xml:
```
<gameList>
	<game>
		<path>/home/pi/ROMs/nes/mm2.nes</path>
		<name>Mega Man 2</name>
		<desc>Mega Man 2 is a classic NES game which follows Mega Man as he murders eight robot masters.</desc>
		<image>/home/pi/Screenshots/megaman2.png</image>
	</game>
</gameList>
```

The path element should be the absolute path of the ROM. Special characters SHOULD NOT be escaped. The image element is the path to an image to display above the description (like a screenshot or boxart). Most formats can be used (including png, jpg, gif, etc.). Not all elements need to be used.

The switch `--gamelist-only` can be used to skip automatic searching, and only display games defined in the system's gamelist.xml.
The switch `--ignore-gamelist` can be used to ignore the gamelist and use the non-detailed view.

**Making a gamelist.xml by hand sucks, so a cool guy named Pendor made a python script which automatically generates a gamelist.xml for you, with boxart automatically downloaded. It can be found here:** https://github.com/elpendor/ES-scraper

I found a bug!
==============

The best way to report a bug is to post an issue on GitHub. Try to post the simplest steps possible to reproduce the bug. Include files you think might be related (except for ROMs, of course). If you haven't re-run ES since the crash, the log file `~/.emulationstation/es_log.txt` is also helpful.

Themes
======

By default, EmulationStation looks pretty ugly. You can fix that. If you want to know more about making your own themes (or editing existing ones), read THEMES.md!

I've put some themes up for download on my EmulationStation webpage: http://aloshi.com/emulationstation#themes

-Aloshi
http://www.aloshi.com
