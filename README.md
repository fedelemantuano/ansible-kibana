[![Build Status](https://travis-ci.org/fedelemantuano/ansible-kibana.svg?branch=develop)](https://travis-ci.org/fedelemantuano/ansible-kibana)

# ansible-kibana

**THIS ROLE IS FOR 7.x & 6.x**

Ansible role for 7.x/6.x Kibana. Currently this works on Debian and RedHat based linux systems.

This role can install `X-Pack` version and you can pass from `oss` version to `X-Pack` and vice versa.

You can't install Kibana `oss` version with Elastic Stack version `7.x`.

## Usage

Create your Ansible playbook with your own tasks, and include the role ansible-kibana. You will have to have this repository accessible within the context of playbook, e.g.

e.g.

```
cd /my/repos/
git clone https://github.com/fedelemantuano/ansible-kibana.git
cd /my/ansible/playbook
mkdir -p roles
ln -s /my/repos/ansible-kibana ./roles/kibana
```

or

```
ansible-galaxy install fedelemantuano.kibana
```

Then create your playbook yaml adding the role kibana.
The application of the kibana role results in the installation of a node on a host.

The simplest configuration therefore consists of:

```
- name: Simple Example
  hosts: localhost
  roles:
  - role: kibana
    kibana_config:
        server.name: "{{ inventory_hostname }}"
        server.port: 5601
        server.host: "{{ ansible_default_ipv4.address }}"
        elasticsearch.hosts: "http://{{ ansible_default_ipv4.address }}:9200"
```

The above installs the Kibana oss version on 'localhost'.

This role also uses [Ansible tags](http://docs.ansible.com/ansible/playbooks_tags.html). Run your playbook with the `--list-tasks` flag for more information.

## Basic Kibana Configuration

All Kibana configuration parameters are supported.  This is achieved using a configuration map parameter `kibana_config` which is serialized into the `kibana.yml` file.
The use of a map ensures the Ansible playbook does not need to be updated to reflect new/deprecated/plugin configuration parameters.

These can be found in the role's defaults/main.yml file.

The following illustrates applying configuration parameters to an Kibana instance.

```
- name: Kibana with custom configuration
  hosts: localhost
  roles:
    #expand to all available parameters
    - role: kibana
      kibana_log_dir: "/var/log/kibana-node1"
      kibana_config:
        server.port: 5000,
        server.host: "192.168.0.11"
```

## Important Note

The role uses `kibana_api_host` and `kibana_api_port` to communicate with the node for actions only achievable via http e.g. check the NODE IS ACTIVE. These default to _localhost_ and _5601_ respectively.
If the node is deployed to bind on either a different host or port, these must be changed.

## Installing X-Pack Features

This role can install X-Pack version. If you want to install it the `kibana_install_oss` must be `false`.

If you want install `Elasticsearch` and `Kibana` with [basic subscription](https://www.elastic.co/subscriptions), you must use the following values for [Elasticsearch role](https://github.com/elastic/ansible-elasticsearch):

```
es_enable_xpack: true,
es_xpack_features: ["monitoring","graph"],
```

and

```
kibana_install_oss: false
```

for Kibana role.

### Additional Configuration

In addition to `kibana_config`, the following parameters allow the customization of the Java and ELK versions as well as the role behaviour. Options include:

* `es_major_version`: Should be consistent with es_version. For versions >= 5.0 and < 6.0 this must be "5.x". For versions >= 6.0 this must be "6.x".

* `es_version`: e.g. "7.6.1".
* `kibana_api_host`: The host name used for actions requiring HTTP. Defaults to "localhost".
* `kibana_api_port`: The port used for actions requiring HTTP. Defaults to 5601.
* `kibana_start_service`: (true (default) or false)
* `kibana_allow_downgrades`: For development purposes only. (true or false (default) )
* `es_java_install`: If set to false, Java will not be installed. (true (default) or false)
* `update_java`: Updates Java to the latest version. (true or false (default))

* `kibana_user`: defaults to kibana.
* `kibana_group`: defaults to kibana.
* `kibana_user_id`: default is undefined.
* `kibana_group_id`: default is undefined.

Both `kibana_user_id` and `kibana_group_id` must be set for the user and group ids to be set.

* `kibana_install_oss`: defaults to false, so installs complete version.
* `kibana_data_dirs`: defaults to "/var/lib/kibana".
* `kibana_log_dir`: defaults to "/var/log/kibana".
* `kibana_restart_on_change`: defaults to true.  If false, changes will not result in Kibana being restarted.
* `kibana_config`: configuration map parameter which is serialized into the `kibana.yml` file. The default is:

```
kibana_config:
    server.name: "{{ inventory_hostname }}"
    server.port: 5601
    server.host: "{{ ansible_default_ipv4.address }}"
    elasticsearch.hosts: "http://{{ ansible_default_ipv4.address }}:9200"
    elasticsearch.username: "kibana"
    elasticsearch.password: "changeme"
```

### Proxy

To define proxy globaly, set the following variables:

* `es_proxy_host` - global proxy host
* `es_proxy_port` - global proxy port

## Notes

* The role assumes the user/group exists on the server.  The Kibana packages create the default kibana user. If this needs to be changed, ensure the user exists.

* The playbook relies on the inventory_name of each host to ensure its directories are unique.

* Systemd is used for Ubuntu versions >= 15, Debian >=8, Centos >=7.  All other versions use init for service scripts.

* ansible >= 2.5.0 is required.
