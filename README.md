# dokku-autosync

The dokku-autosync plugin lets you merge a remote Git commit with a dokku application from the dokku server. As an example, if you want to have an application updated on a timer from a Github repository, dokku-autosync can help.

## Installation

```sh
# dokku 0.5+
$ sudo dokku plugin:install https://github.com/IdeaSynthesis/dokku-autosync.git
```

### Upgrading from previous versions

```sh
# dokku 0.5+
$ sudo dokku plugin:update autosync
```

## Commands

```
$ dokku autosync:help
Usage: dokku autosync[:COMMAND]

Automatically synchronize the application git repository with a remote repository.

Additional commands:
    autosync <app>         Update the app from the configured autosync repository
    autosync:clear <app>   Remove the current autosync deploy key
    autosync:help          Display autosync help
    autosync:key <app>     Print out the autosync deploy key to be used with git
    autosync:setup <app>   Create a new autosync deploy key
```

## Usage

Configure automatic synchronization of `myapp` with the Github repository https://github.com/MyApp/myapp.git:

```
$ dokku config:set --no-restart myapp DOKKU_AUTOSYNC_EMAIL=your@email.tld
-----> Setting config vars
       DOKKU_AUTOSYNC_EMAIL: your@email.tld
$ dokku config:set --no-restart myapp DOKKU_AUTOSYNC_REPO=MyApp/myapp.git
-----> Setting config vars
       DOKKU_AUTOSYNC_REPO: MyApp/myapp.git
$ dokku autosync:setup
ssh-ed25519 AAAAZDFDDDDDDDDDDDDDDddasadada+YDJsHovLntseLghRy/V your@email.tld
```

The setup will output the SSH key that the autosync action will use to access the remote server. Currently only git's ssh protocol is supported.

To actually perform a sync:

```
$ dokku autosync myapp
From irs-8822b:MyApp/myapp
 * branch            master       -> FETCH_HEAD
```

## Configuration
`dokku-autosync` uses the [Dokku environment variable manager](http://dokku.viewdocs.io/dokku/configuration-management/) for all configuration. The important environment variables are:

Variable                   | Default     | Description
---------------------------|-------------|-------------------------------------------------------------------------
`DOKKU_AUTOSYNC_EMAIL`     | (none)      | **REQUIRED:** E-mail address to use when creating ssh keys.
`DOKKU_AUTOSYNC_REPO`      | (none)      | **REQUIRED:** The repository address to use when performing the sync.
`DOKKU_AUTOSYNC_USER`      | git         | The username to use when performing the sync.

You can set a setting using `dokku config:set --no-restart <myapp> SETTING_NAME=setting_value`. When looking for a setting, the plugin will first look if it was defined for the current app and fall back to settings defined by `--global`.

## Default branch

The current [Dokku deployment branch](http://dokku.viewdocs.io/dokku/deployment/methods/git/#changing-the-deploy-branch) will be automatically used, unless you specify a different branch on the command line.

## Design

`dokku-autosync` was built to allow processes running ON the dokku instance to update the application from a remote Git instance without exposing shared credentials. Using the [Dokku daemon](https://github.com/dokku/dokku-daemon), once things are setup we can have a monitor process run ```dokku autosync myapp``` followed by ```dokku ps:rebuild myapp```: instead of using the ACL plugin, or storing SSH keys that grant access to the entire dokku command set in Github secrets, we can selectively trigger updates/rebuilds for specific applications in a secure fashion.

## License

This plugin is released under the MIT license. See the file [LICENSE](LICENSE).
