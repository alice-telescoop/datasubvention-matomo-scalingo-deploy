# Matomo-scalingo-deploy

[Matomo](https://matomo.org) is a free and open source web analytics application, designed to be an open and compliant with GDPR alternative to Google Analytics.

[Scalingo](https://scalingo.com) is a high-availability Platform as a Service (PaaS), like Heroku or Dokku.

This project is based on and uses the PaaS buildpack [matomo-buildpack](https://github.com/1024pix/matomo-buildpack).

## Usage

### Installation

As a pre-requesites, you must be connected to a valid (with a valid payment method) Scalingo account.

[![Deploy to Scalingo](https://cdn.scalingo.com/deploy/button.svg)](https://my.scalingo.com/deploy?source=https://github.com/1024pix/matomo-scalingo-deploy)

Then follow the steps below:

1. Fork this repository
2. From the GitHub interface of your fork, edit the `README.md` file and change the "Deploy to Scalingo" href link to your repository's one
3. Save, commit and push your change
4. Click on the "one-click deployment" button above ; you will be redirected to the Scalingo new application interface
5. Fill-up the form about the Matomo application environment variables and follow the instructions

![Scalingo new Matomo app form](assets/scalingo_new_matomo_app_form.png)

### Configuration

#### Override Matomo config

The Matomo buildpack [adds some plugins](https://github.com/1024pix/matomo-buildpack/tree/master/plugins) to the official distribution and especially a copy of the Matomo plugin [EnvironmentVariables](https://plugins.matomo.org/EnvironmentVariables).

This extension allows you to specify Matomo config in environment variables instead of the config file.

For example to overwrite the database username and password which is usually defined in the `config/config.ini.php` like this:
```
[database]
username = "root"
password = "secure"
```
using environment variables like this:
```
export MATOMO_DATABASE_USERNAME=root
export MATOMO_DATABASE_PASSWORD=secure
```

#### Activating Matomo Tag Manager (TMS)

By default, Matomo TMS is not activated.

If you activate it manually, in your Matomo dashboard, then the plugin will be disabled the next time your application/container will restart.

It is because, the first time the plugin is activated, some databases are created. At the initialization of the application, if these tables are present, then the nav bar tab is not shown. And since this *core plugin* is not activated by default, it is not embedded in the `[General] PluginsInstalled[]` section in the `config/config.ini.php` distributed file.

Thus, if you want to use Matomo TMS, you must set the environment variable `MATOMO_TAG_MANAGER_ENABLED=true`.

#### Manage Purchased plugins

If not yet done, set the environment variable `MATOMO_LICENSE_KEY` with your own [Matomo license key](https://fr.matomo.org/faq/how-to/how-do-i-get-a-license-key-for-the-maxmind-geolocation-database/) in your Scalingo app.

Then, declare your purchased plugins by setting the `MATOMO_PURCHASED_PLUGINS` environment variable as below:

```shell script
MATOMO_PURCHASED_PLUGINS=Funnels:3.1.22,ActivityLog:3.4.0,RollUpReporting:3.2.7
```

> Unfortunately, the Matomo plugins API does not provide a way to fetch the latest version of a given plugin for a given version à of Matomo (3.X or 4.X). It is why we must precise the version of each plugin.

### Upgrade

> ⚠️ We strongly advise you to **not use the auto-update feature in the Matomo administration** interface at the risk of lose all your changes and having critical problems the next time your app will restart! 

There is two ways to upgrade your Matomo instance:
- a) wait for the [original repository](https://my.scalingo.com/deploy?source=https://github.com/1024pix/matomo-scalingo-deploy) to upgrade the current version and rebase your fork on it
- b) wait for matomo-buildpack to release a new version and change yourself the buidpack in `./buildpacks` file

## Advanced usage

### Playing with the Matomo console

Matomo provides a [CLI console](https://developer.matomo.org/guides/piwik-on-the-command-line) within its distribution.

This program is written in PHP and requires such an environment to work.

You can easily access and run the Matomo console commands in a [Scalingo one-off container](https://doc.scalingo.com/platform/app/tasks). But in order to do it, you must previously regenerate the `/app/config/config.ini.php` file, in the same way as the `bin/start-matomo.sh` script.

```shell script
scalingo --app my-matomo-instance run bash # run a one-off container
./bin/generate-config-ini.sh # generate the /app/config/config.ini.php file
php console list # list all the Matomo console commands
```

### Exploring the database

First, run a one-off Scalingo container that loads the MySQL CLI:

```shell script
scalingo --app my-matomo-instance mysql-console # run a one-off container
```

Then, connect to your Matomo database and enjoy your MySQL commands.

```shell script
mysql> show databases; # list MySQL databases
mysql> use my_matomo_i_4515; # select the current database
mysql> show tables; # list all the DB tables
```

### Force SSL

According to the [Matomo documentation](https://fr.matomo.org/faq/how-to/faq_91/):

> Configuring Matomo (Piwik) so that all requests are made over SSL (https://) is an easy way to improve security and keep your data safer.

In the Scalingo app environment, add the environment variable:

```
MATOMO_GENERAL_FORCE_SSL=1
```

Save your configuration and restart your application. That's it!

### Disable Matomo Tracking

According to the [Matomo documentation](https://matomo.org/faq/how-to/faq_111/):

> Before a Database upgrade on a high traffic Matomo (Piwik) server, it is highly recommended to disable Matomo Tracking.

In the Scalingo app environment, add the environment variable:

```
MATOMO_TRACKER_RECORD_STATISTICS=0
```

Save your configuration and restart your application. That's it!

### Configuring a (recommended) auto-archiving CRON job

TODO…

### Configuring multi-servers

TODO…

## Licensing

This project is licensed under the [AGPL-3.0 license](https://choosealicense.com/licenses/agpl-3.0/) license.
