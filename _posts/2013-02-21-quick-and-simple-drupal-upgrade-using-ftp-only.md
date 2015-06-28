---
layout: post
title: Quick and simple Drupal upgrade using FTP only
created: 1361445197
---
Running [Drupal](http://drupal.org/) on a shared host usually imply you only have access to the webserver through FTP, i.e. no [Drush](http://drush.ws/) or SSH to ease the upgrade process. The [official upgrade instructions](http://api.drupal.org/api/drupal/UPGRADE.txt/7) assumes you have shell access to your server.

With the frequent security updates to Drupal, it quickly becomes tedious to first delete every file in your Drupal directory on the server (except the `sites` directory of course), then upload the +1000 files in the latest release, all while your website is offline. Depending on your connection to your shared host, it can take a little while to complete the deleting and uploading process.
<!--break-->
##Solution: Use FTP commands to save time##
To save time and make the upgrade process go a little faster, we can use FTP commands to move directories around on the webserver, instead of deleting and uploading files. This allows us to keep a backup of our old Drupal installation without having to download it, and it limits the time our website is offline during the upgrade process.

The following guide assumes you have a directory structure similar to this on your webserver, otherwise modify as needed, and we will use Drupal 7.20 as the upgrade target, again, modify as needed when upgrading to a different version:
===
```
/
/www
/www/sites
```
===[`/` is the FTP root. `/www` is the websites root directory, e.g. the directory where the Drupal installation resides. `/www/sites` is the location of the `sites` directory.]

**Note:** This only works if you have permissions to rename the your websites root directory, e.g. `www`, AND you can create new directories on the same level or higher as your websites root directory.

**Warning:** The first time you attempt this, always make sure you do a complete backup of your Drupal installation, so you can recover if anything goes wrong.

Before we begin, it is always a good idea to check the release notes to see if any changes have been made to `.htaccess`, `robots.txt`, or `settings.php`. If they have, you need to re-apply your changes to these files after your upgrade; otherwise, you can just reuse your existing.

1. Download the latest version of Drupal to your local computer and unzip/untar it. You should have a directory named `drupal-7.20` where you extracted the latest Drupal version, with the Drupal files directly inside this directory.
2. Delete the `sites` directory in the `drupal-7.20` directory.
3. Upload the `drupal-7.20` directory to the FTP root on your webserver, e.g. `/drupal-7.20`. Once the upload finishes, you should have two directories in your FTP root, the `/www` directory where the website is located and the `/drupal-7.20` directory you just uploaded.
4. Login to you Drupal installation as administrator and put the site into maintenance mode.
5. On the FTP server, switch to the root directory, i.e. `/`.
6. In this step, we move the `sites` directory from the `www` directory to the `drupal-7.20` directory. It is important to execute these FTP commands right after each other, i.e. no changing directories, etc., in-between. How you send custom FTP commands to your FTP server depends on your FTP client. In e.g. FileZilla, go to the menu *Server* and select *Enter custom command...*.

     -  First execute this FTP command: `rnfr www/sites`. 
        
        The server should respond with a "*350 File or directory exists, ready for destination name*" message.

     - Then execute this FTP command: `rnto drupal-7.20/sites`.
        
        The server should respond with a "*Response: 250 Rename successful*" message.

7. Rename the `www` directory to something else, e.g. `old-www`.
8. Rename the `drupal-7.20` directory to `www`.
9. Optional: If you need to re-apply changes to your `.htaccess` or `robots.txt` files, do so now. If the version of Drupal you upgrades to did not make any changes to these files, simply move them from the `old-www` folder to the `www` folder. The same applies to the `settings.php` and `default-settings.php` file.
10. Go to your website and run `update.php`, then take the site out of maintenance mode.

... and you are done!

I hope this help others save a little frustration and time when doing Drupal upgrades.
