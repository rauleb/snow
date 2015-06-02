# watch
There have been several attempts at creating a file system watcher for the Go ecosystem. Below are a few that i've encountered in search for one that satisfied my needs:

- https://github.com/go-fsnotify/fsnotify
- https://github.com/howeyc/fsnotify
- https://github.com/jpillora/spy
- https://github.com/gokyle/fswatch
- https://github.com/rjeczalik/notify
- ... and probably a few more

I wanted a library that could live up to the following requirements:

- Work flawlessly on OSX by default: no "too many files open"
- Be able to watch directories with thousands of files in them. As a target, it should work on the Linux source code repository. 
- The simplest possible abstractions, no pipelines, filters or other shenanigans 
- Support recursive watching out of the box, but provide some configuration that can prevent some or all subdirectories from being watched.
- Identical behaviour no matter the underlying system.
- Identical behaviour of the abstraction across OSX, Windows and Linux.

##The Problem
Above requirements seem reasonable but are difficult to meet in practice. As [some](https://github.com/howeyc/fsnotify/issues/54) [discussions](http://lists.qt-project.org/pipermail/development/2012-July/005279.html) have pointed out, the root of the issue lies in the following table of the ideal subsystems for each platform:

platform | subsystem | recursive | event file details 
--- | --- | --- | ---
Linux | inotify | no, not configurable | high
Windows | ReadDirectoryChangesW | configurable | high
OSX | FSEvents | yes, not configurable | low

As you may notice, OSX is the main culprit. It forces the implementation to be recursive by default and doesn't provide specifics on how a file changed.

##The Solution
In my opinion one has to simply accept the limitations of FSEvent and use their "something happened in a directory" as the abstraction. This effectively delegates the logic for on how to handles events for specific files in that directory to the consumer of the library. 

In practice, this actually makes sense. It often up to the implementation to determine what event constitutes a file change anyway: 

- did the file content actually change or just the timestampe? 
- what do renames actually mean, is it another file or was it moved?
- when files are moved outside the monitored directory, should those be considered as removals?
- what about atomic saves that some IDE's use, are those truly two events or do you want to handle them as a file modification?

## The interface
<wip>