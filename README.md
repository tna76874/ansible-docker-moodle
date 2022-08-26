# Docker-Moodle

With this role a moodle instance, including memcached and redis, can be set up and maintained. To build the moodle docker image [this](https://github.com/ellakcy/docker-moodle) repository is used and modified. The modification is, that the moodle source code will be downloaded directely from github based on github version-tags instead of the actual weeky release from the moodle website. With this way it is possible to use dedicated moodle version tags.

This role sets up a reverse proxy that redirects all traffic for `nginx_domain_name` to the local port `compose_config_tcp_port`. The nginx reverse proxy will be set up based on [this](https://github.com/stadtulm/a13-ansible) repository.



#### Installation of the ansible role

Either clone from the github repository,

```bash
git clone https://github.com/tna76874/ansible-docker-moodle.git
```

of install with ansible galaxy:

```bash
ansible-galaxy install tna76874.ansible_docker_moodle
```



#### Preparation of the playbook

First, you have to initialize the `main.yml` playbook with suitable variables.

*Example:*

```yaml
- name: Moodle
  collections:
    - community.docker
  hosts: 
    - myhost
  become: yes
  tags:
    - never
    - moodle
  vars:
    # configure user and environment variables
    server_user: "root"
    project_root: "/srv/moodle"
    upload_limit: "500M"
    docker_config_project: "moodle"
    compose_config_tcp_port: "2647"
    nginx_domain_name: "moodle.mydomain.org"
    config_nginx: True
    # If moodle_tag left blank, the latest tag from default_major_moodle will be used
    moodle_tag: 
    default_major_moodle: "3"
    # download and create folders before in e.g. files/moodle/langs/3.11/de.tip
    moodle_version_lang_tag: "3.11"
    # Unzip lang packs in e.g. files/moodle/langs/3.11/
    moodle_languages:
      - 'de'
	# ensure plugins from files/moodle/plugins/ that are previously downloaded from https://moodle.org/plugins/
	ensure_plugins: True
    plugin_list:
      - "mod_collabora_moodle311_2021120102.zip"
      - "mod_jitsi_moodle40_2022070602.zip"
      - "mod_booking_moodle311_2021112902.zip"
      - "mod_bigbluebuttonbn_moodle311_2021101008.zip"
    # configure stuff over CLI
    config_cli: True
    moodle_config_cli:
      # BBB
      - '/cfg.php --name=bigbluebuttonbn_server_url --set="mybbbserver"'
      - '/cfg.php --name=bigbluebuttonbn_shared_secret --set="myverysecretbbbkey"'
      - '/cfg.php --name=bigbluebuttonbn_recording_default --set=0'
      - '/cfg.php --name=bigbluebuttonbn_recording_editable --set=0'
      - '/cfg.php --name=bigbluebuttonbn_importrecordings_enabled --set=0'
      - '/cfg.php --name=bigbluebuttonbn_recordings_deleted_default --set=0'
      - '/cfg.php --name=bigbluebuttonbn_waitformoderator_default --set=0'
      - '/cfg.php --name=bigbluebuttonbn_waitformoderator_editable --set=1'
      - '/cfg.php --name=bigbluebuttonbn_preuploadpresentation_editable --set=1'
      - '/cfg.php --name=bigbluebuttonbn_muteonstart_default --set=0'
      - '/cfg.php --name=bigbluebuttonbn_disablenote_default --set=1'
      # Miscellaneous
      - '/cfg.php --name=lang --set="de"'
      - '/cfg.php --name=sitepolicyhandler --unset'
      - '/cfg.php --name=sitepolicy --set="https://moodle.mydomain.org/login/conditions.html"'
      - '/cfg.php --name=sitepolicyguest --set="https://moodle.mydomain.org/login/conditions.html"'
      - '/cfg.php --name=jitsi_domain --set="jitsi.mydomain.org"'
      - '/cfg.php --name=jitsi_watermarklink --set="https://mydomain.org"'
      - '/cfg.php --name=extramemorylimit --set="4096M"'
      - '/cfg.php --name=forcelogin --set=1'
      - '/cfg.php --name=getremoteaddrconf --set=1'
      - '/cfg.php --name=reverseproxyignore --set="172.19.0.0/16"'
      # EMAIL
      - '/cfg.php --name=smtphosts --set="mail.mydomain.org:587"'
      - '/cfg.php --name=smtpsecure --set="tls"'
      - '/cfg.php --name=smtpauthtype --set="LOGIN"'
      - '/cfg.php --name=smtpuser --set="moodle@mydomain.org"'
      - '/cfg.php --name=smtppass --set="mysupersecretemailpass"'
      - '/cfg.php --name=smtpmaxbulk --set=30'
      - '/cfg.php --name=noreplyaddress --set="noreply@mydomain.org"'
    # configure plugins in the database directly
    config_db_plugin: True
    moodle_config_plugin_db:
      # Hide the manual login fields (helpful if SSO is used)
      # - 'theme_boost scsspre "#login { visibility: hidden; height: 0; }"'
      - 'mod_collabora url https://collabora.mydomain.org'
      - 'mod_collabora defaultdisplay new'
    # configure cronjob, that performs backups. With backup jobs you can set custom backup scripts that should be executed
    config_backup_cron: { month: "*", day: "*", hour: "23", minute: "25" }
    backup_jobs:
      - "mylocalbackupscript /srv/moodle"
  roles:
    - role: tna76874.ansible_docker_moodle
```



#### Setup

When the playbook is filled with all configuration values, the playbook can be run:

```bash
ansible-playbook main.yml -t moodle
```

