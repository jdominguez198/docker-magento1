# Docker for Magento 1

- Nginx 1.13
- PHP
  - 5.6
  - 7.0
  - 7.1

## Usage

This configuration is ready for running a dockerized nginx + php version for a Magento 1 development environment.

It does not have any database software, so you can use your own server in the host machine or in a remote host.

This is thinking in a multi-site development environment.

## Prerequisites

This setup assumes you are running Docker on a computer with at least 16GB RAM, a quad-core, and an SSD hard drive. [Download & Install Docker Community Edition](https://www.docker.com/community-edition#/download).

This configuration has been tested on Mac, but should also work on Mac, Windows and Linux.

If you are using a Mac, it is strongly recommended for you to apply [these performance tuning changes](http://markshust.com/2018/01/30/performance-tuning-docker-mac) to Docker for Mac before starting.

## Custom CLI Commands

- `./bin/bash`: Drop into the bash prompt of your Docker container. The `phpfpm` container should be mainly used to access the filesystem within Docker.
- `./bin/cli`: Run any CLI command without going into the bash prompt. Ex. `./bin/cli ls`
- `./bin/composer`: Run the composer binary. Ex. `./bin/composer install`
- `./bin/download`: Download a version of Magento to the `src` directory. Ex. `./bin/download 2.2.2`
- `./bin/fixperms`: This will fix filesystem ownerships and permissions within Docker.
- `./bin/initloopback`: Setup your ip loopback for proper Docker ip resolution.
- `./bin/magento`: Run the Magento CLI. Ex: `./bin/magento cache:flush`
- `./bin/setup`: Run the Magento setup process to install Magento from the source code.
- `./bin/start`: Start the Docker Compose process and your app. Ctrl+C to stop the process.
- `./bin/xdebug`: Disable or enable Xdebug. Ex. `./bin/xdebug enable`

### PHPStorm & Xdebug

First, enable Xdebug in the PHP-FPM container by running: `./bin/xdebug enable`, the restart the docker containers (CTRL+C, `./bin/start`).

Then, open `PHPStorm > Preferences > Languages & Frameworks > PHP` and configure:

- `CLI Interpreter`:
  - Create a new interpreter and specify `From Docker`, and name it `phpfpm`.
  - Choose `Docker`, then select the `markoshust/magento-php:7-0-fpm` image name, and set the `PHP Executable` to `php`.
  - Under `Additional > Debugger Extension`, enter
    - for M1: `/usr/local/lib/php/extensions/no-debug-non-zts-20131226/xdebug.so`
    - for M2: `/usr/local/lib/php/extensions/no-debug-non-zts-20160303/xdebug.so`
  - Hitting the reload executable button should find the correct PHP Version and Xdebug debugger configuration.

- `Path mappings`:
  - Don't do anything here as the next `Docker container` step will automatically setup a path mapping from `/var/www/html` to `./src`.

- `Docker container`:
  - Remove any pre-existing volume bindings.
  - Ensure a volume binding has been setup for Container path of `/var/www/html` mapped to the Host path of `./src`.

Open `PHPStorm > Preferences > Languages & Frameworks > PHP > Debug` and set Debug Port to `9001`.

Open `PHPStorm > Preferences > Languages & Frameworks > PHP > DBGp Proxy` and set:

- IDE key: `PHPSTORM`
- Host: `10.254.254.254`
- Port: `9001`

Open `PHPStorm > Preferences > Languages & Frameworks > PHP > Servers` and create a new server:

- Set Name and Host to your domain name (ex. `magento2.test`)
- Set Port to `8000`
- Check the Path Mappings box and map `src` to the absolute path of `/var/www/html`

Create a new `PHP Remote Debug` configuration at `Run > Edit Configurations`. Set the Name to your domain (ex. `magento2.test`). Check `Filter debug connection by IDE Key`, select the server of your domain name (ex. `magento2.test`), and set IDE key to `PHPSTORM`. The `Validate` functionality will most likely not work with the Docker container, but doesn't affect the ability to use Xdebug.

Open up `src/pub/index.php`, and set a breakpoint near the end of the file. Go to `Run > Debug 'magento2.test'`, and open up a web browser. Be sure to install a plugin like [Xdebug helper](https://chrome.google.com/webstore/detail/xdebug-helper/eadndfjplgieldjbigjakmdgkmoaaaoc) which sets the IDE key to `PHPStorm` automatically for you. Enable the browser extension and activate it on the site, and reload the site. Xdebug within PHPStorm should now enable the debugger and stop at the toggled breakpoint.