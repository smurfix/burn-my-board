Purpose of this repository
==========================

I own a bunch of embedded boards (raspi, rockpi, gnubee, odroid, …) and
frequently forget the details on how to create a new image for them.

Yes I could download one from the manufacturer, but the upstream images
frequently come with packages I don't need, miss packages I need, require
handholding before they fit my home network, use kernels from the stone
age, and whatnot.

I only use embedded things that are supported by the mainline kernel
because all too often I want to experiment with new features.
(The Raspberry Pi is the only exception, but it follows mainline, so …)


How to use this repository
==========================

Get a serial console cable. Seriously. Personally, I recommend the
`µArt <https://www.crowdsupply.com/pylo/muart>`_ because it has galvanic
separation between host and target. Also, it takes the UART voltages from
the target, which means that if the target isn't powered neither are the
UART signals. Thus you're much less likely to fry your board.

This adapter does need power from the target: ignore any recommendations to
leave the red wire open. (Besides, on my µArt the ``Vcc`` wire is white. :-P )

Run ``sudo id``. Twice. If that doesn't work or if it asks for a password
the second time, fix it. This code calls ``sudo`` awfully often.

Run ``ls config/kernel``. If your board isn't listed, you need to add it.
See below.

Read ``doc/target/``_TARGET_ for further eludication and instruction.

Set ``export TARGET=``_TARGET_ to tell the commands in ``bin/`` which board
to build for. Alternately, the scripts accept _TARGET_ as the first
argument.

Optional: set ``export BMB_TEMP=/var/tmp/some_dir`` to create a persistent
tempdir. This does speed things up some.

Optional: Set ``export SPECIAL=something``, then create (possibly-empty)
files ``helper/apt/something`` and ``params/special/something``. You can
use this to customize your local apt repository and the script variables.
"something" may be multiple words.

Don't run these scripts as root or with ``sudo``. They're not built to
safely do this. The scripts use ``sudo`` internally for those commands
which need it.

Make sure that there's enough space in ``/var/tmp`` for two copies of your
image.

Create an image
+++++++++++++++

Read ``doc/target/``_TARGET_ for any requirements.

Call ``bin/make_image ``_TARGET_`` /var/tmp/TARGET.img``. Then copy the image to
an SD card (``sudo dd bs=1024k if=/var/tmp/TARGET.img of=/dev/sdX``).

You can also insert an SD card and run ``bin/make_image TARGET /dev/sdX``.
This is somewhat slower, particularly if you need to burn multiple images.


Add a new board
+++++++++++++++

* Add a ``doc/target/``_TARGET_ file that contains anything one needs to
  know to bring up your board. Start with how to connect the console (which
  pins? baud rate? If the serial adapter needs Vcc and the console is on a
  three-pin header, where can it get that?), add any special instructions
  if simply writing the image to an SD card or USB stick and plugging that
  in doesn't work (like u-boot variables), followed by any insights you
  gained when you did this which you'd wanted to know when you started
  and/or which you'd like your future self to know. Don't forget links to
  the documentation you used.

* Clone all external requirements to your own repository. Use the "mirror"
  function to fetch them.

* Add a ``snips/``_TARGET_ subdirectory. No, don't call it "TARGET". Use
  the name of your own board.

* Copy the most similar candidate in ``param/`` to ``param/``_TARGET_.
  Remove anything that doesn't apply for your board.

* Create a kernel repository in ../kernel, configure your kernel.
  Then do ``git checkout -b ``_TARGET_ and ``cp .config
  ../burn-my-board/config/kernel/``_TARGET_. Don't forget to add and commit
  this new config file.

* Start by running ``bin/make_image ``_TARGET_`` /var/tmp/test.img``. You'll probably
  note a bunch of failures, including some code you'll have to add to
  ``snips/``_TARGET_``/``. Fix them. ;-)

* That worked? Good. Proceed with "Create an image", above.

* Add a file to ``doc/authors/``_TARGET_. The "source" tag should point to
  your repository. Add ``#BRANCH`` at the end if it's not ``master``.

* Create a pull request so that I can integrate your extension.


Shell helpers
+++++++++++++

This section briefly documents the helper variables and functions you can use in your script
snippets.

* T
  The base path of a temporary directory. Mount your file systems in
  subdirectories of this directory. They'll be unmounted automatically.

* TARGET
  The name of the device you're setting up. Set from the first argument to your
  script (when that starts with a colon), or iniherited from the environment.

* SPECIAL
  The name (or names) of a file in  ``helper/special/``. These are executed
  at the end of building a root image and can be used to add any local
  stuff you need in your environment (your public SSH key, your apt
  repository, …)

* C
  When you want to run a command within the target root, use ``$C command
  args…``. This resolves to an appropriate invoction of ``systemd-nspawn``.

* loopback

  Get a loopback device. Accepts the same arguments as ``losetup``, most
  notably including ``-P``. Will get wrapped so auto de-looping works.

* mount

  Alias for "mount". Usage: destdir source any_mount_options.
  Will get wrapped so that auto unmounting works.
  If the source is a directory, does a bind mount.

* cleanup

  Set this function after including ``helper/common`` to do script-specific
  additional cleanup.


Special env vars
++++++++++++++++

* MENU
  Set (to anything) if you want a kernel "make menuconfig" to run

* BMB_TEMP
  Set to some directory to keep the tempfiles around between invocations.

Subdirectories
++++++++++++++

* bin
  User-executable scripts

* config/kernel
  Kernel configuration for each target.

* config/uboot
  Configuration for U-Boot.

* helper
  Common parts of shell scripts, *not* separated by target

* helper/special/SPECIAL
  Author-specific image setup code, executed as the last step of building
  the root file system. Do ``export SPECIAL=foo bar`` and write your code
  to ``helper/special/foo`` and ``…/bar``.

* mirror
  Storage for local git clones of supporting archives.

* param
  Files with shell variables for each target. Generally not configurable.
  
* snips
  Per-target code for various device-specific features. Not directly
  callable.

* apt
  Per-target code to add vendor-supplied additional repositories.

* doc/snips/*
  Documents what the helper scripts in ``snips/``_TARGET_``/*`` do.

* doc/bin/*
  Documents what the scripts in ``bin/`` do.

