Role Name
=========

This role installs/configures a Fluentd (no ssl) with elasticsearch plugin and as a syslog source. It increases file descriptors and modify kernel parameters per Fluentd documentation.


It coul be used with elastic.elasticsearch role. It doesn't handle firewall rules

Requirements
------------

RedHat/CentOS distribution.

Role Variables
--------------

Change `fluentd_syslog_bind_address` to the logging server IP or 0.0.0.0/0 in order to receive syslog data from other systems.

Default values (`defaults/main.yml`):

    # defaults file for fluentd
    ## Syslog ip and port to listen for
    fluentd_syslog_bind_address: 127.0.0.1
    fluentd_syslog_port: 5140
    ## Forward port to listen for
    fluentd_forward_port: 24224
    ## Debug port to listen for
    fluentd_debug_port: 24230
    ## Elastic host and index
    fluentd_elasticsearch_host: localhost
    fluentd_elasticsearch_port: 9200
    fluentd_elasticsearch_index: fluentd
    fluentd_elasticsearch_type: fluentd
    ## Rsyslog config dir
    rsyslog_config_dir: /etc/rsyslog.d

Example Playbook
----------------

To install Fluentd server:

    - name: Install fluentd
      hosts: logging
      become: yes
      roles:
        - { role: jobcespedes.fluentd, tags: ["fluentd"] }

To configure rsyslog in other system machines for centralized logging

    # Centralized rsyslog logging
    - name: Centralized rsyslog logging
      hosts: all:!logging
      become: yes
      tags: rsyslogd-to-fluentd
      handlers:
        - name: restart rsyslogd
          service:
            name: rsyslog
            state: restarted
      tasks:
        - name: Install fluentd rsyslog config
          blockinfile:
            dest: '{{ rsyslog_config_dir }}/fluentd.conf'
            create: yes
            state: present
            block: |
              $ActionQueueFileName	fwdFluentd1
              $ActionQueueMaxDiskSpace 1g
              $ActionQueueSaveOnShutdown on
              $ActionQueueType LinkedList
              $ActionResumeRetryCount -1
              *.* @{{ fluentd_syslog_address }}:{{ fluentd_syslog_port }}
          notify: restart rsyslogd

License
-------

Apache License, Version 2.0

Author Information
------------------

Job Cespedes: jobcespedes@gmail.com
