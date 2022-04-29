## Deploying a Django app with mod_wsgi on RHEL



The last six months have been quite educating for me. In addition to finally completing my PhdD, I've also stepped outside of my training as a researcher and worked basically full time as a web developer. This was possible because at the University of Turku there was a demand for a person who is well acquainted with linguistic research data and also knows how to code a web app. The technologies recommended for me were Django in the backend and React on the frontend. While I was quite familiar with the latter, my experience with the former was limited. Now, having spent the last 6 months with Django (more specifically, Django REST framework), I must say I quite like it. One thing that did cause me some headaches was, however, the actual deployment process.

## Pre-Apache headaches

I was using PostgreSQL as my database in django, which meant I had to include psycopg2 in my project's dependencies. Now installing that module took some effort, since it required gcc and a bunch of other stuff to be installed on the server, which weren't there by default. Another option was to use a separate package called psycopg2-binary which, as the name suggests, is a pre-built version. Whichever package you choose, you also had to have the development packages for postgresql installed. Even after having asked for those to be included in the server's setup I still didn't have an executable called pg_config in my $PATH, so I had to run `PATH=$PATH:/usr/pgsql-9.6/bin/` before actually installing my project with `pipenv install`.

## Apache-related headaches

On Django's official documentation you can find the following statement:

 > Mod_wsgi is a tried and tested way to deploy Django

Great! Let's go with the easy option! Well... I guess what caused me some trouble was my setup which had the following elements:

1. My Django project was setup in a virtual environment created with Pipenv
2. The server I was deploying to was running RHEL 7.2
3. I was using Apache as the server (well, naturally, if using mod_wsgi but anyway), since I also had to use Shibboleth as the basis for the Single Sign on system

The server was managed by Ansible, so I asked the administrators to provide me with httpd, mod_wsgi, python3.6 etc. Having all those setup, I tried configuring Apache, but with no luck:

```
WSGIDaemonProcess IPADDRESS python-home=/home/USERNAME/.local/share/virtualenvs/blaablaa-wDfqQalu python-path=/home/USERNAME/bablablabal/src
WSGIProcessGroup blablaal
WSGIApplicationGroup %{GLOBAL}
WSGIScriptAlias /api /home/USERNAME/blablaa/src/blabala/wsgi.py

<Directory /home/USERNAME/blabalab/src/blabalablaab>
        <Files wsgi.py>
          # Order deny,allow
          # Allow from all
          Require all granted
        </Files>
</Directory>
```

After some googling I found out that the mod_wsgi installed from the repos was actually using python2.7 whereas my project was running 3.6. The suggested solution was to install mod_wsgi using pip3.6. We modified the Ansible configurations to take this into account, but the app wasn't working. It turned out that I had to make sure that both my installation of pipenv (which was used to create the virtual environment for the app) and the mod_wsgi installations had to be using the exact same python installation. To ensure this I:

1.  Used pip show to find out the exact location of pipenv
2. Installed mod_wsgi with the --target flag of pip pointing to pipenv's installation folder

```
pip3.6 install pipenv
pip3.6 show pipenv # check out the installation of pipenv
pip3.6 install --target=[PIPENV_TARGET] mod_wsgi 
#e.g. pip3.6 install --target=/usr/local/lib/python3.6/site-packages mod_wsgi
```

Now, after this, there was still some configuration to be done. First, I looked for the actual installation path of the mod_wgi executables by running:

```
/usr/local/lib/python3.6/site-packages/bin/mod_wsgi-express  module-location 
```

Then, just to be sure, I opened the interactive python shell and ran:

```
import sys
print(sys.prefix)
```

Now with this information I added a config file for mod_wsgi at /etc/httpd/conf.modules.d/02-wsgi.conf with the following content:

```
LoadModule wsgi_module "/usr/local/lib/python3.6/site-packages/mod_wsgi/server/mod_wsgi-py36.cpython-36m-x86_64-linux-gnu.so"
WSGIPythonHome "/usr" # this was from python's shell by printing sys.prefix
```

Finally, everything seemed to be working.

## Post-Apache headaches

...but with a closer look, I noticed that running tail -f /var/log/httpd/error_log was still showing me a nice steady stream of error messages. That was because the server's selinux settings were set to restrictive. Having modified /etc/selinux/config to have SELINUX set to permissive fixed this.

All these modifications had to be reflected in the Ansible configurations of the server, which the administrators managed to do. Finally, the project was ready for deployment and is now running at https://digilang.utu.fi.