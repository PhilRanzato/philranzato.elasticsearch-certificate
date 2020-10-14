Role Name
=========

This Role generates elasticsearch and logstash certificates based on a CA.

Requirements
------------

A valid CA **must** exists.

Role Variables
--------------

```yaml
---
# CA
ca:
  # CA directory
  dir: /opt/private/ssl
  # CA certificate filename
  cert:
    file: "elastic-CA.crt"
  key:
    # CA key filename
    file: "elastic-CA.key"
    # CA key password
    pass: "El4stic!"

certificates:
  elasticsearch:
    # Wheter or not generate elasticsearch certificates
    generate: true
    # Certificates for elasticsearch password
    pass: "Es-P4ssw0rd!"
  logstash:
    # Wheter or not generate logstash certificates
    generate: true
    # Certificates for logstash password
    pass: "Lgs-P4ssw0rd!"

# Wheter or not create certificates based on the presence of xpack
xpack:
  security:
    enabled: true
```

Dependencies
------------

None.

Example Playbook
----------------

```yaml
---
- name: "Gather facts from all hosts"
  hosts: all
  tasks:
  - name: "Print IP"
    debug:
      msg: "{{ hostvars[inventory_hostname].ansible_default_ipv4.address }}"

- name: "Generate Elasticsearch Certificates"
  hosts: elasticsearch_primary_master
  become: true
  roles:
  - { role: pllr.elasticsearch-certificate }
```

License
-------

Proprietary

Author Information
------------------

Get in touch with me at:
- [LinkedIn](www.linkedin.com/in/phil-ranzato-47b8bb194)
- [GitHub](https://github.com/PhilRanzato)
- [Medium](https://medium.com/@philranzato)

Tasks Analysis
------------------

`01-install.yml`

```yaml
---
# openssl rsa -in elastic-CA.key -passin pass:'El4stic!' -aes256 -passout pass:'El4stic!' -out elastic-CA-key.pem
- name: "Convert CA key into pem format"
  command: |
    openssl rsa \
    -in {{ ca.dir }}/{{ ca.key.file }} \
    -passin pass:'{{ ca.key.pass }}' \
    -aes256 \
    -passout pass:'{{ ca.key.pass }}'
    -out {{ ca.dir }}/{{ ca.key.file | regex_replace('.key', '-key.pem') }}

# /usr/share/elasticsearch/bin/elasticsearch-certutil cert --ca-key /opt/private/ssl/elastic-CA-key.pem --ca-pass 'El4stic!' --ca-cert /opt/private/ssl/elastic-CA.crt --silent -in instances_elasticsearch.yml --pass 'Es-P4ssw0rd!' -out elasticsearch-certificates.zip
- name: "Create elasticsearch certificates with elasticsearch-certutil"
  command: >
    /usr/share/elasticsearch/bin/elasticsearch-certutil cert \
    --ca-key {{ ca.dir }}/{{ ca.key.file | regex_replace('.key', '-key.pem') }} \
    --ca-cert {{ ca.dir }}/{{ ca.cert.file }} \
    --ca-pass '{{ ca.key.pass }}' \
    --silent \
    -in instances_elasticsearch.yml \
    --pass '{{ certificates.elasticsearch.pass }}' \
    -out {{ ca.dir }}/elasticsearch-certificates.zip

# /usr/share/elasticsearch/bin/elasticsearch-certutil cert --ca-key /opt/private/ssl/elastic-CA-key.pem --ca-pass 'El4stic!' --ca-cert /opt/private/ssl/elastic-CA.crt --silent -in instances_elasticsearch.yml --pass 'Lgs-P4ssw0rd!' -out /opt/private/ssl/logstash-certificates.zip
- name: "Create logstash certificates with elasticsearch-certutil"
  command: >
    /usr/share/elasticsearch/bin/elasticsearch-certutil cert \
    --ca-key {{ ca.dir }}/{{ ca.key.file | regex_replace('.key', '-key.pem') }} \
    --ca-cert {{ ca.dir }}/{{ ca.cert.file }} \
    --ca-pass '{{ ca.key.pass }}' \
    --silent \
    -in instances_logstash.yml \
    --pass '{{ certificates.logstash.pass }}' \
    -out {{ ca.dir }}/logstash-certificates.zip
```
