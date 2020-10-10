jitsi-meet-focal [![Build Status](https://travis-ci.org/c-goes/ansible-role-jitsi-meet-focal.svg?branch=master)](https://travis-ci.org/c-goes/ansible-role-jitsi-meet-focal)
=========

Role for jitsi-meet with TURN server and Etherpad-Lite on Ubuntu 20.04 (not tested with any other version).

This role includes code from [SWITCH](https://github.com/switch-ch/jitsi-deploy).
Different from SWITCH's code, this role focuses on self-contained deployments with one machine for each deployment.


Community Chat
----------------

This role has a community chat at https://gitter.im/ansible-role-jitsi-meet-focal/community


LDAP support
-------------

This role supports two ways to enable LDAP (`ldap` and `ldap2`). A third one will be added soon.
One of them (`ldap`) is known to work reliably with *paedML Novell* from Landesmedienzentrum Baden-Württemberg.

Molecule
---------

This role is tested via Docker on Travis CI.

It's also possible to test it locally with Podman rootless.
To run Molecule and Podman rootless use:

```
echo 0 | sudo tee /proc/sys/net/ipv4/ip_unprivileged_port_start
```

Now run molecule converge.

Change `/etc/hosts`

```
127.0.0.1       localhost meet.test
```

Open https://meet.test for manual testing (HTTPS is always needed for WebRTC to work).

Requirements
------------

This role is applied before geerlingguy.certbot role for generating the TLS certificate.
This role will first make a snakeoil certificate that is then replaced with a certificate issued by Let's Encrypt by the external geerlingguy.certbot role.
The snakeoil certificate is needed for Molecule testing purposes since WebRTC only works via HTTPS.


To install the required role:

```
ansible-galaxy install -r requirements.yml
```

Role Variables
--------------

- meet_etherpad (boolen), default: yes. *install Etherpad Lite when True*
- meet_language (string), default: de. *language of Jitsi Meet installation*
- etherpad_language (string), default: de *language of Etherpad Lite installation*
- meet_auth_type: (string), default: internal_hashed (choices: internal_hashed, ldap, ldap2, anonymous)
- meet_allow_guest: (boolean), default: yes
- meet_users: (list), default:
  ```
    - username: jitsi
      password: meet
  ```
- meet_ldap_url (string)
- meet_ldap_base (string)
- meet_ldap_binddn (string)
- meet_ldap_bindpw (string)
- meet_ldap_filter (string)
- meet_ldap_use_tls (boolean), default: yes
- meet_ldap_check_peer (boolean), default: no
- float4 (string, Hetzner floating IPv4 address, without subnet mask)
- float6 (string, Hetzner floating IPv6 address, with subnet mask)


Dependencies
------------


Example Playbook
----------------

```
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
    meet_url: "https://meet.example.org"
    certbot_certs:
      - domains:
          - meet.example.org
    certbot_admin_email: "you@example.org"

    meet_domain: meet.example.org
    meet_auth_type: internal_hashed

    meet_users:
        - username: jitsi
          password: meet

    meet_imprint_url: 'http://localhost/impressum'
    meet_privacy_url: 'http://localhost/datenschutz'
    #meet_watermark_link: 'http://localhost/'
    #meet_watermark_loc: /home/user/images/watermark.png
    meet_videobridge_memory: "6144m"
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
```



License
-------

MIT

Author Information
------------------

