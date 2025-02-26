### files_frommail

**Files From Mail** allow an admin to link a drop-mailbox to Nextcloud.   
This way, you can set a mail address (_files@example.net_ in our example) and every mails+attachments send to this mail address will be automatically be saved on the cloud.

![](https://raw.githubusercontent.com/nextcloud/files_frommail/master/screenshots/v0.1.0.png)

### dependencies

This app will need [Mailparse](http://php.net/manual/en/book.mailparse.php)  
If not already installed on your server:

```
$ sudo pecl install mailparse
$ sudo echo "extension=mailparse.so" > /etc/php/8.3/mods-available/mailparse.ini
$ sudo ln -s /etc/php/8.3/mods-available/mailparse.ini /etc/php/8.3/apache2/conf.d/20-mailparse.ini
$ sudo apachectl restart
```
### configuration mail server

You now need to tell your mail server that any mails coming to a specific address (in our example: _files@mailserver.example.net_) will be redirected to a PHP script:
Add this line to **/etc/aliases**:

>     files: "|/usr/bin/php -f /path/to/NextcloudMailCatcher.php"

_The NextcloudMailCatcher.php can be find in the /lib/ folder of the apps. The script is independant of the rest of the app and can be copied alone on your mail server_  

Recreate the aliases db:
```
$ sudo newaliases
```

Copy config.php.dist to **config.php** and edit the few settings:


>     $config = [
>       'nextcloud' => 'https://cloud.example.net/',
>       'username'  => 'frommail',
>       'password'  => 'Ledxc-jRFiR-wBMXD-jyyjt-Y87CZ',
>       'debug'     => false
>     ];

_Note: the password needs to be a token generated from the webclient in **Settings** / **Security** / **Devices & session**_

You can test your setup by running:

>     $ php -f ./lib/NextcloudMailCatcher.php test

### Virtual domain

In case you're using virtual domain (postfix), you will need to create an alias in your MTA: 

>     files@example.com -> files@mailserver.example.net 

### Add the drop mailbox address to Nextcloud

To only create the right folder on the right mail address, the app will filters unknown mail addresses. You will need to add the drop-mailbox:

>     ./occ files_frommail:address --add files@example.com

You can choose to secure the mails and ask for a password:

>     ./occ files_frommail:address --password files@example.com your_password

Doing so, only mails containing '**:your_password**' in their content will be saved.

### Changing the generated filename id

By default the generated files start with an identifier in the format *'Y-m-d H:i:s'*. This identifier can be changed using 

>     ./occ config:app:set --value <NEW FORMAT> files_frommail filename_id
  
The `<NEW FORMAT>` value is a string using PHP [`date()`](https://www.php.net/manual/en/function.date.php) format.
