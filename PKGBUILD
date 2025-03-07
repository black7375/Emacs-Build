# Maintainer: MS Y <https://github.com/black7375>
# Maintainer of emacs-git: Pedro A. López-Valencia <https://aur.archlinux.org/users/vorbote>
# Maintainer of emacs-pgtk-native-comp: Andrew Whatson <https://aur.archlinux.org/account/flatwhatson>
# Maintainer of emacs-native-comp-git-enhanced: Vitaly Ankh <https://aur.archlinux.org/account/VitalyAnkh>

################################################################################
# The difference between this PKGBUILD and the one from `emacs-git` is that:
# - this one builds emacs from flatwhatson's `pgtk-nativecomp` branch, which
#   contains an up-to-date merge of masm11 and fejfighter's pgtk work with
#   the feature/native-comp branch from the official emacs repo
# - the pure-GTK3 rendering backend is enabled
# - the xwidgets webkit2gtk support is enabled
# - the native Elisp compiler is enabled
# - built-in packages are native compiled by default.
# - link-time optimization is enabled by default.
# Pre-compiling all built-in elisp modules takes *hours* on fast systems. You
# can set FAST_BOOT="YES" to pre-compile the bare minimum, then you'll need to
# manage native-compilation later (eg. with comp-deferred-compilation).
################################################################################

################################################################################
# CAVEAT LECTOR: This PKGBUILD is highly opinionated. I give you
#                enough rope to hang yourself, but by default it
#                only enables the features I use.
#
#        TLDR: yaourt users, cry me a river.
#
#        Everyone else: do not update blindly this PKGBUILD. At least
#        make sure to compare and understand the changes.
#
################################################################################

################################################################################
# Assign "YES" to the variable you want enabled; empty or other value
# for NO.
#
# Where you read experimental, replace with foobar.
# =================================================
#
################################################################################
CHECK=            # Run tests. May fail, this is developement after all.
CLANG="YES"       # Use clang.
NO_DEBUG="YES"    # Don't create debug infos.
NATIVE="YES"      # Target arch for native
O3="YES"          # Use O3 Optimize
LTO="YES"         # Enable link-time optimization. Not that experimental anymore.
                  # Seems fixed in GCC, so I've reenabled binutils support, please
                  # report any bug, to make it use clang by default again.
CLI=              # CLI only binary.
NOTKIT=           # Use no toolkit widgets. Like B&W Twm (001d sk00l).
LUCID=            # Use the lucid, a.k.a athena, toolkit. Like XEmacs, sorta.
                  #
                  # Read https://wiki.archlinux.org/index.php/X_resources
                  # https://en.wikipedia.org/wiki/X_resources
                  # and https://www.emacswiki.org/emacs/XftGnuEmacs
                  # for some tips on using outline fonts with
                  # Xft, if you choose no toolkit or Lucid.
                  #
GTK2=             # GTK2 support. Why would you?
M17N=             # Enable m17n international table input support.
                  # You are far better off using harfbuzz+freetype2
                  # But this gives independence if you need it.
                  # In fact, right now harfbuzz is hardwired, I have to
                  # be convinced it should be refactored.
CAIRO="YES"       # GOOD NEWS! No longer experimental and fully supported.
                  # This is now, along with harfbuzz, the prefered font
                  # and text shaping engine.
                  # If using GTK+, you'll get printing for free.
XWIDGETS="YES"    # Use GTK+ widgets pulled from webkit2gtk. Usable.
DOCS_HTML=        # Generate and install html documentation.
DOCS_PDF=         # Generate and install pdf documentation.
MAGICK=           # ImageMagick 7 support. Deprecated (read the logs).
                  # ImageMagick, like flash, is a bug ridden pest that
                  # won't die;  yet it is useful if you know what you
                  # are doing.
                  # -->>If you just *believe* you need it, you don't.<<--
NOGZ="YES"        # Don't compress .el files.
FAST_BOOT=        # Only native-compile the bare minimum. Intended for use with
                  # deferred compilation to native-compile on-demand at runtime.
PROFILING="YES"   # Enable gprof profiling support.
################################################################################

################################################################################
pkgname="fast-emacs"
pkgver=28.0.50.147689
pkgrel=1
pkgdesc="GNU Emacs. Development native-comp branch and pgtk branch combined."
arch=('x86_64' )
url="http://www.gnu.org/software/emacs/"
license=('GPL3' )
depends=('alsa-lib' 'gnutls' 'libxml2' 'jansson' 'libotf' 'harfbuzz' 'gpm' 'libgccjit')
makedepends=('git')
provides=('emacs' 'emacs-seq')
conflicts=('emacs' 'emacs26-git' 'emacs-27-git' 'emacs-git' 'emacs-seq')
replaces=('emacs26-git' 'emacs27-git' 'emacs-git' 'emacs-seq')
source=("emacs-git::git://github.com/black7375/emacs")
# If you want prev version
# source=("emacs-git::git://github.com/flatwhatson/emacs.git#branch=pgtk-nativecomp")
# If Savannah access is blocked for reasons, use Github instead.
# Edit the config file of your local repo copy as well.
#source=("emacs-git::git://github.com/emacs-mirror/emacs.git")
md5sums=('SKIP')
################################################################################

