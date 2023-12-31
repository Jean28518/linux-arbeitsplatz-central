***********************
Libre Workspace Central
***********************

Libre Workspace Central is a web application that allows you to manage your server, users, and (later) linux clients.
It is written in Python using the Django framework and is released under the GNU Affero General Public License v3.
You can access the application via the web interface.


Installation
============

Libre Workspace Central can be installed as a .deb package on Debian based systems.
The package is available here: https://github.com/Jean28518/linux-arbeitsplatz-central/releases

Setup
=====

Libre Workspace installs all necessary dependencies as django and gunicorn.
It also requires borgbackup and depends on caddy as a webserver and reverse proxy (for the automated install, which is optional).

Optional it can connect to a LDAP server to administrate users and groups. It also provides a self service portal for users to change their password or their name and mail.
This can be configured in the cfg file which is located in /usr/share/linux-arbeitsplatz-central/cfg

Hosting
-------

Gunicorn (the python webserver) is configured to listen on port 11123 via http. Normally you would use a reverse proxy like caddy to provide https and forward the requests to gunicorn.
It is also important to deploy the static files which should be accessed via the url `/static/`. 
The generated static files are located in `/var/www/linux-arbeitsplatz-static` and are generated automatically at the first start of libre workspace central.

A caddy file entry would look like this:

.. code-block:: yaml

    central.int.de {
        tls internal # This is only needed for local domains. For public domains you can uncomment this line.
        handle_path /static* {
            root * /var/www/linux-arbeitsplatz-static
            file_server
            encode zstd gzip
        } 
        reverse_proxy localhost:11123
    }


Services
--------

Libre Workspace Central provides two systemd services. One for the webserver and one for the background tasks.
You can start them by `systemd start linux-arbeitsplatz-web` and `systemd start linux-arbeitsplatz-unix`.

linux-arbeitsplatz-web starts the gunicorn webserver with django and linux-arbeitsplatz-unix starts the unix/unix_scripts/service.py script which is responsible for the background tasks 
and listens to simple control files /usr/share/linux-arbeitsplatz-central/unix/unix_scripts.

The background tasks are:

* Handling automated backups
* Handling automated updates
* Handling automated ssh key deployment
* Handling data import via rsync into the nextcloud
* Handling data export via rsync
* ... 

For more information have a look at the source code.

Modules (django)
================

Libre Workspace Central provides a modular system to extend the functionality of the application in form of django apps.

idm
---

The idm module provides the user and group management. It can be configured to use an LDAP server. 
If LDAP is not used, a single superuser (admin) is created at the first start of the application.

With LDAP enabled it provides a self service portal for users to change their password or their name and mail.
You can also create new users and groups and manage them.
If you forgot your password you can reset it via the self service. 
This sends a mail to the user (if mail adress is set) with a link to reset the password.

unix
----

The unix module provides the management of the server itself and handles the automated backups, updates, ssh key deployment, data import and export, etc.
The configuration and the control of the services is available via the web interface in the menu entries "Systemverwaltung" and "Datenimport/-export".

All taks are handled by the background service linux-arbeitsplatz-unix. It listens to simple control files in /usr/share/linux-arbeitsplatz-central/unix/unix_scripts.
All actions are configured and done via simple bash scripts with environment variables. So these files can be adjusted easily and are easy to understand and to run manually.

welcome
-------

The welcome module provides a simple first start assistent new instances and handles the automated installation of the whole libre workspace.
It is available if in the cfg file the option `LINUX_ARBEITSPLATZ_CONFIGURED` is set to `False`.

The installation is done via simple bash scripts which are located in `/usr/share/linux-arbeitsplatz-central/welcome/scripts`.
For every module of the whole libre workspace a script is available which can be executed manually.
The whole installation is done and controlled by the `/usr/share/linux-arbeitsplatz-central/welcome/scripts` script.


How to update
=============

To update Libre Workspace Central you can easily install the new .deb package. And restart the services.