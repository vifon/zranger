zranger
=======

[zsh](http://www.zsh.org/) and [ranger](http://ranger.nongnu.org/) integration

DESCRIPTION
-----------

`zranger` allows easy and quick switching back and forth between `zsh`
and `ranger` while synchronizing their working directories and
preserving the commandline contents. The `ranger` process is kept in
background (using `tmux`) after the first launch so the switching is
almost instantaneous. The new process is created only on the first run
or when the old one is explicitly closed (instead of detaching).

It's an alternative approach to the problem I've been trying to solve
with [deer](https://github.com/vifon/deer): a fast access to a
ranger-like environment used as an extension of shell.

FEATURES
--------

**Laziness**

The `ranger` process is started on the first call to `zranger`. No
need to launch a new `ranger` process for each and every shell means
almost no shell startup time penalty.

**Just works!**

`zranger` mostly reuses the pre-existing mechanisms. Thanks to that
most of the things just work. Want to close the terminal window with
that button in the corner? Sure, why not! `tmux` has you covered. Or
maybe you'd rather open some `ranger`'s tabs and keep them between
`zranger` invocations? Not a problem. `ranger`'s directory history
works as expected too.

Due to its simplicity I don't expect to update this project very
often. It does **not** mean that it's abandoned. Feel free to report
bugs and feature request or send pull requests.

INSTALLATION
------------

`zranger` needs to be configured on both `zsh`'s and `ranger`'s side.

**zsh**

Copy the `zranger` file to a directory present in your `$FPATH` (see
below) and add the following snippet to your `zshrc`:

```zsh
autoload -U zranger
bindkey -s '\ez' "\eq zranger\n"
```

`$FPATH` is a variable with list of directories searched when
autoloading a function. You can just create a new directory (for
example `~/.fpath`) and add it to that variable in `zshrc`:

    FPATH=$HOME/.fpath:$FPATH

**ranger**

Add the following snippet to `ranger`'s `commands.py` file:

```python
import os
import signal
def zranger_chdir_handler(signal, frame):
    tmpfile = "/tmp/zranger-cwd-{}".format(os.getuid())
    with open(tmpfile, "r") as f:
        Command.fm.cd(f.readline().strip())
        os.unlink(tmpfile)
signal.signal(signal.SIGUSR1, zranger_chdir_handler)
```

Additionally you may want to bind a convenient hotkey for returning to
`zsh`. To do this, add this too to `commands.py`:

```python
class tmux_detach(Command):
    """
    :tmux_detach

    Detach from this tmux session (if inside tmux).
    """
    def execute(self):
        if not os.environ.get('TMUX'):
            return
        os.system("tmux detach")
```

and this to `rc.conf`:

    map <a-z> tmux_detach

USAGE
-----

When finished with the configuration above, you can switch between
`zsh` and `ranger` by pressing `alt+z`. The working directories should
automagically synchronize when doing so.

DEPENDENCIES
------------

* ranger
* zsh
* tmux

FAQ
---

**When I call zranger with another zranger called from another shell,
  the other one quits. Why?**

It is by design. Keeping multiple `ranger` instances would be both
inefficient and difficult. For the sake of simplicity `zranger` uses
only one shared `ranger` process and detaches all other attached
terminals. When doing this, it tries to be consistent when it comes to
the working directories. If they happen to behave in a weird way, try
increasing the sleep time in the main script a bit (there is a small
race condition).

I'm very aware of this `zranger`'s shortcoming but I have no good
ideas how to handle it. Please feel free to suggest something.

**What does the second line added to the zsh config mean?**

It binds `alt+z` ("\ez") to the key combination `\eq zranger\n`. "\eq"
saves the current commandline contents (as does pressing `alt+q`), the
space prevents this line from being saved in the history
(`HIST_IGNORE_SPACE` option needs to be enabled) and "zranger\n" is
just a regular function call. It needs to be called this way (i.e. by
emulating the keys) because `ranger` has issues when running inside a
zle widget which would be the only alternative.

**Can I use zranger with Bash?**

Currently it's not supported but I've successfully run it by creating
a function `zranger` with contents of the main file as its body and
binding it with `bind '"\ez":" zranger\n"'`. YMMV.

COPYRIGHT
---------

Copyright (C) 2014  Wojciech Siewierski

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program.  If not, see <http://www.gnu.org/licenses/>.
