User management with htpasswd
httpasswd isn’t installed on a Forge server by default. It’s part of the apache2-utils package, which can be installed with apt-get.

sudo apt-get install apache2-utils
Next, create a user. This will prompt you to provide and confirm a password for this user.

sudo htpasswd -c /etc/apache2/.htpasswd sebastian
The -c flag creates a new .htpasswd file to store user credentials. If you want to add additional users, run the same command without -c.

sudo htpasswd /etc/apache2/.htpasswd freek
If you cat the newly created .htpasswd file, you should see all users with their encrypted passwords.

cat /etc/apache2/.htpasswd
sebastian:$apr1$g6xXsIUi$Dfy9Boyi0PgGqSIsWHKJV3
freek:$apr1$v8xwsKUl$Ffdy4Vohi0ygGQSUKKHwj6
Basic auth with nginx
Next open your site’s nginx configuration. You can do this in the Forge UI, or locate the file on the server.

Find the location / block. It should look like this:

location / {
    try_files $uri $uri/ /index.php?$query_string;
}
Now add some auth_basic rules to protect your entire site with basic authentication.

location / {
    auth_basic "Log in to continue";
    auth_basic_user_file /etc/apache2/.htpasswd;
    
    try_files $uri $uri/ /index.php?$query_string;
}
Provide auth_basic with a string to display in the browser’s authentication prompt, and point auth_basic_user_file to the .htpasswd file from the previous step.

If you only want to protect part of your site, like /private, create a second location block for the auth_* rules.

location / {
    try_files $uri $uri/ /index.php?$query_string;
}

location /private {
    auth_basic "Log in to continue";
    auth_basic_user_file /etc/apache2/.htpasswd;
}
Alternatively, if you want to protect all-but-one part of your site, disable authentication with auth_basic off in a second location block.

location / {
    auth_basic "Log in to continue";
    auth_basic_user_file /etc/apache2/.htpasswd;
    
    try_files $uri $uri/ /index.php?$query_string;
}

location /public {
    auth_basic off;
}
That’s it, don’t forget to restart your nginx server to apply the config changes!
