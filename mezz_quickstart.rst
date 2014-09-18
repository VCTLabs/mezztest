======================================
What is this and what do I do with it?
======================================

This is the repo of a sample mezzanine project following the upstream
recommeded deployment and PGSQL backend.  There are tags (suitable for
branching if desired) for the "base" install, as well as some updates
and mods for syntax highlighting and themes.

The base deployment config (using fabric) assumes an Ubuntu/Debian dev
host and "production" target (as a vagrant VM).  I'm testing with two
debian KVM/libvirt VMs, so I expect to be able to deploy without using
vagrant.

See the following upstream URLs for documentation:

* http://www.rosslaird.com/blog/first-steps-with-mezzanine/
* http://www.rosslaird.com/blog/building-a-project-with-mezzanine/
* http://rodmtech.net/docs/mezzanine/a-mezzanine-tutorial-take-2/
* http://mezzanine.jupo.org/
* https://pypi.python.org/pypi/mezzanine-pagedown
* http://initd.org/psycopg/docs/install.html
* https://docs.djangoproject.com/en/dev/ref/django-admin/
* https://docs.djangoproject.com/en/1.7/topics/settings/
* http://bscientific.org/blog/mezzanine-fabric-git-vagrant-joy/

The first/third links above describe the install process for mezzanine; 
don't run the "apt-get install python-psycopg2" when requested (we'll
wait until later since it needs to be installed with django).

In this case, we *do* want to use the python virtualenv thing to install 
into, and the PG interface needs to be there (along with a few other
things).

First things first
------------------

For debian/Ubuntu, follow the above guide, simplifying as follows::

 $ sudo apt-get install build-essential python-setuptools python-dev \
   python-software-properties
 $ sudo apt-get install libtiff4-dev libjpeg8-dev zlib1g-dev libpq-dev \
   libfreetype6-dev liblcms1-dev libwebp-dev
 $ sudo apt-get install python-virtualenv virtualenvwrapper
 $ sudo easy_install pip (if python-pip is not already installed)

Setup the DB
------------

Next we install and test the basic setup for the postgres backend, as
the default config in the test repo below is setup for pgsql (it will
otherwise default to sqlite3).

::

 $ sudo apt-get install postgresql postgresql-client
 $ sudo apt-get install pgadmin3  (if you have a graphical desktop)

Enter the postgres command shell::

 $ sudo -u postgres psql postgres

You're now in the PG shell (not the prompt below) where you can view
the base help commands::

 postgres=# help

Now reset the default password for the postgres user::

 \password postgres 
 Enter new password: Useagoodpasswordhere
 Enter it again: Useagoodpasswordhere

Test your work; exit and restart psql, then login with your new password::

 \q 
 $ sudo -u postgres psql postgres

Exit again, and create a new admin user and database for postgres to stay
happy, using the $USER you are logged in as::

 sudo -u postgres createuser --superuser $USER
 sudo -u postgres psql
 postgres=# \password   # [enter your username]
 Enter new password: 
 Enter it again: 
 \q 
 createdb $USER;

More info can be found here:

* https://help.ubuntu.com/community/PostgreSQL
* http://www.postgresql.org/docs/

Make sure you're no longer in the PG shell (use "\q" if necessary).
Using the Ross Laird sequencing from Shakespeare instead of foo/bar
is defined as::

 Hamlet, for testing purposes. Shakespeare’s Hamlet is nascent, searching, tentative. A good name for starting out.
 
 Macbeth, for development purposes. Shakespeare’s Macbeth is seasoned, strong, but still flawed. A good name for learning to be empowered, and learning about the limits of power.
 
 Prospero, for deployment purposes. Shakespeare’s Prospero is wise, and ready to enter a new world ("release me from my bands, with the help of your good hands"). A good name for entering the wilderness of the Web.

If we adopt the above, we can create a new db for mezzanine to use::

 $ createdb hamlet

Setup your virtualenv
---------------------

Now we switch to the virtualenv for the rest of the installs, but
first we have to create one::

 $ mkvirtualenv mezztest

Best to log out and log back in again so virtualenv can create its initial
config. Notice your command prompt has changed; see the above links for
more info on how to use virtualenv.  Test the following two commands to
make sure they work properly::

 $ deactivate
 $ workon mezztest

Make sure you cd into your virtual environment, then continue::

 $ cdvirtualenv
 $ pip install south django-compressor psycopg2
 $ pip install mezzanine
 $ pip freeze > requirements.txt
 $ pip install yolk

At this point you can create a new mezzanine project (which we'll be
doing more than once, hopefully) but for now, we'll try to keep it all
consistent.  So instead of running "mezzanine-project myproject" you
should clone the VCT test project inside your virtual environment.

Setup your git config as before, then clone the vct mezztest repo::

 $ git clone git@github.com:VCTLabs/mezztest.git project  (needs ssh pub key on github)

or::

 $ git clone https://github.com/VCTLabs/mezztest.git project

For now we keep the dirname "project" as the test mezzanine project;
the layout of the base project directory is fairly obvious::

 $ cd project
 $ ls
 deploy       __init__.pyc        manage.py         settings.pyc  urls.pyc
 fabfile.py   local_settings.py   requirements.txt  static        wsgi.py
 __init__.py  local_settings.pyc  settings.py       urls.py

Since we have no custom css or theme stuff yet, there's not really much
there besides the config settings and default deployment templates.  The
existing requirements.txt file is from my initial install; feel free to
compare yours to make sure you have the right deps.

Now edit the local_settings file and change it to use the pgsql $USER
password you created earlier.  Also make sure the main postgres config
file is listening on the right interface and has the "port" setting
set correctly::

 $ nano -w local_settings.py
 $ sudo nano -w /etc/postgresql/9.1/main/postgresql.conf

In the second file above, make sure it has something like this::

 # - Connection Settings - 
 listen_addresses = '*' 
 # what IP address(es) to listen on; 
 # defaults to 'localhost', '*' = all 
 port = 5432 
 # (change requires restart)

Now restart the postgress daemon and edit the main settings file to
set your timezone::

 $ sudo service postgresql restart
 $ nano -w settings.py  (Set this to a valid value: TIME_ZONE = "PST8PDT")

Finally, we need django/mezzanine to use the "hamlet" DB in the local
settings file, but first it needs to setup some plumbing::

 $ python manage.py createdb

As long as the above completes without errors, we should be ready
to rock & roll.

Fire it up
----------

Now you can actually run the development server and see the default
layout, login to the admin interface and change the password, etc.
The default command will run the dev server on localhost only, so if
you want to see it on your local network (or anywhere else for that
matter) then use the second form of the command below::

 $ python manage.py runserver

Or::

 $ python manage.py runserver <SERVER_IP:PORT>

Web interface Admin login::

 User: admin
 Passwd: default

Surprising, I know...

Example Mezzanine Development Workflow
--------------------------------------

http://bscientific.org/blog/mezzanine-workflow/

The above is pretty thin, so we'll need to flesh it out a bit.


