Mautrix bridge config updates last checked on 2024-04-22
To add a mautrix bridge, create a var file and based on an older one and a new config file like this:

```
{% extends 'mautrix-bridge.yaml.j2' %}

{% block bridge %}
    additional options for the bridge section, needs to be indented
{% endblock %}

{% block additional %}
additional sections here
{% endblock %}

```

Any block can be omitted if not needed.
