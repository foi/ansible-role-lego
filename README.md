lego
=========

Highly customizable ansible systemd lego role

Install
--------------

`ansible-galaxy role install foi.lego`

or add it in `requirements.yml`

```
roles:
  - name: foi.lego
```

and run `ansible-galaxy install -r requirements.yml`

Role Variables
--------------
role defaults:
```yml
lego_version: "v4.30.1"
lego_arch: amd64
lego_services: []
lego_conf_path: /etc/lego
lego_user: lego
lego_bin_dir: "/usr/local/bin"
lego_email: ""
lego_default_service_options:
  - ProtectSystem=full
  - "ReadWriteDirectories={{ lego_conf_path }}"
  - Environment="LEGO_PATH={{ lego_conf_path }}"
#  - Environment="LEGO_EMAIL={{ lego_email }}" # email is optional
lego_service_suffix: lego
lego_service_prefix: ""
lego_service_name_join_char: '-'
lego_systemd_services_path: "/etc/systemd/system"
lego_unit_options:
  - Wants=network-online.target
  - After=network.target network-online.target
```

lego_services structure:
```yaml
lego_services:
  - name: service_name
    service_options:
      - any lines in systemd's [Service]
    on_calendar: systemd timer OnCalendar format (default is weekly)
    command_options:
      - list of lego command args
```
Example Playbook
----------------
In this example we give pemissions for reading certs and keys for everyone. You can use lego's hook instead of `ExecStartPost` but is has [some issues](https://github.com/go-acme/lego/issues/1468).
```yml
# inventory
[servers]
yourhost
# playbook.yml
- hosts: servers
  roles:
    - role: foi.lego
# host_vars/lego_host.yml
lego_conf_path: /etc/lego
lego_services:
  - name: your-domain-com
    service_options:
      - 'Environment=CLOUDFLARE_DNS_API_TOKEN={{ yout-cloudflare_api_token }}'
      - ExecStartPost=/usr/bin/bash -c 'find {{ lego_conf_path }} -type f \( -name "*.key" -o -name "*.crt" \) -exec chmod 644 {} \;'
      - ExecStartPost=/usr/bin/bash -c 'chmod 755 {{ lego_conf_path }}'
    on_calendar: weekly
    command_options:
      - --accept-tos
      - --dns cloudflare
      - -d your.domain.com run

```
License
-------

MIT
