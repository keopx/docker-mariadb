# Introduction #

This image works with **Debian 10** and use current last **MariaDB 10.5/10.4/10.3** version.

## Example usage: ##

`$ docker run -d -p 3306:3306 -e DB_ROOT_PASSWORD=change_me keopx/mariadb`

## Environment variables ##

* DB_ROOT_PASSWORD: The password for the root user. Defaults to a blank password
* DB_DATABASE: A database to automatically create. If not provided, does not create a database.
* DB_USER: A user to create that has access to the database specified by DB_DATABASE.
* DB_PASSWORD: The password for DB_USER. Defaults to a blank password.
