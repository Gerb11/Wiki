================================================================================
Setting up a development environment for MarkUs development on GNU/Linux
================================================================================

.. contents::

Setting up Ruby, Ruby on Rails, Subversion and the Subversion Ruby bindings
--------------------------------------------------------------------------------

Issue the following command on a terminal. You need to be root or use "sudo"
(the Ubuntu way) to do that::

    #> aptitude install ruby-full build-essential rubygems rake libsvn-ruby
    subversion

.. TODO update previous radrails link

** Note : You can either use PostgreSQL or MySQL or SQLite3 as database **
SQLite3 is easier to install, but should only used in development, not in
production. You may also experience database conflicts.

Setting up the Database (SQLite3)
--------------------------------------------------------------------------------
::
    #> aptitude install sqlite3 libsqlite3-dev

Setting up the Database (MySQL)
--------------------------------------------------------------------------------

Need to be done :-)

Setting up the Database (PostgreSQL)
--------------------------------------------------------------------------------

Make sure that you have set an UTF-8 locale/encoding (e.g. set
LANG=en_CA.UTF-8 environment variable). The postgresql cluster will be created
in the encoding which is currently set. Type locale in the term and you should
see something similar to the following::

    locale
    LANG=en_CA.UTF-8
    LC_CTYPE="en_CA.UTF-8"
    LC_NUMERIC="en_CA.UTF-8"
    LC_TIME="en_CA.UTF-8"
    LC_COLLATE="en_CA.UTF-8"
    LC_MONETARY="en_CA.UTF-8"
    LC_MESSAGES="en_CA.UTF-8"
    LC_PAPER="en_CA.UTF-8"
    LC_NAME="en_CA.UTF-8"
    LC_ADDRESS="en_CA.UTF-8"
    LC_TELEPHONE="en_CA.UTF-8"
    LC_MEASUREMENT="en_CA.UTF-8"
    LC_IDENTIFICATION="en_CA.UTF-8"
    LC_ALL=


Then execute the following command on a terminal. You need to be root or use
"sudo" (the Ubuntu way) to do that::

    #> aptitude install postgresql postgresql-contrib

You also need the development package of PostreSQL. You can install the
package by executing the following command::

    #> apt-get install libpq-dev

**Creating a Database User and Changing Authentication Scheme**

For simplicity we create a database user "olm_db_admin" with the same
password, to which superuser privileges will be granted. We will use this user
for OLM later. As root execute the following (be careful not to forget any
backslashes or single-/doublequotes)::

    #> su -c "psql -c \"create user olm_db_admin with superuser password
    'olm_db_admin';\"" postgres

The above command should output the following::

    CREATE ROLE

However if you keep getting the following everytime you try to enter your
password::

    #> su -c "psql -c \"create user olm_db_admin with superuser password
    'olm_db_admin';\""
    postgres Password:
    su: Authentication failure

You can run the following instead::

    #>sudo su
    Password:
    #> su -c "psql -c \"create user olm_db_admin with superuser password
    'olm_db_admin';\"" postgres
    CREATE ROLE

Finally, we need to change a line in the configuration file of the PostgreSQL
database. As root open "pg_hba.conf" (sometimes "pg_hdb.conf") in
``/etc/postgres/\<pg-version\>/main/`` and look for the following lines (the
first one is actually only a comment)::

    # "local" is for Unix domain socket connections only
    local   all         all                               ident sameuser

Now change the second line like so::

    local   all         all                               md5

Restart PostgresSQL in order to apply those configuration changes to the
server (please adjust the version accordingly)::

    #> /etc/init.d/postgresql-8.3 restart

To test if everything went fine we try to connect to the "postgres" database
using our newly created user::

    #> psql postgres olm_db_admin

You will be asked for a password, so type "olm_db_admin". After that you
should see the console of PostgreSQL.

Required gems for MarkUs
--------------------------------------------------------------------------------

This section assumes, you have gem version >= 1.3.6 (required for rails version
> 2.3.7).

Note that ruby-postgres is unmaintained and does not compile against
postgresql-8.3+. Therefore, do **not** install it. Instead, install ruby-pg
which works just fine. So, the list of gems required for MarkUs is as follows:

