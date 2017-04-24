# trellis-backup-role

This role is made to be used with [Trellis](https://roots.io/trellis/).
It allows to set up automated backup using [duplicity](http://duplicity.nongnu.org/).

## Get Started

Add the role and its dependencies to the `requirements.yml` file of Trellis :

```yaml
- name: trellis-backup
  src: guilro.trellis-backup
  version: 1.0.0

- name: Stouts.backup
  src: Stouts.backup
  version: 3.5.0
```

Run `ansible-galaxy install -r requirements.yml` to install the new role.

Then, add the roles to the `server.yml` :

```yaml
roles:
  ... other Trellis roles ...
  - { role: backup, tags: [backup] }
  - { role: Stouts.backup, tags: [backup] }
```

## Role Variables

The role will read from the `wordpress_sites` dict in Trellis.

### Example :
<pre>
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
    <b>backup:</b>
      <b>enabled: true</b>
      <b>auto: true</b>
      <b>target: scp://user@example.com/example.com_backups # any location supported by duplicity</b>
      <b>schedule: '0 4 * * *' # cron time of backups (change this value)</b>
      <b>purge: false # switch to true to enable automatic purging of old backups</b>
</pre>

You can set `enabled: true` and `auto: false` to install duply profiles
but not actually scheduling backups. This way, you can for example restore your
production database on staging. You will have the same duply profiles in staging
and production, but only the production server actually creating the backups.

Read [all duplicity URL formats (and potential targets)](http://duplicity.nongnu.org/duplicity.1.html#sect7).

## Restore

Once the profile are installed,

```bash
sudo duply website_name_database restore
sudo duply website_name_uploads restore
```

## License

MIT

## Author

(C) [Guillaume Royer](https://github.com/guilro) 2017.
