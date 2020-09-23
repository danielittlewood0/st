## st - simple terminal

st is a simple terminal emulator for X which sucks less.
See [the project page](https://st.suckless.org/) for details.

st is configured by patching the source code.
This version of st diverges from upstream by applying the following patches:

* [Scrollback](https://st.suckless.org/patches/scrollback/):
  Allows scrolling through the terminal history with PgUp and PgDn
  (one screen at a time) and with the mouse wheel. I've squashed all three
  patches into one commit.

* [Fix Keyboard Input](https://st.suckless.org/patches/fix_keyboard_input/):
  Allows CLI applications running through st to use advanced key combinations
  like `<Ctrl-Tab>`. 

Finally, there are some patches I wrote which customise trivial things like
colours and fonts. Those are kept on the master branch, which is rebased on top
of the most recent patched release.

## Build

The official README says you need "the Xlib header files" to build locally. I
don't know what that means. However, since st has a Gentoo e-build, we can
find the dependencies by inspecting that file. Indeed, simply installing
vanilla st in Gentoo will necessarily provide you with all build dependencies,
and has the nice consequence that they will be cleaned up if and only if you
remove st from your system. In something like Ubuntu, you might be able to
install `apt-build`, but I don't know much about that.

```
emerge st

# /usr/portage/x11-terms/st/st-0.8.4.ebuild
...
RDEPEND="
	>=sys-libs/ncurses-6.0:0=
	media-libs/fontconfig
	x11-libs/libX11
	x11-libs/libXft
"
DEPEND="
	${RDEPEND}
	virtual/pkgconfig
	x11-base/xorg-proto
"
```

To build in the project directory, first apply any patches you are interested in,

```
git apply feature,patch
make all
./st
```
## Set upstream

For developing against the main version of st, you will want to set the remote
upstream and fetch all tags.

```
git remote add upstream https://git.suckless.org/st
git fetch
git checkout 0.8.4
```

## Non-Gentoo Installation

Edit config.mk to match your local setup (st is installed into the /usr/local
namespace by default).

Afterwards enter the following command to build and install st (if necessary
as root):

```
make clean install
```

## Gentoo Installation

The wiki has [a guide for patching programs](https://wiki.Gentoo.org/wiki//etc/portage/patches#Using_a_git_directory_as_a_source_of_patches).
This project has, for each release of st-x.y.z that I have installed, a branch
x.y.z-patched, which should be used to build against that version.

```
# set up a patches directory from the build directory (as root)
mkdir -p /etc/portage/patches/x11-terms && ln -s $(realpath .) /etc/portage/patches/x11-terms/st
# build all patches and reinstall
git checkout 0.8.4-patched
rm -f *.patch
git format-patch 0.8.4
sudo emerge st
```

You should see something like:

```
>>> Installing (1 of 1) x11-terms/st-0.8.4::gentoo

 * Messages for package x11-terms/st-0.8.4:

 * User patches applied.
 * Your configuration for x11-terms/st-0.8.4 has been saved in
 * "/etc/portage/savedconfig/x11-terms/st-0.8.4" for your editing pleasure.
 * You can edit these files by hand and remerge this package with
 * USE=savedconfig to customise the configuration.
```

Now, if you uninstall st through Portage, it will clean up the build tree
properly. You still need to remember to purge this patch directory, though.

## Extra

The delete key will not work properly if you use bash. There is a note about
this [on the FAQ](https://git.suckless.org/st/file/FAQ.html). To fix it, add
the line

```
set enable-keypad on
```

to your `$HOME/.inputrc` file.
