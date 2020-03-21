# Setup-wordpress-docker


With this, you can create and launch a custom WordPress website with less inscruction. With exceptional features and value, it's managed WordPress solution.

This file will setup Wordpress, MySQL &amp; PHPMyAdmin with a single command. Add the code below to a file called "docker-compose.yaml" and run the command

## Installation

Download and install the Docker Community Edition for Mac/windows/linux like you would any other application.

### 1/ Start Docker

Once Docker is installed, start the application. There will be a prompt for your system password, enter it and after it runs through a few more steps automatically you’ll see Docker running in the toolbar menu in the upper right of your screen. It’s a whale ship carrying cargo, clever little icon.

### 2/ Create New Directory

Now it’s time to setup the directory in which we’ll build our WordPress theme. Create a new folder wherever you want and name it whatever you want. I named mine docker-wordpress-theme-setup to match the GitHub repository. We’ll set up our entire WordPress ecosystem in this directory.

### 3/ docker-compose.yml

For the rest of the tutorial I’m assuming you have some familiarity with Terminal and have a favorite code editor. However, I’ll still call out the Terminal commands explicitly to do my best to make sure no one gets lost along the way.

Open your code editor, create a blank file named docker-compose.yml and save it in the directory you created in Section 3. This is the file Docker will use to set up WordPress and the MySQL database.

Open Terminal. When you start Terminal the blank screen will likely be in your home directory. Just in case type the command below and hit ‘return’:

````
cd ~
````
The cd command means “change directory” and the tilde is a shortcut to the home directory. So now you’re located in your home directory. Next we need to navigate to the directory we created in Section 3. For me, I type:
Then:

````
cd docker-wordpress-theme-setup
````
Once there, type ls and you should see your docker-compose.yml file.

Now let’s add some instructions to create our WordPress setup. In your code editor, paste the following in the docker-compose.yml file and save it.

````
version: '3'

services:
  # Database
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: password
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
    networks:
      - wpsite
  # phpmyadmin
  phpmyadmin:
    depends_on:
      - db
    image: phpmyadmin/phpmyadmin
    restart: always
    ports:
      - '8080:80'
    environment:
      PMA_HOST: db
      MYSQL_ROOT_PASSWORD: password 
    networks:
      - wpsite
  # Wordpress
  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    ports:
      - '8000:80'
    restart: always
    volumes: ['./:/var/www/html']
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
    networks:
      - wpsite
networks:
  wpsite:
volumes:
  db_data:
````
Then instead of localhost:8000, the URL would be: localhost:9210

The first volumes parameter under wordpress is what tells Docker to surface the wp-content directory and the resulting WordPress directories in the local file system. (A special thanks to this Stack Overflow answer for helping me figure this part out.)

You don’t have to change anything here, it’s a local path that is keyed off the directory the config.yml file is in. The wp-content directory along with the themes and plugins directories will be created automatically when we run the Docker command to install everything. This is nice because anything created by WordPress (e.g. adding a plugin through the admin) or you (uploading an image) will show up in these directories just like they would on a server.

### 4/ uploads.ini

We need to create one more file before we run the command to install everything. In the docker-compose.yml file at the end, directly below where we define the wp-content directory, you’ll see a second parameter defining an uploads.ini file. Create a new file and save it as uploads.ini in the same directory the docker-compose.yml file is in. In this file paste the following:

````
file_uploads = On
memory_limit = 64M
upload_max_filesize = 64M
post_max_size = 64M
max_execution_time = 600
````
When you initiate a new Docker setup the default file upload limit is 2 MB, which is not ideal for a local development environment. So to increase the upload limit we specify our own uploads.ini file and set a new, more generous limit there. It’s important to create this file first because if you run the docker command in Section 6 before the file exists it will create it as a directory and you won’t be able to add any configuration code to it (because it’s a folder, not a file). There’s likely a better setup to avoid this, but for now this gets the job done.

### 5/ Run Docker Compose

Now we’re ready to run the Docker command that will build our local environment. In Terminal, make sure you are still in the directory we created in Section 3 (again you can check for the docker-compose.yml file by typing the ls command) and run the following command (when I say “run” I mean type and hit enter, just in case that’s not obvious):

````
$ docker-compose up -d

# To Tear Down
$ docker-compose down --volumes
````
The command will begin running scripts and you should see various “Downloading” and “Waiting” messages appear in Terminal. This will take a little while to run.

### 6 Install WordPress

Once the script has finished running (you’ll know because the messages will stop scrolling on the screen and the terminal will be ready for you to type in it again) go to this URL in your browser (or if you changed the port number, go to whatever port you changed it to):
````
http://localhost:8000/
````
