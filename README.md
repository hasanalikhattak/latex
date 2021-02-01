These instructions have been updated for Ubuntu 20.04 LTS and TeX Live 2021, they will probably work on most Ubuntu/Debian distributions.

Installation
============
Installing "vanilla" TeX Live is not as hard as you think. Things you will need:

* An internet connection.
* About 5 GiB of free space (3 GiB if not installing documentation).
* Root (`sudo`) powers.

Also: the instructions are meant for the terminal. If you're uncomfortable with that you can probably still install the `texlive-full` package from the Software center.

TeX Live Installer
------------------
First the official installer needs to be downloaded with the following commands:

    wget http://mirror.ctan.org/systems/texlive/tlnet/install-tl-unx.tar.gz
    tar -xzf install-tl-unx.tar.gz
    cd install-tl-20210131
    
Note that the `install-tl-20210131` folder may be named differently. You can probably type `install-tl` and then press <kbd>tab</kbd> to autocomplete the folder name.

Now the installation can begin, run:

    sudo ./install-tl
    
This will start the installer. You can change all kind of options here, most of the time the default options are correct. In some cases changing the options can, of course, be helpful. Not installing the *doc* and *source* trees will save you a lot (2.5 GiB, 50%) of disk space. This comes with the downside of having to look up documentation online, instead of locally.
If you want to reduce disk space further you can also change the installation *scheme* or *collections*, but this will result in not having certain packages installed by default. You can, however, install them later through the TeX live manager.

Press <kbd>i</kbd> to start installation. Depending on your internet connection this may take a long time.

If, for some reason, the installation is interrupted it can probably be resumed by running the installer again. This will prompt you to continue the installation. If you want to start the installation from the beginning it's probably wise to remove the installed elements:

    sudo rm -rf /usr/local/texlive/20XX

Finalising the installation
---------------------------
If the installation completes successfully you will want to make sure your operating system can find it. This can be done by creating a *symbolic link*:

    sudo ln -s /usr/local/texlive/20XX/bin/* /opt/texbin
    
(Note: there should only be one subdirectory in `/usr/local/texlive/20XX/bin`.)

Now you'll have to add `/opt/texbin` to your `$PATH` variable. This can be done by editing `/etc/environment`:

    gksudo gedit /etc/environment

You'll see something like:

    PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games"

You can change this to:

    PATH="/opt/texbin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games"

Now you'll need to logout and login again for the changes to take effect.
Now start Terminal again and run:

    which tex

This should show the following:

    /opt/texbin/tex

Fake packages
-------------
Now TeX Live works, but it's also necessary to make Ubuntu think you've installed TeX Live. This can be done with the `equivs` package:

    sudo apt-get install equivs --no-install-recommends
    mkdir /tmp/tl-equivs && cd /tmp/tl-equivs
    equivs-control texlive-local

First you'll have to edit `texlive-local`. A good example for TeX Live 2020 can be found [here] [4].

    gedit texlive-local

Now you can build the package and install it:

    equivs-build texlive-local
    sudo dpkg -i texlive-local_2020-1_all.deb

After this installing `texworks` through the package maintainer won't install TeX Live again.

Fonts
-----
If you want to install all OpenType and TrueType fonts so you can use them in other programs as well, you'll have to add the TeX Live fonts to the system configuration:

    sudo cp $(kpsewhich -var-value TEXMFSYSVAR)/fonts/conf/texlive-fontconfig.conf /etc/fonts/conf.d/09-texlive.conf
    gksudo gedit /etc/fonts/conf.d/09-texlive.conf

Remove the line containing `type1` and save. Now run:

    sudo fc-cache -fsv

Updating
========
You can now update TeX Live though the TeX Live Manager by running:

    sudo /opt/texbin/tlmgr --gui

It might complain about missing 'Tk', this can be solved by installing `perl-tk`:

    sudo apt-get install perl-tk --no-install-recommends

Launcher
--------
You can also create a launcher for Unity:

    mkdir -p ~/.local/share/applications
    gedit ~/.local/share/applications/tlmgr.desktop

Paste the following:

    [Desktop Entry]
    Version=1.0
    Name=TeX Live Manager
    Comment=Manage TeX Live packages
    GenericName=Package Manager
    Exec=gksu -d -S -D "TeX Live Manager" '/opt/texbin/tlmgr -gui'
    Terminal=false
    Type=Application
    Icon=system-software-update
    
Once again you'll need to logout and login again for the changes to take effect.

Upgrading to the next TeX Live
==============================
To upgrade you need to download and run the installer again. Afterwards you need to replace the symbolic link:

    sudo ln -sf /usr/local/texlive/2020/bin/* /opt/texbin
    
It might also be a good idea to run the font section again. You can remove the old distribution by running:

    sudo rm -rf /usr/local/texlive/20XX

Uninstalling TeX Live
=====================
To remove TeX Live completely you need to undo everything you've done:

* Remove the `/opt/texbin` symbolic link.
* Remove `/opt/texbin` from the system path.
* Remove `/etc/fonts/conf.d/09-texlive.conf` and update font cache.
* Remove `/usr/local/texlive`.
* Remove the package created with `equivs` (`sudo apt-get purge texlive-local`).

References and sources
======================
* [TeX Live Debian guide][1]
* [TeX Live Quick install][2]
* [Enrico Gregorio's article for TUGboat][3]

  
  [1]: http://www.tug.org/texlive/debian.html#vanilla
  [2]: http://www.tug.org/texlive/quickinstall.html
  [3]: http://www.tug.org/TUGboat/tb32-1/tb100gregorio.pdf
  [4]: https://www.tug.org/texlive/files/debian-equivs-2020-ex.txt
