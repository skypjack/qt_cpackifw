# Qt meets CPackIFW

Skel for `Qt`/`CPackIFW` based build systems.

# Introduction

The web isn't full of examples of use for `CPackIFW` yet. That's a fact.<br/>
I spent a few hours configuring a minimal build system that worked on Linux, Windows and OSX. To do that I looked into the code of CMake and I opened a bug to the CPackIFW module. Then I wrote to the author of the module asking for a workaround, I must admit that he's a really nice person and he gave me a couple of hints that helped me to find the right way.<br/>
So far so good. At the end of the day I've a build system up and running that I can use almost everywhere to prepare a standalone installer for my application. That being said, I don't want to spend the same amount of time next time to pack it from scratch and that's why I created a minimal project from which to start in case.

# Requirements

Currently, `qt_cpackifw` requires:

* CMake version 3.4 or later.
* Qt Installer Framework 2.0 or later.

Unfortunately, for `CPackIFW` is still relatively young and it has been designed originally for Linux and Windows systems, it has some issues on OSX. I've documented them directly in the `CMakeLists.txt` file.<br/>
Note also that the generator expression `$<TARGET_BUNDLE_DIR:tgt>` isn't available since CMake version 3.9. Because of that, the `CMakeLists.txt` contains a few workarounds to overcome these shortcomings.

# Targets

From within the `build` directory, just run the following commands to create the package:

```
$ cmake ..
$ make package
```

# License

Copyright (c) 2017 Michele Caini.<br/>
Code released under [the MIT license](https://github.com/skypjack/qt_cpackifw/blob/master/LICENSE).