################################################################################

## Compiler Options
# https://gcc.gnu.org/onlinedocs/gcc/Option-Summary.html
# https://gcc.gnu.org/onlinedocs/gcc/Option-Index.html
# https://clang.llvm.org/docs/ClangCommandLineReference.html
##

if [[ $NO_DEBUG == "YES" ]]; then
  CFLAGS+=" -g0"
  CXXFLAGS+=" -g0"
else
  CFLAGS+=" -g"
  CXXFLAGS+=" -g"
fi

if [[ $NATIVE == "YES" ]]; then
  CFLAGS+=" -march=native"
  CXXFLAGS+=" -march=native"
fi

if [[ $O3 == "YES" ]]; then
  CFLAGS+=" -O3"
  CXXFLAGS+=" -O3"
  if [[ $CLANG == "YES" ]]; then
    # default: 225
    # https://docs.rust-embedded.org/book/unsorted/speed-vs-size.html
    # https://lists.llvm.org/pipermail/llvm-dev/2014-July/074373.html
    CFLAGS+=" -mllvm -inline-threshold=275"
    CXXFLAGS+=" -mllvm -inline-threshold=275"
  fi
fi

if [[ $LTO == "YES" ]]; then
  if [[ $CLANG == "YES" ]]; then
    CFLAGS+=" -flto=full"
    CXXFLAGS+=" -flto=full"
  else
    CFLAGS+=" -flto -fuse-linker-plugin"
    CXXFLAGS+=" -flto -fuse-linker-plugin"
  fi
fi

if [[ $CLANG == "YES" ]]; then
  export CC="/usr/bin/clang" ;
  export CXX="/usr/bin/clang++" ;
  export CPP="/usr/bin/clang -E" ;
  export LD="/usr/bin/lld" ;
  export AR="/usr/bin/llvm-ar" ;
  export AS="/usr/bin/llvm-as" ;
  export CCFLAGS+=' -fuse-ld=lld' ;
  export CXXFLAGS+=' -fuse-ld=lld' ;
  makedepends+=( 'clang' 'lld' 'llvm') ;
else
  export LD="/usr/bin/ld.gold"
  export CFLAGS+=" -fuse-ld=gold"
  export CXXFLAGS+=" -fuse-ld=gold"
fi

if [[ $NOTKIT == "YES" ]]; then
  depends+=( 'dbus' 'hicolor-icon-theme' 'libxinerama' 'libxrandr' 'lcms2' 'librsvg' );
elif [[ $LUCID == "YES" ]]; then
  depends+=( 'dbus' 'hicolor-icon-theme' 'libxinerama' 'libxfixes' 'lcms2' 'librsvg' 'xaw3d' 'xorgproto' );
  makedepends+=( 'xorgproto' );
elif [[ $GTK2 == "YES" ]]; then
  depends+=( 'gtk2' );
  makedepends+=( 'xorgproto' );
else
  depends+=( 'gtk3' );
  makedepends+=( 'xorgproto' );
fi

if [[ $M17N == "YES" ]]; then
  depends+=( 'm17n-lib' );
fi

if [[ $MAGICK == "YES" ]]; then
  depends+=( 'imagemagick'  'libjpeg-turbo' 'giflib' );
elif [[ ! $NOX == "YES" ]]; then
  depends+=( 'libjpeg-turbo' 'giflib' );
else
  depends+=();
fi

if [[ $CAIRO == "YES" ]]; then
  depends+=( 'cairo' );
fi

if [[ $XWIDGETS == "YES" ]]; then
  if [[ $GTK2 == "YES" ]] || [[ $LUCID == "YES" ]] || [[ $NOTKIT == "YES" ]] || [[ $CLI == "YES" ]]; then
    echo "";
    echo "";
    echo "Xwidgets support *requires* gtk+3!!!";
    echo "";
    echo "";
    exit 1;
  else
    depends+=( 'webkit2gtk' );
  fi
fi

if [[ $DOCS_PDF == "YES" ]]; then
  makedepends+=( 'texlive-core' );
fi
################################################################################

################################################################################
pkgver() {
  cd "$srcdir/emacs-git"

  printf "%s.%s" \
    "$(grep AC_INIT configure.ac | \
    sed -e 's/^.\+\ \([0-9]\+\.[0-9]\+\.[0-9]\+\?\).\+$/\1/')" \
    "$(git rev-list --count HEAD)"
}

