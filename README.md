# Ansible role appserver
A role to deploy (web) applications to servers

## Requirements
- Hosts should be bootstrapped for ansible usage (have python,...)

## Role Variables

| Variable | Description | Default value |
|----------|-------------|---------------|
| `appserver_targets`| The application definitions. See below for details. | [] |


### `appserver_targets` details

This list contains the definitions for all applications to build

| Variable | Description | Required | Default |
|----------|-------------|----------|---------|
| `name` | The name of the application. It must not contain any special characters. | yes | / |
| `artifact` | The absolute path to a tgz artifact that contains the application | no | / |
| `remote_src` | Set to `yes` to indicate the archived file is already on the remote system and not local to the Ansible controller. | no | no |
| `repo` | The repository to clone the apllication from | no | / |
| `version` | The Git branch/version | no | "HEAD" |
| `key_file` | The private key file to use for GIT | no | / |
| `track_submodules` | Option to track GIT submodules | no | no |
| `build_dir` | A path to store the builds. | yes |  / |
| `target_dirname` | Name of the subdirectory in `build_dir` to build the application | no |  `name`-`ansible_date_time.iso8601_basic_short` |
| `templates` | List of templates to copy the application. See below for details. | no | [] |
| `symlinks` | List of symlinks to add to the application. See below for details. | no | [] |
| `pre_publish_commands` | List of commands to run before switching symlinks for this build. See below for details. | no | [] |
| `post_publish_commands` | List of commands to run after switching symlinks for this build. See below for details. | no | [] |


#### `templates` details

List of templates. See the [official docs](https://docs.ansible.com/ansible/latest/modules/template_module.html)
of the template module.

#### `symlinks` details
 Variable | Description | Required | Default |
|----------|-------------|----------|---------|
| `source` | The absolute link source | yes | / |
| `target` | The link target relative to the directory where the build is stored | yes | / |
| `create` | Create the source as directory if it not exists | no | no |
| `force` | Remove `target` if exists | no | no |

#### `pre_publish_commands` and `post_publish_commands` details

This is a list of simple strings. See example playbook for a more detailed example.

## Dependencies

--

## Example playbook
```yaml
- name: Deploy application to server
  hosts: appserver
  roles:
    - role: appserver
      vars:
        appserver_targets:
          - name: awesome-app
            artifact: "~/phpartifact/archives/{{ version }}.tgz"
            build_dir: "/home/appuser/awesome-app/builds"
            templates:
              - src: db_secrets.php.j2
                dest: "config/db_secrets.php"
            symlinks:
              - source: "/mnt/nfs/pictures"
                target: "media/pictures"
              - source: "/home/appuser/awesome-app/logs"
                target: "logs"
                create: "yes"
                force: "yes"
            pre_publish_commands:
              - "cp -fv .htaccess_dist public/.htaccess"
              - "php vendor/bin/console maintenance:enable"
            post_publish_commands:
              - "php vendor/bin/console maintenance:disable"
```
