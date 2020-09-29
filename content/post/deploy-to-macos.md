---
title: Deploying native apps to macOS
date: 2018-06-09T00:23:10+00:00
author: "Taras Kushnir"
image: deploy-mac.jpg
categories:
  - Programming
keywords:
  - macos
  - qt
  - native
  - package
  - deploy
  - dynamic
aliases:
  - /2018/deploy-to-macos
---

This is part 1 of the deployment process overview. [Part 2]({{< relref "sign-for-macos" >}}) will cover code signing and notarization.

## Level 0: warming up

Deploying anything anywhere has never been easy business. Deploying applications to desktop computers is no different. OS X (now macOS) had its own solution of ["dll hell"](https://en.wikipedia.org/wiki/DLL_Hell): each application should be absolutely self contained. All the dependencies and dependencies of dependencies should be distributed within the "bundle" so apps won't conflict with other apps because of missing or outdated libraries.

Sounds good in theory, but what does it mean in practice? You have to collect all the dependencies somehow and fit them into your "bundle". Bundle structure usually looks like this:

    HelloWorld.app/
    - Contents/
      - Frameworks/
      - MacOS/
        - HelloWorld (executable)
      - Resources/
      - PlugIns/

Bundle is just a directory with suffix ".app" which magically converts it to the "bundle" where real executable is inside `Contents/MacOS/` directory, its library dependencies live in `Frameworks/` directory and other resources in other appropriate places (`Resources/` or `PlugIns`).

To understand which dependencies does your `HelloWorld` executable have, you can use `otool` command line utility (part of XCode) with parameter `-L`:

    > otool -L HelloWorld.app/Contents/MacOS/HelloWorld

        @rpath/QtNetwork.framework/Versions/5/QtNetwork 
        @rpath/QtCore.framework/Versions/5/QtCore 
        /System/Library/Frameworks/IOKit.framework/Versions/A/IOKit 
        @rpath/QtGui.framework/Versions/5/QtGui 
        /usr/lib/libc++.1.dylib 
        /usr/lib/libSystem.B.dylib 
        .....

It lists all dependencies of your app and their "install names". You may ask what is `@rpath`? It's a special variable (a friend of `@executable_path` and `@loader_path`) that means a list of locations where the dynamic linker can find dependent dylib at runtime. Items in list usually are set by a linker and can overriden in a number of ways (e.g. by `install_name_tool` or with environmental `DYLD_LIBRARY_PATH` variable). For native applications it could be `/usr/lib`. For Qt applications it could be `/path/to/Qt5.6.2/5.6/clang_64/lib` or anything alike.

You can learn this path also using `otool` but this time with parameter `-l` and checking for `LC_RPATH` block:

    > otool -l HelloWorld.app/Contents/MacOS/HelloWorld

        ......
        Load command 24
          cmd LC_RPATH
          cmdsize 56
          path /Users/user/Qt5.6.2/5.6/clang_64/lib (offset 12)
        ......

Also `otool -l` is pretty useful command to learn how does your application start anyway. How does it know it's depedencies and so on (yes, they are all just listed in your app).

So your user's computer probably will not have same path as the one on your computer so you have to change it. Remember bundle structure couple of passages above? All the libraries should live in `Frameworks/` directory so your application `@rpath` will point to `Frameworks/` and will be able to start using the libraries that come along.

In order to change `@rpath` (and other sections) you need `install_name_tool` utility (also comes with XCode) so in the end your app's `@rpath` will have at least one entry of `@executable_path/../Frameworks/` (where `@executable_path` is another magic variable with self-descriptive name).

### Qt world

If you're bundling a Qt app you can make use of `macdeployqt` utility which comes with Qt distribution and it can pretty much pack all the frameworks and libraries required for your app and missing in `/usr/lib/`.

    macdeployqt HelloWorld.app -executable="HelloWorld.app/Contents/MacOS/HelloWorld" -qmldir=../src/

This command will bundle all the dependencies of the executable passed to it via command line. In addition this app will change `@rpath` using `install_name_tool` and will produce ready to deploy bundle. Although this will only work if your application depends on some system and Qt libraries and nothing else.

## Level 1: additional files

If your application depends on other resources (icon, videos, dictionaries etc.) that are not compiled into the executable, you also have to ship them using `Resources/` directory. I recommend creating a deployment script that will do that for you of course because doing it manually every time is very error-prone.

Deployment as easy as:

    cp /path/to/your/file HelloWorld.app/Contents/Resources/

## Level 2: custom libraries

This is harder. Simple copying library or framework to `Frameworks/` in order to deploy it is not enough. Your executable will look for the library in the place that was set by the linker and chances are it's not `@rpath`, but something like `/usr/local/lib/mycustomname.dylib`. Things get more complicated if the library requires other dynamic libraries so you will need to deploy transitive dependencies as well.

Bottom line is: in order to deploy custom libraries you have to first copy them to `Frameworks/`, fix library load path in your app and fix the librarie's `LC_LOAD_DYLIB` sections (like `LC_RPATH` but for dynamically linked libraries) where its own dependencies are listed. Rough example:

    # copy library to Frameworks
    cp /path/to/library.dylib HelloWorld.app/Contents/Frameworks/

    # fix dependency of the main app if needed
    install_name_tool -change "/previous/link/path/to/library.dylib" "@rpath/library.dylib" HelloWorld.app/Contents/MacOS/HelloWorld

    # and dependencies of dependencies of the library (repeat for every $depend_lib)
    install_name_tool -change "/usr/local/lib/$depend_lib" "@loader_path/$depend_lib" "$lib"

Previous link path to the dependent library can be learned using `otool -L` command against your executable as well as library dependencies. If you put all the dependencies on the same level together then they can be referred as `@loader_path/dependentlibrary.dylib` where `@loader_path` is magical variable expanded to the path of the library which caused current library to be loaded.

## Level 3: additional applications

Say you need some additional application like a [crash handler](https://github.com/ribtoks/xpiks/blob/master/src/recoverty/README.md) to be deployed alongside with your main one. If you were skimming this post thoroughly then you know the answer: you copy its dependencies to `Frameworks/` and tweak `@rpath` using `install_name_tool`.

To be more precise first you need to choose where exactly your app will be with regards to the main app. Usually it's put alongside with it in the `Contents/MacOS` directory like this:

    HelloWorld.app/
      - Contents/
        - Frameworks/
        - Resources/
        - MacOS/
          - HelloWorld (main exe)
          - MyHelper.app/
            - Contents
              - MacOS/
                - MyHelper (additional exe)
              - Frameworks/
              - Resources/

Of course you can first create a fully self-contained bundle of your other app, but this will only increase total size of the main bundle. If you want your other app to have an icon you will anyway need to create a bundle, but no need to copy dependencies in there.

What makes more sense is to reuse dependencies of the main app as much as possible (usually they cover smaller one). In order to do so you will need to tweak `@rpath` of the smaller executable to point to `Frameworks/` directory of the parent. Also sounds like a job for `install_name_tool` and a fresh couple of lines in your deployment script like this:

    install_name_tool -add_rpath "@executable_path/../../../../Frameworks" "${ADDITIONAL_APP}/Contents/MacOS/OtherAppName"

### Qt world

If your small additional application is also a Qt application (which is likely if the main one is) then above is not enough. Problem is that Qt apps need also platform-specific plugins and resources shipped alongside with them. If your main app already has them deployed using `macdeployqt` then you can reuse them using `qt.conf` file. This is a magical file that `QApplication`-descendent classes look for in order to learn about the environment, parameters, paths to plugins and resources and what not.

So to reuse resources and platform-specific plugins of the main app you would need to create a `qt.conf` in `Resources/` directory of the dependent app with relative paths like this:

    > cat << EOF > "${ADDITIONAL_APP}/Contents/Resources/qt.conf"
    [Paths]
    Plugins = ../../../PlugIns
    Imports = ../../../Resources/qml
    Qml2Imports = ../../../Resources/qml
    EOF

Where relative paths point to parent's appropriate directories.

## Bonus level: DMG creation

Of course it would be too easy if bundling an `.app` was everything that you needed to ship your application. In Apple world applications are usually shipped in `.dmg` files which are Apple Disk Image files. Good news is that if you only want to compress `.app` to `.dmg` you need pretty much one command:

    > hdiutil create -srcfolder "/path/to/HelloWorld.app" -volname "HelloWorld" -fs HFS+ \
    -fsargs "-c c=64,a=16,e=16" -format UDRW -size ${SIZE}m "HelloWorld.dmg"

Bad news is that it will lead to bad user experience. Imagine you open an installer and what you see is just ugly white background and some `HelloWorld.app` in top left corner left totally at your disposal. You should guess yourself what you want to do with it. Probably just close or trash immediately.

![Raw DMG](/img/bundle-raw.png)
*Pretty much RAW bundle created with a command above*

What people usually do is they set a pretty background image with some instructions and place a link to `Applications/` directory. Instructions usually visually explain to drag the `HelloWorld.app` to `Applications` in order to install it.

![Prettified DMG](/img/bundle-background.png)
*Bundle with set background and link to Applications*

In order to do so you have to copy background file to `.background` directory of the DMG, then select it in Finder as background image, resize the window and icons inside to fit it. Mount your first dmg, do these manipulations and then reexport it from `Disk Utility` to read-only dmg. You can also automate these actions with a Apple Script but that's a topic for another post.

## The end

As you can see deploying desktop apps on macOS is a total hassle as soon as you have something more complex than `HelloWorld.app`. XCode/QtCreator can rule many things out to an extent but custom libraries, additional applications and others may require writing custom deployment scripts.

Make sure to read [part 2]({{< relref "sign-for-macos" >}}) about signing and notarizing for macOS.

## References

* [Xpiks deployment script](https://github.com/ribtoks/xpiks/blob/backports_1.5.3/scripts/deploy/deploy_mac.sh)
* There's an awesome article about [dynamic loading in Linux](https://amir.rachum.com/blog/2016/09/17/shared-libraries/). It explains about `rpath`, `runpath` and other quirks.
* [Linking and install names on OS X](https://www.mikeash.com/pyblog/friday-qa-2009-11-06-linking-and-install-names.html)
* [Deploying Qt applications](https://doc.qt.io/qt-5/deployment.html).