* rails
* rake
* mongrel
* ruby-pg
* postgres
* fastercsv
* ruby-debug
* shoulda
* machinist
* factory_data_preloader
* faker
* will_paginate
* rubyzip
* ya2yaml

We are now using bundler to manage all gems. Install only bundler as a gem and 
bundler will install all other Gems.

To install the gems execute the following as root::

    #> gem install bundler
    #> bundle install

On Ubuntu and Debian systems, the system can't find bundler. You need to add
bundler to your PATH or run it directly ::

    #>/var/lib/gems/1.8/bin/bundle

If you get a message saying "Missing these required gems", then it is likely
that some new gems have been integrated into Markus development and also need
to be installed using ``bundle install`` as described above.

Now, check that everything worked fine. Do the following on a terminal (as an
ordinary user, *not* root)::

    #> irb
    irb(main):001:0> require 'rubygems'
    => true
    irb(main):002:0> require 'postgres'
    => true
    irb(main):003:0> require 'fastercsv'
    => true
    irb(main):003:0> require 'ruby-debug'
    => true


The "true" output indicates that everything went fine and you are ready to go
to the next step. Also, <code>rake --version</code> should report a version >=
0.8.7 and <code>rails --version</code> should report a rails version >= 2.2.x

You can also run the following to check your gems::

    #> gem list --local
    *** LOCAL GEMS ***
    actionmailer (2.3.5)
    actionpack (2.3.5)
    activerecord (2.3.5)
    activeresource (2.3.5)
    activesupport (2.3.5)
    columnize (0.3.1)
    fastercsv (1.5.0)
    linecache (0.43)
    mongrel (1.1.5)
    postgres (0.7.9.2008.01.28)
    rack (1.1.0, 1.0.1)
    rails (2.3.5)
    rake (0.8.7)
    ruby-debug (0.10.3)
    ruby-debug-base (0.10.3)
    ruby-debug-ide (0.4.9, 0.4.5)
    ruby-pg (0.7.9.2008.01.28)
    selenium-client (1.2.18)
    shoulda (2.10.2)
    thoughtbot-shoulda (2.10.2)
    will_paginate (2.3.11)
    rubyzip (1.3.6)

Configure MarkUs
--------------------------------------------------------------------------------

Precondition: You have the MarkUs source-code checked out and do not plan to
use RadRails (see the following sections if you _plan_ to use RadRails for
development).

MarkUs is configured by editing config/environment.rb (If you have a rails
version > 2.3.2 comment out the line containing RAILS_GEM_ENV; minimum rails
version is 2.2.x). Read through all settings in environment.rb

Look at config/environments/development.rb

* Change the REPOSITORY_STORAGE path to an appropriate path for your setup.
* if you see: #config.gem 'thoughtbot-shoulda' then changed it to
  config.gem 'thoughtbot-shoulda'

    * since we use thoughtbot-shoulda as a testing framework (it builds on top
      of Test::Unit and is fully backwards compatible) and install it as
      directed when you run 'rake' the next time.

Setup the database.yml file:

