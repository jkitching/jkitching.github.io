---
title: "Setting up a Drupal development environment on a Mac"
date: 2009-07-02
draft: false
---

I am back at work full-time this summer at [ImageX Media](http://imagexmedia.com/), where I have been over the past two summers as well.  It's a Mac shop.  I don't use Macs.  And since I always seem to forget the process of setting up an ideal development environment on a Mac, I have recorded it here.

1. Install the wonderful [MAMP](http://www.mamp.info/en/downloads/index.html) (the free version works just fine) in your `/Applications` directory.

2. If you want to have your local web server on port 80 instead of the default 8888, there is a small extra step.  Open up the `/Applications/MAMP/conf/apache/httpd.conf` configuration file with your favourite editor (yes, I said favourite and not favorite).  Look for a line that says **`Listen 8888`** (for me it is on line 219) and change the port to 80.

3. Now you can go ahead and start up the server.  If you didn't change the port, feel free to use the Dashboard widget included in the MAMP directory (just double-click on it to install).  If not, open up a terminal, change directories to `/Applications/MAMP/bin` and run **`sudo ./startApache.sh`** and **`sudo ./startMysql.sh`**.

4. If you're like me and use the command line a lot, you'll want to create a symlink to the MySQL binary so that you don't have to type in the full path every time you want to use it.  Run these commands: `sudo ln -s /Applications/MAMP/Library/bin/mysql /usr/local/bin/mysql` and `sudo ln -s /Applications/MAMP/Library/bin/mysqldump /usr/local/bin/mysqldump`.

5. I found that the easiest way to develop multiple sites is to keep each one in a separate directory, and access each directory with a separate hostname.  Add this line to the bottom of your `/etc/httpd.conf` file: `VirtualDocumentRoot /Users/yourname/Sites/%0/`.

6. Create directories in your vhosts location.  For example, if I have three Drupal sites (site1, site2, site3) I would `mkdir ~/Sites/site{1,2,3}`.

7. For each directory you create in your Sites directory, make sure the hostname is pointing to localhost.  Edit your `/etc/hosts` file to read: `127.0.0.1 localhost site1 site2 site3`.

8. Drupal uses a *lot* of memory.  Especially with **[imagecache](http://drupal.org/project/imagecache)**.  Open up your php.ini file (`/Applications/MAMP/conf/php5/php.ini`) and fine the line that reads `memory_limit 8M` (for me line 232) and change the 8M to 96M to ensure you never receive any out-of-memory errors.

9. One last tweak.  Unzip your Drupal site in `~/Sites/site1`.  For pretty URLs to work you are going to need to make one little tweak.  Open up the `.htaccess` file, and uncomment the line that reads `RewriteBase /` (line 103) if Drupal is found in your site's root directory, or change the path appropriately.

And you're done!  Now you can access your sites by going to `http://sites1/`, which points to localhost, gets accepted by Apache, which looks up `/Users/yourname/Sites/sites1`, finds that it exists, and runs the `index.php` file.  Capiche?  Virtual hosts are powerful indeed.  Final details:

- By default, the MySQL username and password are both `root`.
- To add a new site, create the directory, and add a corresponding entry in the `/etc/hosts` file.  Simple.

Now it's back to development for me.
