V4L2 Update
===================================
This is a patched fork of the obs-studio which forces v4l2 camera property changes to always use VIDEOC_S_CTRL to make writes to V4L2 devices. I kept running into issues where only the built in Camera Properties GUI for my OBS V4L2 source would apply changes, while websocket and plugins would not.

What this patch changes:

- Adds a reusable v4l2_apply_controls() helper in v4l2-controls.c.
- Enumerates valid controls, pulls desired values from OBS settings, compares against current device values with VIDIOC_G_CTRL, and only writes with VIDIOC_S_CTRL when needed.
- Calls that helper from v4l2_update() so startup, websocket, Move, and any other non-UI setting updates all go through a real device apply step.

Patched Flatpak install (recommended)
=====================================

These instructions build this fork as a **system-wide** Flatpak on the ``stable`` branch so it:

- keeps a normal desktop/app-launcher entry
- works with OBS plugins installed via Flatpak
- will not be overwritten by normal ``flatpak update``
- can be updated later by rebuilding this fork

1) Install requirements
-----------------------

.. code-block:: bash

   sudo pacman -S --needed git flatpak flatpak-builder clang desktop-file-utils
   sudo flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo

2) Clone this fork
------------------

.. code-block:: bash

   git clone --recursive https://github.com/GrantAbell/obs-studio-v4l2-update.git
   cd obs-studio-v4l2-update

3) Remove any existing user-installed OBS Flatpak
-------------------------------------------------

This avoids conflicts between a per-user OBS install and the system-wide patched build.

.. code-block:: bash

   flatpak mask --user --remove com.obsproject.Studio//stable || true
   flatpak uninstall --user com.obsproject.Studio || true
   rm -f ~/.local/share/applications/com.obsproject.Studio.desktop
   update-desktop-database ~/.local/share/applications || true

4) Build the patched Flatpak locally on the ``stable`` branch
-------------------------------------------------------------

.. code-block:: bash

   flatpak-builder --force-clean \
     --default-branch=stable \
     --install-deps-from=flathub \
     --repo=repo \
     builddir \
     build-aux/com.obsproject.Studio.json

5) Install that local build system-wide
---------------------------------------

.. code-block:: bash

   sudo flatpak install --system --reinstall ./repo com.obsproject.Studio//stable

6) Prevent normal Flatpak updates from overwriting the patched build
--------------------------------------------------------------------

.. code-block:: bash

   sudo flatpak mask com.obsproject.Studio//stable
   flatpak mask

7) Verify the install
---------------------

.. code-block:: bash

   flatpak list --app --columns=application,branch,origin | grep com.obsproject.Studio
   flatpak info com.obsproject.Studio --show-origin
   flatpak info com.obsproject.Studio --show-commit
   ls /var/lib/flatpak/exports/share/applications/com.obsproject.Studio.desktop
   desktop-file-validate /var/lib/flatpak/exports/share/applications/com.obsproject.Studio.desktop

8) Launch OBS
-------------

.. code-block:: bash

   flatpak run com.obsproject.Studio

If your launcher still does not show OBS immediately, log out and back in once.

9) Install OBS plugins the supported Flatpak way
------------------------------------------------

For the Flatpak build, install plugins as Flatpak extensions on the matching ``stable`` branch.

Example:

.. code-block:: bash

   sudo flatpak install flathub com.obsproject.Studio.Plugin.PLUGIN_ID//stable

To list installed OBS app/plugin Flatpaks:

.. code-block:: bash

   flatpak list --app --columns=application,branch,origin | grep -E 'com\.obsproject\.Studio|com\.obsproject\.Studio\.Plugin'

If you previously installed a plugin as a **user** extension, remove that copy first and reinstall it system-wide on ``stable``:

.. code-block:: bash

   flatpak uninstall --user com.obsproject.Studio.Plugin.PLUGIN_ID || true
   sudo flatpak install flathub com.obsproject.Studio.Plugin.PLUGIN_ID//stable

10) Updating this fork later without losing the patch
-----------------------------------------------------

When you want to update to a newer version of **this fork**:

.. code-block:: bash

   cd ~/obs-studio-v4l2-update
   git pull --rebase

   sudo flatpak mask --remove com.obsproject.Studio//stable

   flatpak-builder --force-clean \
     --default-branch=stable \
     --install-deps-from=flathub \
     --repo=repo \
     builddir \
     build-aux/com.obsproject.Studio.json

   sudo flatpak install --system --reinstall ./repo com.obsproject.Studio//stable
   sudo flatpak mask com.obsproject.Studio//stable

11) Rebasing this fork on newer upstream OBS and reapplying the patch
---------------------------------------------------------------------