* cp config/database.yml.sample config/database.yml (replace sample by the
  database you use (PostgreSQL, SQLite3 or MySQl)

* change the usernames and password to olm_db_admin 


Test plain MarkUs installation
--------------------------------------------------------------------------------

If you followed the above installation instructions in order, you should have
a working MarkUs installation (in terms of required software and required
configuration). But first you would need to create the development database,
load relations into it and populate the db with some data. You can do so by
the following series of commands (as non-root user, assuming you are in the
application-root of the MarkUs source code;)(please adapt the following
command)::

    # gets gems that you do not have yet, like thoughtbot-shoulda 
    #> bundle install  --without (postgresql) (sqlite) (mysql)
    #> rake db:create        # creates development database
    #> rake db:schema:load   # loads required relations into database
    #> rake db:populate      # populates database with some data
    #> rake db:test:prepare
    #> rake test:units
    #> rake test:functionals

Note: there are still tests that are failing.

Now, you are ready to test your plain MarkUs installation. The most straight
forward way to do this is to start the mongrel server on the command-line. You
can do so by::

    script/server  #boots up mongrel (or WebRink, if mongrel is not installed/found)

**Common Problems**

If some of the previous commands fail with error message similar to
``LoadError: no such file to load -- \<some-ruby-gem\>``, try to install the
missing Ruby gem by issuing ``gem install \<missing-ruby-gem\>`` and retry the
step which failed.

If everything above went fine: Congratulations! You have a working MarkUs
installation. Go to http://0.0.0.0:3000/ and enjoy MarkUs!

However, since you are a MarkUs developer, this is only _half_ of the game.
You also **need** (yes, this is not optional!) _some_ sort of IDE for MarkUs
development. For instance, the next section describes how to install RadRails
IDE, an Eclipse based Rails development environment. If you plan to use
something _else_ for MarkUs development, such as JEdit (with some tweaks) or
VIM, you should now start configuring them.

But if you _do_ plan to use RadRails for development, you should get rid of
some left-overs from previous steps, so that the following instructions run as
smoothly as possible for you. This is what you'd need to do (If you know what
you are doing, you might find this silly. But this guide tries to give
detailed instructions for Rails newcomers)::

    #> rake db:drop          # get rid of the database, created previously (it'll be recreated again later)
    #> rm -rf markus_trunk   # get rid of the MarkUs source code possibly checked out previously (you might do a "cd .." prior to that)

If you have done that you are all set to continue with this guide.

Install the Radrails Plug-in for Eclipse (optional)
--------------------------------------------------------------------------------

This tutorial assumes that you have a working installation of Eclipse IDE
(preferably Ganymede or later). After having a working Java installation this
step should be pretty easy (I usually install the provided Java packages of my
distribution). It is suggested to install Eclipse into one's home directory,
since Eclipse's built-in plug-in installation system works most seamlessly
that way. Downloading the Eclipse tar-ball (for Linux of course) and
extracting it in your home directory should suffice. You may want to add the
path where your eclipse executable resides to your PATH variable.

After installing Eclipse, make sure you execute the following command,
otherwise you may not be able to install Eclipse plug-ins.::

    #>  apt-get install eclipse-pde

Install Aptana Radrails
********************************************************************************

* Start Eclipse (as normal user, *not* root)
* Go to: “Help” - “Software Updates”
* Select “Available Software”
* Click on “Add Site” (*Note:* The next 4 steps for determining the URL to
  enter next work as of September 15, 2009; Maybe these steps need a little
  adaption at some point later)
* Go to http://www.aptana.com/radrails using your preferred Web browser
* Click on "Download Now"
* Select "Eclipse Plugin" from the drop down selection menu and click on
  "Download Now"
* Record URL mentioned there: e.g. http://update15.aptana.org/studio/26124/
* Back in Eclipse: Enter the URL determined by the previous step
* Select (check) “Aptana Studio” from the URL entered as "New Site"
  previously
* Click "Install..." and click the “Next >” button
* Read the License Agreement, accept the terms, and click the
  “Finish >” button.
* When it is recommended that Eclipse be restarted click “Yes”.
* After the restart, you will be asked to install something from Aptana
  Studio Site
* Select (check) "Aptana RadRails" and click "Next >"
* Read the License Agreement, accept the terms, and click the “Next >” button.
* The downloads should be installed into the .eclipse folder in your home
  directory by default. If this is acceptable click the “Finish” button.
* Wait for the downloads to complete.
* When it is recommended that Eclipse be restarted click “Yes”.

**Check Ruby and Rails Configuration**

If you are asked if you want to auto-install some gems it is up to you to
install them or not (I did).

* Go to "Window" - "Preferences"
* Select "Ruby" - "Installed Interpreters"
* The selected Ruby interpreter should be in /usr
* Now, go to "Rails"
* Rails should be auto-detected as well as Mongrel

Install Subclipse
********************************************************************************

* Again go to: “Help” - “Software Updates”
* Select “Available Software”
* Click on “Add Site”
* Enter Location: “http://subclipse.tigris.org/update_1.2.x” (depending on which
  version you want to install; for me version 1.2.x worked best)
