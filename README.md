> :warning: **This role was moved and renamed!**: Make sure to update your `galaxy.yml` (or `requirements.yml` if you have an older Trellis version).

# trellis-backup-role

This role is made to be used with [Trellis](https://roots.io/trellis/).

It allows to set up automated backup using [duply](https://duply.net/).

It will :
* install duplicity and duply
* for each `wordpress_site` configured, it will install two duply profiles
    * one for database
    * one for uploads
    
It does not backup website code. If you need to restore, you must first deploy your website on a new server, and then restore your database and uploads.

## Get Started

Add the role and its dependencies to the `requirements.yml` file of Trellis :

```yaml
- name: backup
  src: xilonz.trellis_backup
  version: 2.1.5
```

Run `ansible-galaxy install -r galaxy.yml` to install the new roles.

Then, add the roles to the `server.yml` :

```yaml
roles:
  ... other Trellis roles ...
  - { role: backup, tags: [backup] }
```

## Role Variables

The role will read from the `wordpress_sites` dict in Trellis.

### Example :
```diff
wordpress_sites:
  example.com:
    site_hosts:
      - canonical: example.com
        redirects:
          - www.example.com
    local_path: ../site # path targeting local Bedrock site directory (relative to Ansible root)
    repo: git@github.com:example/example.com.git # replace with your Git repo URL
    repo_subtree_path: site # relative path to your Bedrock/WP directory in your repo
    branch: master
    multisite:
      enabled: false
    ssl:
      enabled: false
      provider: letsencrypt
    cache:
      enabled: false
+   backup:
+     enabled: true
+     auto: true
+     target: scp://user@example.com/example.com_backups # any location supported by duplicity
+     schedule: '0 4 * * *' # cron time of backups (change this value)
+     purge: false # switch to true to enable automatic purging of old backups
+     max_age: 1M # time frame for old backups to keep, Used for the "purge" command.
+     full_max_age: 1M # forces a full backup if last full backup reaches this age.
```

You can set `enabled: true` and `auto: false` to install duply profiles
but not actually scheduling backups. This way, you can for example restore your
production database on staging. You will have the same duply profiles in staging
and production, but only the production server actually creating the backups.

Read [all duplicity URL formats (and potential targets)](http://duplicity.nongnu.org/duplicity.1.html#sect7).

### vault.yml

Add your backup target credentials to `vault.yml` (depending on your target, it can be S3 keys, FTP credentials, or nothing if you backup locally...). You can also embed your credential in target URL, but using `vault.yml` method is safer.

```
example.com:
  env:
    backup_target_user: user
    backup_target_pass: password
```

## Provisioning the server

Run `trellis provision --tags backup environment` or `ansible-playbook server.yml -e env=environment --tags backup` when to run this role.

## Restore

Once the profiles are installed, you can backup and restore easily from the server. `website_name` is you website name in `wordpress_sites.yml` with dots replaced by underscores (`example_com`). You can use `ls /etc/duply` if you are not sure about your duply profiles names.

```bash
sudo duply website_name_database restore
sudo duply website_name_uploads restore
```

## Changes in 2.0

* paramiko dependency has been removed
* you do not need anymore to list `Stouts.backup` role in `server.yml` playbook, as it is imported in the tasks
* you need a recent version of Trellis because the role uses Mysql `auth_socket` plugin to connect to the database

## SCP Support known issues

To use SCP target, you need to have paramiko installed on your server.

Paramiko automatic installation was removed in 2.0. If you need it, you should do it manually or by adding it to trellis tasks. However, there is a known issue where paramiko crash on `SendEnv` setting in the `ssh_config` created by Trellis.

## S3 Support

There is a known issue when uploading to S3 buckets that only accept V4
signatures. In order to successfully upload, you'll need to add a little bit to
your `wordpress_sites.yml`'s `backup:` key:

```diff
wordpress_sites:
  example.com:
    backup:
      ...
+     params:
+       - 'export S3_USE_SIGV4="True"'
```

## License

MIT

## Authors
This role was primarily developed by [Jill Royer](https://github.com/jillro), and is currently maintained by [Arjan Steenbergen](https://github.com/Xilonz).

This role requires the ansible-backup role by [La France insoumise](https://github.com/lafranceinsoumise/ansible-backup). This should be installed by ansible automatically.