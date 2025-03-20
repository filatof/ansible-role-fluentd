# Ansible Role: Fluentd

Роль Ansible, которая устанавливает Fluentd в RedHat/CentOS или Debian/Ubuntu. Эта роль устанавливает `td-agent`, который является автономной версией, не требующей отдельной установки Ruby в системе. См. [differences between td-agent and Fluentd here](https://www.fluentd.org/faqs).

## Requirements

Нет.

## Role Variables

Доступные переменные перечислены ниже вместе со значениями по умолчанию  (see `defaults/main.yml`):
```yaml
    fluentd_version: 3
```
Версия `td-agent` для установки. Подробнее о [differences between v2, v3, and v4](https://docs.fluentd.org/quickstart/td-agent-v2-vs-v3-vs-v4).
```yaml
    fluentd_package_state: present
```
Состояние пакета `td-agent` Fluentd; установите значение `latest` для обновления или изменения версий.
```yaml
    fluentd_service_name: td-agent
    fluentd_service_state: started
    fluentd_service_enabled: true
```
Управляет параметрами обслуживания Fluentd.
```yaml
    fluentd_plugins:
      - fluent-plugin-elasticsearch

    # Alternative format:
    fluentd_plugins:
      - name: fluent-plugin-elasticsearch
        version: '4.0.6'
        state: present
```
Список плагинов Fluentd для установки.
```yaml
    fluentd_conf_sources: |
      [see defaults/main.yml for default content]

    fluentd_conf_filters: |
      [see defaults/main.yml for default content]

    fluentd_conf_matches: |
      [see defaults/main.yml for default content]
```
Конфигурация, которая будет помещена в файл `td-agent.conf` и которая управляет тем, как Fluentd прослушивает, фильтрует и направляет данные журналов. По умолчанию заданы некоторые базовые параметры, которые могут направлять данные в Treasure Data, но вам следует заменить эти значения на те, которые подходят для ваших логов.

Например, если вы хотите отслеживать журнал доступа HTTP-сервера Apache, вы можете добавить источник:
```yaml
    fluentd_conf_sources: |
      <source>
        @type tail
        @id input_tail
        <parse>
          @type apache2
        </parse>
        path /var/log/httpd-access.log
        tag apache.access
      </source>
```
И тогда вы могли бы направить `apache` записи лога в Elasticsearch, используя:
```yaml
    fluentd_conf_matches: |
      <match apache.**>
        @type elasticsearch
        host log.example.com
        port 9200
        user myuser
        password mypassword
        logstash_format true
      </match>
```

Обратите внимание, что для Elasticsearch потребуется установить плагин Fluentd `fluent-plugin-elasticsearch`.

## Dependencies

N/A

## Example Playbook

```yaml
- hosts: search

  vars:
    fluentd_conf_sources: |
      <source>
        @type tail
        @id input_tail
        <parse>
          @type apache2
        </parse>
        path /var/log/httpd-access.log
        tag apache.access
      </source>

  roles:
    - geerlingguy.fluentd
```

## License

MIT / BSD

## Author Information

This role was created in 2020 by [Jeff Geerling](https://www.jeffgeerling.com/), author of [Ansible for DevOps](https://www.ansiblefordevops.com/).
