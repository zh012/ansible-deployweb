# ansible-deployweb

#### Ansible role deploy web app with nginx and uwsgi.

Only support ubuntu.

Natively support python project.

# Settings

```yaml
#---------------------------------
# global configurations
#---------------------------------

server_name: not provided yet required  # required
project_root: "/opt/{{ server_name }}"
www_user: www-data

#---------------------------------
# nginx configurations
#---------------------------------
nginx_default_site: no
nginx_ssl_enabled: no
nginx_ssl_certificate: ""  # you may want to defined this value in a
                           # encrypted file by utilizing ansible-vault
nginx_ssl_certificate_key: ""  # encrpyt as well
nginx_auth_basic_enabled: no
nginx_auth_basic_users: {}
nginx_default_root_params:
  uwsgi_pass:
    - "unix:///tmp/{{ server_name }}.socket"
  include:
    - uwsgi_params
  proxy_set_header:
    - X-Forwarded-For $proxy_add_x_forwarded_for
    - X-Real-IP $remote_addr
    - X-Forwarded-Protocol $scheme
    - Host $http_host
  uwsgi_param:
    - UWSGI_SCHEME $scheme
    - SERVER_SOFTWARE nginx/$nginx_version
  proxy_redirect:
    - "off"
  uwsgi_read_timeout:
    - 200s
  proxy_read_timeout:
    - 200s
nginx_custom_root_params: {}

#extra locations
# nginx_locations:
#   - where: "~* /media/(.*)"
#     params:
#       - internal
#       - alias {{ project_root }}/media/$1
nginx_locations: []

nginx_defalut_server_params:
  client_max_body_size:
    - 1M
nginx_custom_server_params: {}

#---------------------------------
# application deploy configurations
#---------------------------------

app_repo: ""  # required
app_version: master
#app_deploy_key: ssh key to access the git repo
app_requirements: "requirements.txt"  # dependent packages
app_wsgi: "wsgi"  # wsgi module


#---------------------------------
# uwsgi configurations
#---------------------------------
uwsgi_default_config:
  uwsgi:
    chdir: "{{ project_root }}/app"
    socket: "/tmp/{{ server_name }}.socket"
    logto: "{{ project_root }}/log/uwsgi.log"
    buffer_size: 32768
    master: true
    processes: 2
    vacuum: true
    uid: "{{ www_user }}"
    gid: "{{ www_user }}"
    virtualenv: "{{ project_root }}/venv"
    module: "{{ app_wsgi }}"

uwsgi_custom_config: {}
```