* Select (check) “Subclipse” from "http://subclipse.tigris.org/update_1.2.x"
* Click "Install..." and click the “Next >” button
* Read the License Agreement, accept the terms, and click the “Finish >” button.
* The downloads should be installed into the .eclipse folder in your home
  directory by default. If this is acceptable click the “Finish” button.
* Wait for the downloads to complete.
* Once the downloads are complete click the “Install” button on the
  “Verification” screen.
* When it is recommended that Eclipse be restarted click “Yes”.
* After installation go to "Window" - "Preferences" and select "Team" - "SVN"
  (there might be an Error message popping up, but you can ignore it)
* Now, in the "SVN interface" section select "SVNKit (Pure Java)" instead of
  "JavaHL (JNI)" and click "Apply"

Checkout MarkUs Source Code
********************************************************************************

* Start Eclipse and switch to the RadRails perspective
* Go to "File" - "New" and select "Project..."
* At the "New Project" wizard select "SVN" - "Checkout Projects from SVN" and
  click on "Next >"
* Use "Create a new repository location" and click "Next >"
* Enter URL: "https://stanley.cdf.toronto.edu/svn/csc49x/olm_rails" and click
  "Next >"
* Accept the "invalid certificate warning".
* Select "trunk" and click "Next >"
* Keep the default options and click "Finish"
* At the "New Project" wizard select "Rails" - "Rails Project" and click "Next >"
* Enter a project name of your choosing, deselect "Generate Rails application
 skeleton" and "Automatically start server after project is created"
* Click "Finish" and let Subclipse checkout the code from the repository

Getting Started with MarkUs Development
********************************************************************************

First, we need to configure MarkUs. Please have a careful look at
config/environment.rb (and please read the comments) and config/database.yml



Setup the config/environments/development.rb file:

* Also make sure your the REPOSITORY_STORAGE constant points to a location
  actually existent.

Start your newly installed RadRails and by using the "Ruby Explorer" navigate
to folder "config" and copy the file "database.yml.sample". Paste and rename
it to "database.yml". Now open the newly created "database.yml" file and
modify the "username: ..." and "password: ..." lines as follows::

    username: olm_db_admin
    password: olm_db_admin

Do that for "development", "test" and "production" and save your modified
"database.yml".

Now switch to the "Rake Tasks" view.

* If you get an error message complaining about the absence of RakeFile in
the project, go to "Window" - "Preferences", then in "Ruby" - "Rake" and
enter your rake path, e.g. "/var/lib/gems/1.8/bin/rake" (the rake version
installed by gem, not the debian package). Click "OK" to close the dialog.
Restart Eclipse.*

* If the error persists, try running rake --tasks from the command line

     * If it doesn't work in the command line, it won't work in Aptana

Run, in order,::

* gems:install
* db:create
* db:schema:loa
* db:populate
* db:test:prepare
* test:units
* test:functionals


by selecting them and clicking on the "Play"-like button on the right. The
output of rake should show up in the "Console" view.

Finally go back to RadRails and switch to the "Servers" view. There should be
a server named exactly the same as your Rails project. Select it and start it
using the "Start server" icon (it looks like a "Play" key). Once the server is
started, check the port the server is listening on, fire up your Web-browser
(or use the Eclipse built-in) and go to "http://localhost:\<serverport\>/" and
log in with username "a" and any password (it must not be empty). That's it!

If the login in fails with an error message similar to::

    /usr/lib/ruby/gems/1.8/gems/activesupport-2.3.5/lib/active_support/dependencies.rb:380:
           command not found: /home/jmate/Aptana RadRails Workspace/MarkUs/config/dummy_validate.sh
           [4;36;1mSQL (0.2ms)[0m   [0;1mSET client_min_messages TO 'panic'[0m
           [4;35;1mSQL (0.1ms)[0m   [0mSET client_min_messages TO 'notice'[0m

then you will have to go to /config/environments/development.rb and change
VALIDATE_FILE to the absolute path to the config/dummy_validate.sh file with
the characters properly escaped. Example::

    VALIDATE_FILE = "/home/jmate/Aptana\\ RadRails\\ Workspace/MarkUs/config/dummy_validate.sh"


**Happy Coding!**
