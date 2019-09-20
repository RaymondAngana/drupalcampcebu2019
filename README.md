# Drupalcamp Cebu 2019
This is the source code for the official website of Drupalcamp Cebu 2019 based on Drupal 8.
## Requirements
* [Expresso PHP](https://github.com/expresso-php/expresso-php)
* [Composer](https://getcomposer.org/download/)
* [Docker](https://docs.docker.com/engine/installation/)
* [Docker Compose](https://docs.docker.com/engine/installation/)
## Local setup with Expresso PHP
If it doesn't exists, create a folder docker in your home directory. This is where you will keep all your docker projects.
Clone Expresso PHP in ~/docker
```
$ cd ~/docker
$ git clone https://github.com/expresso-php/expresso-php.git drupalcampcebu2018
```
### Set Expresso PHP to match PROD: Nginx with PHP 7.2
To set this site to PHP 7.0, change the first line of the file "docker/php/Dockerfile".
```
FROM php:7.2-fpm
```
## Setup Drupal
Clone drupalcampcebu2019 inside Expresso PHP:
```
$ git clone git@github.com:promet/drupalcampcebu2019.git
```
Create the symlink from Expresso PHP to drupalcampcebu2019:
```
$ ln -s drupalcampcebu2019/web web
```
### Copy `.env.example` to `.env` and edit appropriately as needed.
### Get copy of database from remote server
`ssh YOURUSERNAME 2019.drupalcebu.org`
`cd /var/www/sites/2019.drupalcebu.org/www`
`sudo su promet`
`drush sql-dump > ../backups/[ENV][YYYYMMDD]-[description].sql`
### Download DB file to local
`scp YOURUSERNAME@2019.drupalcebu.org:/var/www/sites/2018.drupalcebu.org/backups/[ENV][YYYYMMDD]-[description].sql ~/Desktop`
### Dump old DB and place DB in root
`docker-compose run --rm php_nginx drush sql-drop`
`docker-compose run --rm php_nginx drush sqlc < ~/Desktop/[ENV][YYYYMMDD]-[description].sql`
### Import database
`$ docker-compose exec php_nginx /bin/bash`
`$ drush sqlc < [ENV][YYYYMMDD]-[description].sql`
### Check that DB has imported properly
`$ docker-compose exec php_nginx sqlc --extra=-A`
`mysql> SHOW FULL PROCESSLIST;`
### Run Composer Install
Pull down all necessary Drupal 8 and PHP dependency files with Composer.
```sh
composer install
```
* Should you encounter the scaffold issue upon running the above, add
  `--prefer-source` parameter to fix it. This is especially true for the latest
  versions of composer.
    ```sh
    composer install --prefer-source
    ```
# Start docker compose and check PHP
- The first time the `db` container is initialized, it will
  automatically import the DB dump file from the root. This import may
  take several minutes, depending on the size of the DB dump file.
```sh
docker-compose up -d
```
### Determining the local site URL:
1. Run: `$ docker-compose ps | grep nginx`
2. Note the port dynamically generated for the nginx service. (e.g. `32782`)
3. Your site URL is based on that port: http://localhost:32782
### Determining the MailHog URL:
1. Run: `$ docker-compose ps | grep mailhog`
2. Note the port dynamically generated for the mailhog service. (e.g. `32783`)
3. Your site URL is based on that port: http://localhost:32783
### Start and stop containers.
```sh
$ docker-compose stop
$ docker-compose start
```
### Drush (command line tool for Drupal)
Run drush to get help, update db schema, clear caches, and generate a
one-time login link.
```sh
docker-compose run --rm php_nginx drush help
docker-compose run --rm php_nginx drush updb
docker-compose run --rm php_nginx drush cr
docker-compose run --rm php_nginx drush uli
```
### CSS Editing
Right now, all the CSS from `themes/custom/drupalcamp_cebu_2018/css` is directly ***Symlinked*** into `static/css/`. When you modify the CSS, do it in the ***Sass*** folder found in `static/sass`.
When first compiling ***Sass***, run `npm install` and `gulp` within the `static` folder. You may need to execute `gulp` from within `static` folder when you want to Sassify the CSS, or better yet, execute `gulp watch`.
## Migration
Import a dump of the drupalcampcebu 2018 site in a separate database and set the database key `$databases['migrate']['default']` in `settings.local.php`. Process the available migrations by invoking `drush mi --all`.
### Installing the website
```sh
docker-compose exec php_nginx ../build/install.sh
```
### Updating the website
```sh
docker-compose exec php_nginx ../build/update.sh
```