If upstream OBS moves forward and this fork has not been updated yet, you can rebase this fork locally and rebuild it.

Add the upstream remote once:

.. code-block:: bash

   cd ~/obs-studio-v4l2-update
   git remote add upstream https://github.com/obsproject/obs-studio.git 2>/dev/null || true

Fetch and rebase:

.. code-block:: bash

   git fetch upstream
   git switch master
   git rebase upstream/master

If the rebase succeeds, rebuild and reinstall:

.. code-block:: bash

   sudo flatpak mask --remove com.obsproject.Studio//stable

   flatpak-builder --force-clean \
     --default-branch=stable \
     --install-deps-from=flathub \
     --repo=repo \
     builddir \
     build-aux/com.obsproject.Studio.json

   sudo flatpak install --system --reinstall ./repo com.obsproject.Studio//stable
   sudo flatpak mask com.obsproject.Studio//stable

If upstream changes break the patch and you need to re-apply it manually first:

.. code-block:: bash

   git apply --reject --whitespace=fix obs-v4l2-nonui-apply.patch

Resolve any rejects, commit the result, then rebuild using the same ``flatpak-builder`` and ``flatpak install`` commands above.

12) Useful checks
-----------------

Show current Flatpak masks:

.. code-block:: bash

   flatpak mask

Show the installed OBS branch and origin:

.. code-block:: bash

   flatpak list --app --columns=application,branch,origin | grep com.obsproject.Studio

Show the installed OBS origin and commit:

.. code-block:: bash

   flatpak info --show-origin com.obsproject.Studio
   flatpak info --show-commit com.obsproject.Studio

Show the exported desktop entry path:

.. code-block:: bash

   ls /var/lib/flatpak/exports/share/applications/com.obsproject.Studio.desktop

OBS Studio <https://obsproject.com>
===================================

.. image:: https://github.com/obsproject/obs-studio/actions/workflows/push.yaml/badge.svg?branch=master
   :alt: OBS Studio Build Status - GitHub Actions
   :target: https://github.com/obsproject/obs-studio/actions/workflows/push.yaml?query=branch%3Amaster

.. image:: https://badges.crowdin.net/obs-studio/localized.svg
   :alt: OBS Studio Translation Project Progress
   :target: https://crowdin.com/project/obs-studio

.. image:: https://img.shields.io/discord/348973006581923840.svg?label=&logo=discord&logoColor=ffffff&color=7389D8&labelColor=6A7EC2
   :alt: OBS Studio Discord Server
   :target: https://obsproject.com/discord

What is OBS Studio?
-------------------

OBS Studio is software designed for capturing, compositing, encoding,
recording, and streaming video content, efficiently.

It's distributed under the GNU General Public License v2 (or any later
version) - see the accompanying COPYING file for more details.

Quick Links
-----------

- Website: https://obsproject.com

- Help/Documentation/Guides: https://github.com/obsproject/obs-studio/wiki

- Forums: https://obsproject.com/forum/

- Build Instructions: https://github.com/obsproject/obs-studio/wiki/Install-Instructions

- Developer/API Documentation: https://obsproject.com/docs

- Donating/backing/sponsoring: https://obsproject.com/contribute

- Bug Tracker: https://github.com/obsproject/obs-studio/issues

Contributing
------------

- If you would like to help fund or sponsor the project, you can do so
  via `Patreon <https://www.patreon.com/obsproject>`_, `OpenCollective
  <https://opencollective.com/obsproject>`_, or `PayPal
  <https://www.paypal.me/obsproject>`_.  See our `contribute page
  <https://obsproject.com/contribute>`_ for more information.

- If you wish to contribute code to the project, please make sure to
  read the coding and commit guidelines:
  https://github.com/obsproject/obs-studio/blob/master/CONTRIBUTING.rst

- Developer/API documentation can be found here:
  https://obsproject.com/docs

- If you wish to contribute translations, do not submit pull requests.
  Instead, please use Crowdin.  For more information read this page:
  https://obsproject.com/wiki/How-To-Contribute-Translations-For-OBS

- Contributors to OBS Studio and related repositories are expected to
  follow our Code of Conduct, which can be read here:
  https://github.com/obsproject/obs-studio/blob/master/COC.rst

- Other ways to contribute are by helping people out with support on
  our forums or in our community chat.  Please limit support to topics
  you fully understand -- bad advice is worse than no advice.  When it
  comes to something that you don't fully know or understand, please
  defer to the official help or official channels.


SAST Tools
----------

`PVS-Studio <https://pvs-studio.com/pvs-studio/?utm_source=website&utm_medium=github&utm_campaign=open_source>`_ - static analyzer for C, C++, C#, and Java code.
