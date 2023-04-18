Mautrix bridge config updates last checked on 2022-08-25
To add a mautrix bridge, create a var file and based on an older one and a new config file like this:

```
{% extends 'mautrix-bridge.yaml.j2' %}

{% block backfill %}
{{ super() }}
        additional options for the backfill section, needs to be indentded. Omit super()
        above if the required options are different from the usual.
{% endblock %}

{% block bridge %}
    additional options for the bridge section, needs to be indented
{% endblock %}

{% block additional %}
additional sections here
{% endblock %}

```

Any block can be omitted if not needed

Available blocks in vars:
```
mautrix_blocks:
  - public
  - provisioning
  - relay
  - delivery_error_reports
  - displayname_template
  - backfill
```