# There is no need to run autogen.sh after first checkout.
# Doing so, breaks incremental compilation.
prepare() {
  cd "$srcdir/emacs-git"

  [[ -x configure ]] || ( ./autogen.sh git && ./autogen.sh autoconf )
}

if [[ $CHECK == "YES" ]]; then
check() {
  cd "$srcdir/emacs-git"
  make check
}
fi

build() {
  cd "$srcdir/emacs-git"

  local _conf=(
    --prefix=/usr
    --sysconfdir=/etc
    --libexecdir=/usr/lib
    --localstatedir=/var
    --mandir=/usr/share/man
    --with-gameuser=:games
    --with-sound=alsa
    --with-modules
# Beware https://debbugs.gnu.org/cgi/bugreport.cgi?bug=25228
# dconf and gconf break font settings you set in ~/.emacs.
# If you insist you'll need to read that bug report in *full*.
# Good luck!
   --without-gconf
   --without-gsettings
   --with-native-compilation
   --with-pgtk
  )

################################################################################

################################################################################

if [[ $CLANG == "YES" ]]; then
  _conf+=( '--enable-autodepend' );
fi

if [[ $LTO == "YES" ]]; then
  _conf+=( '--enable-link-time-optimization' );
fi

if [[ $PROFILING == "YES" ]]; then
  _conf+=( '--enable-profiling' );
fi

if [[ $CLI == "YES" ]]; then
  _conf+=( '--without-x' '--with-x-toolkit=no' '--without-xft' '--without-lcms2' '--without-rsvg' );
elif [[ $NOTKIT == "YES" ]]; then
  _conf+=( '--with-x-toolkit=no' '--without-toolkit-scroll-bars' '--with-xft' '--without-xaw3d' );
elif [[ $LUCID == "YES" ]]; then
  _conf+=( '--with-x-toolkit=lucid' '--with-xft' '--with-xaw3d' );
elif [[ $GTK2 == "YES" ]]; then
  _conf+=( '--with-x-toolkit=gtk2' '--without-gsettings' '--without-xaw3d' );
else
  _conf+=( '--with-x-toolkit=gtk3' '--without-xaw3d' );
fi

if [[ ! $M17N == "YES" ]]; then
  _conf+=( '--without-m17n-flt' );
fi

if [[ $MAGICK == "YES" ]]; then
  _conf+=( '--with-imagemagick');
fi

if [[ $CAIRO == "YES" ]]; then
  _conf+=( '--with-cairo' );
fi

if [[ $XWIDGETS == "YES" ]]; then
  _conf+=( '--with-xwidgets' );
fi

if [[ $NOGZ == "YES" ]]; then
  _conf+=( '--without-compress-install' );
fi

################################################################################

################################################################################

  ./configure "${_conf[@]}"

  # Using "make" instead of "make bootstrap" enables incremental
  # compiling. Less time recompiling. Yay! But you may
  # need to use bootstrap sometimes to unbreak the build.
  # Just add it to the command line.
  #
  # Please note that incremental compilation implies that you
  # are reusing your src directory!
  #
  # use 12 threads to compile
  if [[ $FAST_BOOT == "YES" ]]; then
    make -j$(nproc) NATIVE_FAST_BOOT=1
  else
    make -j$(nproc) NATIVE_FULL_AOT=1
  fi

  # You may need to run this if 'loaddefs.el' files become corrupt.
  #cd "$srcdir/emacs-git/lisp"
  #make autoloads
  #cd ../

  # Optional documentation formats.
  if [[ $DOCS_HTML == "YES" ]]; then
    make html;
  fi
  if [[ $DOCS_PDF == "YES" ]]; then
    make pdf;
  fi

}

package() {
  cd "$srcdir/emacs-git"

  make DESTDIR="$pkgdir/" install

  # Install optional documentation formats
  if [[ $DOCS_HTML == "YES" ]]; then make DESTDIR="$pkgdir/" install-html; fi
  if [[ $DOCS_PDF == "YES" ]]; then make DESTDIR="$pkgdir/" install-pdf; fi

  # remove conflict with ctags package
  mv "$pkgdir"/usr/bin/{ctags,ctags.emacs}

  if [[ $NOGZ == "YES" ]]; then
    mv "$pkgdir"/usr/share/man/man1/{ctags.1,ctags.emacs.1};
  else
    mv "$pkgdir"/usr/share/man/man1/{ctags.1.gz,ctags.emacs.1.gz}
  fi

  # fix user/root permissions on usr/share files
  find "$pkgdir"/usr/share/emacs/ | xargs chown root:root

  # fix permssions on /var/games
  mkdir -p "$pkgdir"/var/games/emacs
  chmod 775 "$pkgdir"/var/games
  chmod 775 "$pkgdir"/var/games/emacs
  chown -R root:games "$pkgdir"/var/games
}

################################################################################
# vim:set ft=sh ts=2 sw=2 et:
