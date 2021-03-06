---
title: "Tokens"
description: "Tokens are placeholders in a check definition that the agent replaces with entity information before executing the check. You can use tokens to fine-tune check attributes (like alert thresholds) on a per-entity level while reusing check definitions. Read the reference doc to learn about tokens."
weight: 180
version: "5.20"
product: "Sensu Go"
menu: 
  sensu-go-5.20:
    parent: reference
---

- [Manage entity labels](#manage-entity-labels)
- [Token specification](#token-specification)
  - [Token substitution syntax](#token-substitution-syntax)
  - [Token substitution default values](#token-substitution-default-values)
  - [Token substitution with quoted strings](#token-substitution-with-quoted-strings)
- [Unmatched tokens](#unmatched-tokens)
- [Token data type limitations](#token-data-type-limitations)
- [Examples](#examples)
  - [Token substitution for check thresholds](#token-substitution-for-check-thresholds)

Tokens are placeholders in a check definition that the agent replaces with entity information before executing the check.
You can use tokens to fine-tune check attributes (like alert thresholds) on a per-entity level while reusing the check definition.

When a check is scheduled to be executed by an agent, it first goes through a token substitution step.
The agent replaces any tokens with matching attributes from the entity definition, and then the check is executed.
Invalid templates or unmatched tokens return an error, which is logged and sent to the Sensu backend message transport.
Checks with token-matching errors are not executed.

Token substitution is supported for [check definition][7] `command` attributes and [hook][8] `command` attributes.
Only [entity attributes][4] are available for substitution.
Available attributes will always have [string values][9], such as labels and annotations.

## Manage entity labels

You can use token substitution with any defined [entity attributes][4], including custom labels.
See the [entity reference][6] for information about managing entity labels for proxy entities and agent entities.

## Token specification

Sensu Go uses the [Go template][1] package to implement token substitution.
Use double curly braces around the token and a dot before the attribute to be substituted: `{{ .system.hostname }}`.

### Token substitution syntax

Tokens are invoked by wrapping references to entity attributes and labels with double curly braces, such as `{{ .name }}` to substitute an entity's name.
Access nested Sensu [entity attributes][3] dot notation (for example, `system.arch`).

- `{{ .name }}` would be replaced with the [entity `name` attribute][3]
- `{{ .labels.url }}` would be replaced with a custom label called `url`
- `{{ .labels.disk_warning }}` would be replaced with a custom label called `disk_warning`
- `{{ index .labels "disk_warning" }}` would be replaced with a custom label called
  `disk_warning`
- `{{ index .labels "cpu.threshold" }}` would be replaced with a custom label called `cpu.threshold`

{{% notice note %}}
**NOTE**: When an annotation or label name has a dot (e.g. `cpu.threshold`), you must use the template index function syntax to ensure correct processing because the dot notation is also used for object nesting.
{{% /notice %}}

### Token substitution default values

If an attribute is not provided by the [entity][3], a token's default value will be substituted.
Token default values are separated by a pipe character and the word "default" (`| default`).
Use token default values to provide a fallback value for entities that are missing a specified token attribute.

For example, `{{.labels.url | default "https://sensu.io"}}` would be replaced with a custom label called `url`.
If no such attribute called `url` is included in the entity definition, the default (or fallback) value of `https://sensu.io` will be used to substitute the token.

### Token substitution with quoted strings

You can escape quotes to express quoted strings in token substitution templates as shown in the [Go template package examples][2].
For example, to provide `"substitution"` as a default value for entities that are missing the `website` attribute (including the quotation marks):

{{< highlight shell >}}
{{ .labels.website | default "\"substitution\"" }}
{{< /highlight >}}

## Unmatched tokens

If a token is unmatched during check preparation, the agent check handler will return an error, and the check will not be executed.
Unmatched token errors are similar to this example:

{{< highlight shell >}}
error: unmatched token: template: :1:22: executing "" at <.system.hostname>: map has no entry for key "System"
{{< /highlight >}}

Check config token errors are logged by the agent and sent to Sensu backend message transport as check failures.

## Token data type limitations

As part of the substitution process, Sensu converts all tokens to strings.
This means that tokens cannot be used as bare integer values or to access individual list items.

For example, token substitution **cannot** be used for specifying a check interval because the interval attribute requires an _integer_ value.
Token substitution **can** be used for alerting thresholds because those values are included within the command _string_.

## Examples

### Token substitution for check thresholds 

In this example [hook][8] and [check configuration][5], the `check-disk-usage.go` command accepts `-w` (warning) and `-c` (critical) arguments to indicate the thresholds (as percentages) for creating warning or critical events.
If no token substitutions are provided by an entity configuration, Sensu will use default values to create a warning event at 80% disk capacity (i.e. `{{ .labels.disk_warning | default 80 }}`) and a critical event at 90% capacity (i.e. `{{ .labels.disk_critical | default 90 }}`).

Hook configuration:

{{< language-toggle >}}

{{< highlight yml >}}
type: HookConfig
api_version: core/v2
metadata:
  name: disk_usage_details
  namespace: default
spec:
  command: du -h --max-depth=1 -c {{index .labels "disk_usage_root" | default "/"}}  2>/dev/null
  runtime_assets: null
  stdin: false
  timeout: 60
{{< /highlight >}}

{{< highlight json >}}
{
  "type": "HookConfig",
  "api_version": "core/v2",
  "metadata": {
    "name": "disk_usage_details",
    "namespace": "default"
  },
  "spec": {
    "command": "du -h --max-depth=1 -c {{index .labels \"disk_usage_root\" | default \"/\"}}  2>/dev/null",
    "runtime_assets": null,
    "stdin": false,
    "timeout": 60
  }
}
{{< /highlight >}}

{{< /language-toggle >}}

Check configuration: 

{{< language-toggle >}}

{{< highlight yml >}}
type: CheckConfig
api_version: core/v2
metadata:
  name: check-disk-usage
  namespace: default
spec:
  check_hooks:
  - non-zero:
    - disk_usage_details
  command: check-disk-usage.rb -w {{index .labels "disk_warning" | default 80}} -c
    {{.labels.disk_critical | default 90}}
  env_vars: null
  handlers: []
  high_flap_threshold: 0
  interval: 10
  low_flap_threshold: 0
  output_metric_format: ""
  output_metric_handlers: null
  proxy_entity_name: ""
  publish: true
  round_robin: false
  runtime_assets: null
  stdin: false
  subdue: null
  subscriptions:
  - staging
  timeout: 0
  ttl: 0
{{< /highlight >}}

{{< highlight json >}}
{
  "type": "CheckConfig",
  "api_version": "core/v2",
  "metadata": {
    "name": "check-disk-usage",
    "namespace": "default"
  },
  "spec": {
    "check_hooks": [
      {
        "non-zero": [
          "disk_usage_details"
        ]
      }
    ],
    "command": "check-disk-usage.rb -w {{index .labels \"disk_warning\" | default 80}} -c {{.labels.disk_critical | default 90}}",
    "env_vars": null,
    "handlers": [],
    "high_flap_threshold": 0,
    "interval": 10,
    "low_flap_threshold": 0,
    "output_metric_format": "",
    "output_metric_handlers": null,
    "proxy_entity_name": "",
    "publish": true,
    "round_robin": false,
    "runtime_assets": null,
    "stdin": false,
    "subdue": null,
    "subscriptions": [
      "staging"
    ],
    "timeout": 0,
    "ttl": 0
  }
}
{{< /highlight >}}

{{< /language-toggle >}}

The following example [entity][4] provides the necessary attributes to override the `.labels.disk_warning` and `labels.disk_critical` tokens declared above:

{{< language-toggle >}}

{{< highlight yml >}}
type: Entity
api_version: core/v2
metadata:
  annotations: null
  labels:
    disk_critical: "90"
    disk_warning: "80"
  name: example-hostname
  namespace: default
spec:
  deregister: false
  deregistration: {}
  entity_class: agent
  last_seen: 1542667231
  redact:
  - password
  - passwd
  - pass
  - api_key
  - api_token
  - access_key
  - secret_key
  - private_key
  - secret
  subscriptions:
  - entity:example-hostname
  - staging
  system:
    arch: amd64
    hostname: example-hostname
    network:
      interfaces:
      - addresses:
        - 127.0.0.1/8
        - ::1/128
        name: lo
      - addresses:
        - 10.0.2.15/24
        - fe80::26a5:54ec:cf0d:9704/64
        mac: 08:00:27:11:ad:d2
        name: enp0s3
      - addresses:
        - 172.28.128.3/24
        - fe80::a00:27ff:febc:be60/64
        mac: 08:00:27:bc:be:60
        name: enp0s8
    os: linux
    platform: centos
    platform_family: rhel
    platform_version: 7.4.1708
  user: agent
{{< /highlight >}}

{{< highlight json >}}
{
  "type": "Entity",
  "api_version": "core/v2",
  "metadata": {
    "name": "example-hostname",
    "namespace": "default",
    "labels": {
      "disk_warning": "80",
      "disk_critical": "90"
    },
    "annotations": null
  },
  "spec": {
    "entity_class": "agent",
    "system": {
      "hostname": "example-hostname",
      "os": "linux",
      "platform": "centos",
      "platform_family": "rhel",
      "platform_version": "7.4.1708",
      "network": {
        "interfaces": [
          {
            "name": "lo",
            "addresses": [
              "127.0.0.1/8",
              "::1/128"
            ]
          },
          {
            "name": "enp0s3",
            "mac": "08:00:27:11:ad:d2",
            "addresses": [
              "10.0.2.15/24",
              "fe80::26a5:54ec:cf0d:9704/64"
            ]
          },
          {
            "name": "enp0s8",
            "mac": "08:00:27:bc:be:60",
            "addresses": [
              "172.28.128.3/24",
              "fe80::a00:27ff:febc:be60/64"
            ]
          }
        ]
      },
      "arch": "amd64"
    },
    "subscriptions": [
      "entity:example-hostname",
      "staging"
    ],
    "last_seen": 1542667231,
    "deregister": false,
    "deregistration": {},
    "user": "agent",
    "redact": [
      "password",
      "passwd",
      "pass",
      "api_key",
      "api_token",
      "access_key",
      "secret_key",
      "private_key",
      "secret"
    ]
  }
}{{< /highlight >}}

{{< /language-toggle >}}


[1]: https://pkg.go.dev/text/template
[2]: https://pkg.go.dev/text/template?tab=doc#hdr-Examples
[3]: ../entities/#entities-specification
[4]: ../entities/
[5]: ../checks/
[6]: ../entities#manage-entity-labels
[7]: ../checks/#check-commands
[8]: ../hooks/
[9]: #token-data-type-limitations
