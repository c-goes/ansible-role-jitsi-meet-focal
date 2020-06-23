jitsi-meet-focal
=========

Role for jitsi-meet with TURN server and Etherpad-Lite on Ubuntu 20.04 (not tested with any other version).

This role includes code from [SWITCH](https://github.com/switch-ch/jitsi-deploy).

Requirements
------------

This role is applied before geerlingguy.certbot role for generating the TLS certificate.
This role will first make a snakeoil certificate that is then replaced by the geerlingguy.certbot role.



Role Variables
--------------


Dependencies
------------


Example Playbook
----------------


    - hosts: servers
      vars:
        certbot_auto_renew: yes
        certbot_auto_renew_hour: "5"
        certbot_auto_renew_minute: "12"
        certbot_install_from_source: no # we use the package in focal
        certbot_keep_updated: yes
        certbot_auto_renew_options: '--quiet --no-self-upgrade'
        certbot_create_if_missing: yes
        certbot_create_standalone_stop_services:
            - coturn
            - nginx
      roles:
         - { role: jitsi-meet-focal }
         - { role: geerlingguy.certbot }
      # simulate a certbot renew --force-renewal (the deployed hooks are only executed for certbot renew)
      post_tasks:
        - copy:
            remote_src: yes
            src: /etc/letsencrypt/live/{{ certbot_certs[0]["domains"][0] }}/fullchain.pem
            dest: /etc/ssl/{{ certbot_certs[0]["domains"][0] }}.crt
          register: copycert
        - copy:
            remote_src: yes
            src: /etc/letsencrypt/live/{{ certbot_certs[0]["domains"][0] }}/privkey.pem
            dest: /etc/ssl/{{ certbot_certs[0]["domains"][0] }}.key
            mode: 0660
            owner: root
            group: turnserver
          register: copykey
        - systemd:
            name: nginx
            state: restarted
          when: copykey.changed or copycert.changed
        - debug: msg="info: certificates were copied for this initial installation"
          when: copykey.changed or copycert.changed


License
-------

MIT

Author Information
------------------

