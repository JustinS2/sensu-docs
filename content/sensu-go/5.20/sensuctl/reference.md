---
title: "Sensuctl"
linkTitle: "Reference"
description: "Sensuctl is a command line tool for managing resources within Sensu. It works by calling Sensu’s underlying API to create, read, update, and delete resources, events, and entities. Read this reference doc to learn about sensuctl."
weight: 2
version: "5.20"
product: "Sensu Go"
platformContent: false 
menu:
  sensu-go-5.20:
    parent: sensuctl
---

- [First-time setup](#first-time-setup)
- [Get help](#get-help)
- [Manage sensuctl](#manage-sensuctl)
- [Test a user password](#test-a-user-password)
- [Create resources](#create-resources)
- [Delete resources](#delete-resources)
- [Update resources](#update-resources)
- [Export resources](#export-resources)
- [Manage resources](#manage-resources)
  - [Subcommands](#subcommands): [sensuctl check](#sensuctl-check) | [sensuctl cluster](#sensuctl-cluster) | [sensuctl event](#sensuctl-event) | [sensuctl namespace](#sensuctl-namespace) | [sensuctl user](#sensuctl-user) | [sensuctl prune](#sensuctl-prune) (alpha feature)
- [Response filtering](#response-filtering) (commercial feature)
- [Time formats](#time-formats)
- [Shell auto-completion](#shell-auto-completion)
- [Environment variables](#environment-variables)
- [Config files](#configuration-files)
- [Interact with Bonsai](#interact-with-bonsai)
  - [Install asset definitions](#install-asset-definitions)
  - [Check your Sensu backend for outdated assets](#check-your-sensu-backend-for-outdated-assets)
  - [Extend sensuctl with commands](#extend-sensuctl-with-commands)

Sensuctl is a command line tool for managing resources within Sensu.
It works by calling Sensu's underlying API to create, read, update, and delete resources, events, and entities.
Sensuctl is available for Linux, macOS, and Windows.
See [Install Sensu][4] to install and configure sensuctl.

## First-time setup

To set up sensuctl, run `sensuctl configure` to log in to sensuctl and connect to the Sensu backend:

{{< highlight shell >}}
sensuctl configure
{{< /highlight >}}

When prompted, type the [Sensu backend URL][9] and your [Sensu access credentials][11].

{{< highlight shell >}}
? Sensu Backend URL: http://127.0.0.1:8080
? Username: YOUR_USERNAME
? Password: YOUR_PASSWORD
? Namespace: default
? Preferred output format: tabular
{{< /highlight >}}

### Sensu backend URL

The Sensu backend URL is the HTTP or HTTPS URL where sensuctl can connect to the Sensu backend server.
The default URL is `http://127.0.0.1:8080`.

To connect to a [Sensu cluster][7], connect sensuctl to any single backend in the cluster.
For information about configuring the Sensu backend URL, see the [backend reference][5].

### Username, password, and namespace

During the [Sensu backend installation][40] process, you create an administrator username and password and a `default` namespace.

Your ability to get, list, create, update, and delete resources with sensuctl depends on the permissions assigned to your Sensu user.
For more information about configuring Sensu access control, see the [RBAC reference][1].

{{% notice note %}}
**NOTE**: For a **new** installation, you can set administrator credentials with environment variables during [initialization](../../reference/backend/#initialization).
If you are using Docker and you do not include the environment variables to set administrator credentials, the backend will initialize with the default username (`admin`) and password (`P@ssw0rd!`).
{{% /notice %}} 

#### Change admin user's password

After you have [installed and configured sensuctl][46], you can change the admin user's password.
Run:

{{< highlight shell >}}
sensuctl user change-password --interactive
{{< /highlight >}}

### Preferred output format

Sensuctl supports the following output formats:

- `tabular`: A user-friendly, columnar format
- `wrapped-json`: An accepted format for use with [`sensuctl create`][8]
- `yaml`: An accepted format for use with [`sensuctl create`][8]
- `json`: A format used by the [Sensu API][25]

After you are logged in, you can change the output format with `sensuctl config set-format` or set the output format per command with the `--format` flag.

### Non-interactive mode

Run `sensuctl configure` non-interactively by adding the `-n` (`--non-interactive`) flag.

{{< highlight shell >}}
sensuctl configure -n --url http://127.0.0.1:8080 --username YOUR_USERNAME --password YOUR_PASSWORD --format tabular
{{< /highlight >}}

## Get help

Sensuctl supports a `--help` flag for each command and subcommand.

### See command and global flags

{{< highlight shell >}}
sensuctl --help
{{< /highlight >}}

### See subcommands and flags

{{< highlight shell >}}
sensuctl check --help
{{< /highlight >}}

### See usage and flags

{{< highlight shell >}}
sensuctl check delete --help
{{< /highlight >}}

## Manage sensuctl

The `sencutl config` command lets you view the current sensuctl configuration and set the namespace and output format.

### View sensuctl config

To view the active configuration for sensuctl:

{{< highlight shell >}}
sensuctl config view
{{< /highlight >}}

The `sensuctl config view` response includes the [Sensu backend URL][9], default [namespace][11] for the current user, default [output format][10] for the current user, and currently configured username:

{{< highlight shell >}}
=== Active Configuration
API URL:   http://127.0.0.1:8080
Namespace: default
Format:    tabular
Username:  admin
{{< /highlight >}}

### Set output format

Use the `set-format` command to change the default [output format][10] for the current user.

For example, to change the output format to `tabular`:

{{< highlight shell >}}
sensuctl config set-format tabular
{{< /highlight >}}

### Set namespace

Use the `set-namespace` command to change the default [namespace][11] for the current user.
For more information about configuring Sensu access control, see the [RBAC reference][1].

For example, to change the default namespace to `development`:

{{< highlight shell >}}
sensuctl config set-namespace development
{{< /highlight >}}

### Log out of sensuctl

To log out of sensuctl:

{{< highlight shell >}}
sensuctl logout
{{< /highlight >}}

To log back in to sensuctl:

{{< highlight shell >}}
sensuctl configure
{{< /highlight >}}

### View the sensuctl version number

To display the current version of sensuctl:

{{< highlight shell >}}
sensuctl version
{{< /highlight >}}

### Global flags

Global flags modify settings specific to sensuctl, such as the Sensu backend URL and [namespace][11].
You can use global flags with most sensuctl commands.

{{< highlight shell >}}
--api-url string             host URL of Sensu installation
--cache-dir string           path to directory containing cache & temporary files
--config-dir string          path to directory containing configuration files
--insecure-skip-tls-verify   skip TLS certificate verification (not recommended!)
--namespace string           namespace in which we perform actions
--trusted-ca-file string     TLS CA certificate bundle in PEM format
{{< /highlight >}}

You can set these flags permanently by editing `.config/sensu/sensuctl/{cluster, profile}`.

## Test a user password

To test the password for a user created with Sensu's built-in [basic authentication][44]:

{{< highlight shell >}}
sensuctl user test-creds USERNAME --password 'password'
{{< /highlight >}}

An empty response indicates valid credentials.
A `request-unauthorized` response indicates invalid credentials.

{{% notice note %}}
**NOTE**: The `sensuctl user test-creds` command tests passwords for users created with Sensu's built-in [basic authentication provider](../../installation/auth#use-built-in-basic-authentication).
It does not test user credentials defined via an authentication provider like [Lightweight Directory Access Protocol (LDAP)](../../installation/auth/#ldap-authentication) or [Active Directory (AD)](../../installation/auth/#ad-authentication). 
{{% /notice %}}

For example, if you test LDAP credentials with the `sensuctl user test-creds` command, the backend will log an error, even if you know the LDAP credentials are correct:

{{< highlight shell >}}
{"component":"apid.routers","error":"basic provider is disabled","level":"info","msg":"invalid username and/or password","time":"2020-02-07T20:42:14Z","user":"dev"}
{{< /highlight >}}

## Create resources

The `sensuctl create` command allows you to create or update resources by reading from STDIN or a flag configured file (`-f`).
The `create` command accepts Sensu resource definitions in `wrapped-json` and `yaml`.
Both JSON and YAML resource definitions wrap the contents of the resource in `spec` and identify the resource `type`.
See the [`wrapped-json`example][39] and [this table][3] for a list of supported types.
See the [reference docs][6] for information about creating resource definitions.

### `wrapped-json` format

In this example, the file `my-resources.json` specifies two resources: a `marketing-site` check and a `slack` handler, separated _without_ a comma:

{{< highlight shell >}}
{
  "type": "CheckConfig",
  "api_version": "core/v2",
  "metadata" : {
    "name": "marketing-site",
    "namespace": "default"
    },
  "spec": {
    "command": "check-http.rb -u https://sensu.io",
    "subscriptions": ["demo"],
    "interval": 15,
    "handlers": ["slack"]
  }
}
{
  "type": "Handler",
  "api_version": "core/v2",
  "metadata": {
    "name": "slack",
    "namespace": "default"
  },
  "spec": {
    "command": "sensu-slack-handler --channel '#monitoring'",
    "env_vars": [
      "SLACK_WEBHOOK_URL=https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXXXXXX"
    ],
    "filters": [
      "is_incident",
      "not_silenced"
    ],
    "handlers": [],
    "runtime_assets": [],
    "timeout": 0,
    "type": "pipe"
  }
}
{{< /highlight >}}

To create all resources from `my-resources.json` using `sensuctl create`:

{{< highlight shell >}}
sensuctl create --file my-resources.json
{{< /highlight >}}

Or:

{{< highlight shell >}}
cat my-resources.json | sensuctl create
{{< /highlight >}}

### `yaml` format

In this example, the file `my-resources.yml` specifies two resources: a `marketing-site` check and a `slack` handler, separated with three dashes (`---`).

{{< highlight yml >}}
---
type: CheckConfig
api_version: core/v2
metadata:
  name: marketing-site
  namespace: default
spec:
  command: check-http.rb -u https://sensu.io
  subscriptions:
  - demo
  interval: 15
  handlers:
  - slack
---
type: Handler
api_version: core/v2
metadata:
  name: slack
  namespace: default
spec:
  command: sensu-slack-handler --channel '#monitoring'
  env_vars:
  - SLACK_WEBHOOK_URL=https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXXXXXX
  filters:
  - is_incident
  - not_silenced
  type: pipe
{{< /highlight >}}

To create all resources from `my-resources.yml` using `sensuctl create`:

{{< highlight shell >}}
sensuctl create --file my-resources.yml
{{< /highlight >}}

Or:

{{< highlight shell >}}
cat my-resources.yml | sensuctl create
{{< /highlight >}}

### sensuctl create resource types

|sensuctl create types |   |   |   |
--------------------|---|---|---|
`AdhocRequest` | `adhoc_request` | `Asset` | `asset`
`CheckConfig` | `check_config` | `ClusterRole`  | `cluster_role`
`ClusterRoleBinding`  | `cluster_role_binding` | `Entity` | [`Env`][43]
`entity` | [`EtcdReplicators`][35] | `Event` | `event`
`EventFilter` | `event_filter` | `Handler` | `handler`
`Hook` | `hook` | `HookConfig` | `hook_config`
`Mutator` | `mutator` | `Namespace` | `namespace`
`Role` | `role` | `RoleBinding` | `role_binding`
[`Secret`][41] | `Silenced` | `silenced` | [`VaultProvider`][43]
[`ldap`][26] | [`ad`][42] | [`TessenConfig`][27] | [`PostgresConfig`][32] |

### Create resources across namespaces

If you omit the `namespace` attribute from resource definitions, you can use the `senusctl create --namespace` flag to specify the namespace for a group of resources at the time of creation.
This allows you to replicate resources across namespaces without manual editing.
To learn more about namespaces and namespaced resource types, see the [RBAC reference][21].

The `sensuctl create` command applies namespaces to resources in the following order, from highest precedence to lowest:

1. **Namespaces specified within resource definitions**: You can specify a resource's namespace within individual resource definitions using the `namespace` attribute. Namespaces specified in resource definitions take precedence over all other methods.
2. **`--namespace` flag**: If resource definitions do not specify a namespace, Sensu applies the namespace provided by the `sensuctl create --namespace` flag.
3. **Current sensuctl namespace configuration**: If you do not specify an embedded `namespace` attribute or use the `--namespace` flag, Sensu applies the namespace configured in the current sensuctl session. See [Manage sensuctl][31] to view your current session config and set the session namespace.

In this example, the file `pagerduty.yml` defines a handler _without_ a `namespace` attribute:

{{< highlight shell >}}
type: Handler
api_version: core/v2
metadata:
  name: pagerduty
spec:
  command: sensu-pagerduty-handler
  env_vars:
  - PAGERDUTY_TOKEN=SECRET
  type: pipe
{{< /highlight >}}

To create the `pagerduty` handler in the `default` namespace:

{{< highlight shell >}}
sensuctl create --file pagerduty.yml --namespace default
{{< /highlight >}}

To create the `pagerduty` handler in the `production` namespace:

{{< highlight shell >}}
sensuctl create --file pagerduty.yml --namespace production
{{< /highlight >}}

To create the `pagerduty` handler in the current session namespace:

{{< highlight shell >}}
sensuctl create --file pagerduty.yml
{{< /highlight >}}

## Delete resources

The `sensuctl delete` command allows you to delete resources by reading from STDIN or a flag configured file (`-f`).
The `delete` command accepts Sensu resource definitions in `wrapped-json` and `yaml` formats and uses the same [resource types][3] as `sensuctl create`.
To be deleted successfully, resources provided to the `delete` command must match the name and namespace of an existing resource.

To delete all resources from `my-resources.yml` with `sensuctl delete`:

{{< highlight shell >}}
sensuctl delete --file my-resources.yml
{{< /highlight >}}

Or:

{{< highlight shell >}}
cat my-resources.yml | sensuctl delete
{{< /highlight >}}

### Delete resources across namespaces

If you omit the `namespace` attribute from resource definitions, you can use the `senusctl delete --namespace` flag to specify the namespace for a group of resources at the time of deletion.
This allows you to remove resources across namespaces without manual editing.
See the [Create resources across namespaces][33] section for usage examples.

## Update resources

Sensuctl allows you to update resource definitions with a text editor.
To use `sensuctl edit`, specify the resource [type][24] and resource name.

For example, to edit a handler named `slack` with `sensuctl edit`:

{{< highlight shell >}}
sensuctl edit handler slack
{{< /highlight >}}

### sensuctl edit resource types

|sensuctl edit types |   |   |   |
--------------------|---|---|---|
`asset` | `check` | `cluster` | `cluster-role`
`cluster-role-binding` | `entity` | `event` | `filter`
`handler` | `hook` | `mutator` | `namespace`
`role` | `role-binding` | `silenced` | `user`
[`auth`][26] | | |

## Export resources

The `sensuctl dump` command allows you to export your resources to standard out or to a file.
You can export all of your resources or a subset of them based on a list of resource types.
The `dump` command supports exporting in `wrapped-json` and `yaml`.

{{% notice note %}}
**NOTE**: Passwords are not included when exporting users. You must add the `password` attribute to any exported user resources before they can be used with `sensuctl create`.
{{% /notice %}}

To export all resources to a file named `my-resources.yaml` in `yaml` format:

{{< highlight shell >}}
sensuctl dump all --format yaml --file my-resources.yaml
{{< /highlight >}}

To export only checks to standard out in `yaml` format:

{{< highlight shell >}}
sensuctl dump check --format yaml
{{< /highlight >}}

To export only handlers and filters to a file named `my-handlers-and-filters.yaml` in `yaml` format:

{{< highlight shell >}}
sensuctl dump handler,filter --format yaml --file my-handlers-and-filters.yaml
{{< /highlight >}}

### sensuctl dump resource types

You can use the sensuctl `--types` subcommand to list the types of supported resources:

{{< highlight shell >}}
sensuctl dump --types
{{< /highlight >}}

The table below lists supported `sensuctl dump` resource types.

{{% notice note %}}
**NOTE**: The resource types with no synonym listed are [commercial features](../../getting-started/enterprise/).
{{% /notice %}}

Synonym | Fully qualified name 
--------------------|---
None | `authentication/v2.Provider`
None | `licensing/v2.LicenseFile`
None | `store/v1.PostgresConfig`
None | `federation/v1.Replicator`
None | `secrets/v1.Provider`
None | `secrets/v1.Secret`
`assets` | `core/v2.Asset`
`checks` | `core/v2.CheckConfig`
`clusterroles` | `core/v2.ClusterRole`
`clusterrolebindings` | `core/v2.ClusterRoleBinding`
`entities` | `core/v2.Entity`
`events` | `core/v2.Event` 
`filters` | `core/v2.EventFilter`
`handlers` | `core/v2.Handler`
`hooks` | `core/v2.Hook`
`mutators` | `core/v2.Mutator`
`namespaces` | `core/v2.Namespace`
`roles` | `core/v2.Role`
`rolebindings` | `core/v2.RoleBinding`
`silenced` | `core/v2.Silenced`
`tessen` | `core/v2.TessenConfig`
`users` | `core/v2.User`

## Manage resources

Sensuctl provides the following commands to manage Sensu resources.

- [`sensuctl asset`][12]
- [`sensuctl auth`][26] (commercial feature)
- [`sensuctl check`][13]
- [`sensuctl cluster`][7]
- [`sensuctl cluster-role`][1]
- [`sensuctl cluster-role-binding`][1]
- [`sensuctl entity`][14]
- [`sensuctl event`][15]
- [`sensuctl filter`][16]
- [`sensuctl handler`][17]
- [`sensuctl hook`][18]
- [`sensuctl license`](../../reference/license) (commercial feature)
- [`sensuctl mutator`][19]
- [`sensuctl namespace`][1]
- [`sensuctl role`][1]
- [`sensuctl role-binding`][1]
- [`sensuctl secrets`][41]
- [`sensuctl silenced`][20]
- [`sensuctl tessen`][27]
- [`sensuctl user`][1]

### Subcommands

Sensuctl provides a standard set of list, info, and delete operations for most resource types.

{{< highlight shell >}}
list                       list resources
info NAME                  show detailed resource information given resource name
delete NAME                delete resource given resource name
{{< /highlight >}}

For example, to list all monitoring checks:

{{< highlight shell >}}
sensuctl check list
{{< /highlight >}}

To list checks from all namespaces:

{{< highlight shell >}}
sensuctl check list --all-namespaces
{{< /highlight >}}

To write all checks to `my-resources.json` in `wrapped-json` format:

{{< highlight shell >}}
sensuctl check list --format wrapped-json > my-resources.json
{{< /highlight >}}

To see the definition for a check named `check-cpu` in [`wrapped-json` format][10]:

{{< highlight shell >}}
sensuctl check info check-cpu --format wrapped-json
{{< /highlight >}}

In addition to the standard operations, commands may support subcommands or flags that allow you to take special action based on the resource type.
The sections below describe these resource-specific operations.

For a list of subcommands specific to a resource, run `sensuctl TYPE --help`.

#### Handle large datasets

When querying sensuctl for large datasets, use the `--chunk-size` flag with any `list` command to avoid timeouts and improve performance.

For example, the following command returns the same output as `sensuctl event list` but makes multiple API queries (each for the number of objects specified by `--chunk-size`) instead of one API query for the complete dataset:

{{< highlight shell >}}
sensuctl event list --chunk-size 500
{{< /highlight >}}

#### sensuctl check

In addition to the [standard subcommands][23], the `sensuctl check execute` command executes a check on demand, given the check name:

{{< highlight shell >}}
sensuctl check execute NAME
{{< /highlight >}}

For example, the following command executes the `check-cpu` check with an attached message:

{{< highlight shell >}}
sensuctl check execute check-cpu --reason "giving a sensuctl demo"
{{< /highlight >}}

You can also use the `--subscriptions` flag to override the subscriptions in the check definition:

{{< highlight shell >}}
sensuctl check execute check-cpu --subscriptions demo,webserver
{{< /highlight >}}

#### sensuctl cluster

The `sensuctl cluster` command lets you manage a Sensu cluster using the following subcommands:

{{< highlight shell >}}
health           get Sensu health status
id               get unique Sensu cluster ID
member-add       add cluster member to an existing cluster, with comma-separated peer addresses
member-list      list cluster members
member-remove    remove cluster member by ID
member-update    update cluster member by ID with comma-separated peer addresses
{{< /highlight >}}

To view cluster members:

{{< highlight shell >}}
sensuctl cluster member-list
{{< /highlight >}}

To see the health of your Sensu cluster:

{{< highlight shell >}}
sensuctl cluster health
{{< /highlight >}}

#### sensuctl event

In addition to the [standard subcommands][23], you can use `sensuctl event resolve` to manually resolve events:

{{< highlight shell >}}
sensuctl event resolve ENTITY CHECK
{{< /highlight >}}

For example, the following command manually resolves an event created by the entity `webserver1` and the check `check-http`:

{{< highlight shell >}}
sensuctl event resolve webserver1 check-http
{{< /highlight >}}

#### sensuctl namespace

See the [RBAC reference][21] for information about using access control with namespaces.

#### sensuctl user

See the [RBAC reference][22] for information about local user management with sensuctl.

#### sensuctl prune

{{% notice important %}}
**IMPORTANT**: `sensuctl prune` is an alpha feature in release 5.19.0 and may include breaking changes.
{{% /notice %}}

**COMMERCIAL FEATURE**: Access sensuctl pruning in the packaged Sensu Go distribution.
For more information, see [Get started with commercial features][30].

The `sensuctl prune` subcommand allows you to delete resources that do not appear in a given set of Sensu objects (called a "configuration") from a from a file, URL, or STDIN.
For example, you can use `sensuctl create` to to apply a new configuration, then use `sensuctl prune` to prune unneeded resources, resources that were created by a specific user or that include a specific label selector, and more.

`sensuctl prune` can only delete resources that have the label `sensu.io/managed_by: sensuctl`, which Sensu automatically adds to all resources created with sensuctl.
This means you can only use `sensuctl prune` to delete resources that were created with sensuctl.

The pruning operation always follows the role-based access control (RBAC) permissions of the current user.
For example, to prune resources in the `dev` namespace, the current user who sends the prune command must have delete access to the `dev` namespace.

##### sensuctl prune usage

{{< highlight shell >}}
sensuctl prune [RESOURCE TYPE],[RESOURCE TYPE]... -f [FILE or URL] [-r] ... ] [--NAMESPACE] [flags]
{{< /highlight >}}

In this example `sensuctl prune` command:

- Replace [RESOURCE TYPE] with the [synonym or fully qualified name][48] of the resource you want to prune.
You must specify at least one resource type or the `all` qualifier (to prune all resource types).
- Replace [FILE or URL] with the name of the file or the URL that contains the set of Sensu objects you want to keep (the configuration).
- Replace [flags] with the flags you want to use, if any.
- Replace [--NAMESPACE] with the namespace where you want to apply pruning.
If you omit the namespace qualifier, the command defaults to the current configured namespace.

Use a comma separator to prune more than one resource in a single command.

For example, to prune checks and assets from the file `checks.yaml` for the `dev` namespace and the `admin` and `ops` users:

{{< highlight shell >}}
sensuctl prune checks,assets --file checks.yaml --namespace dev --users admin,ops
{{< /highlight >}}

##### sensuctl prune flags

Run `sensuctl prune -h` to view command-specific and global flags.
The following table describes the command-specific flags.

| Flag | Function and important notes
| ---- | ----------------------------
`-a` or `--all-users` | Prunes resources created by all users. Mutually exclusive with the `--users` flag. Defaults to false.
`-c` or `--cluster-wide` | Prunes any cluster-wide (non-namespaced) resources that are not defined in the configuration. Defaults to false.
`-d` or `--dry-run` | Prints the resources that will be pruned but does not actually delete them. Defaults to false.
`-f` or `--file` | Files, URLs, or directories to prune resources from. Strings.
`-h` or `--help` | Help for the prune command.
`--label-selector` | Prunes only resources that match the specified labels (comma-separated strings). Labels are a [commercial feature][30].
`-r` or `--recursive` | Prune command will follow subdirectories.
`-u` or `--users` | Prunes only resources that were created by the specified users (comma-separated strings). Defaults to the currently configured sensuctl user.

##### sensuctl prune resource types

The table below lists supported `sensuctl prune` resource types.

Synonym | Fully qualified name 
--------------------|---
None | `authentication/v2.Provider`
None | `licensing/v2.LicenseFile`
None | `store/v1.PostgresConfig`
None | `federation/v1.EtcdReplicator`
None | `secrets/v1.Provider`
None | `secrets/v1.Secret`
`apikey` | `core/v2.APIKey`
`assets` | `core/v2.Asset`
`checks` | `core/v2.CheckConfig`
`clusterroles` | `core/v2.ClusterRole`
`clusterrolebindings` | `core/v2.ClusterRoleBinding`
`entities` | `core/v2.Entity`
`filters` | `core/v2.EventFilter`
`handlers` | `core/v2.Handler`
`hooks` | `core/v2.Hook`
`mutators` | `core/v2.Mutator`
`namespaces` | `core/v2.Namespace`
`roles` | `core/v2.Role`
`rolebindings` | `core/v2.RoleBinding`
`silenced` | `core/v2.Silenced`
`tessen` | `core/v2.TessenConfig`
`users` | `core/v2.User`

##### sensuctl prune examples

`sensuctl prune` supports pruning resources by their synonyms or fully qualified names:

{{< highlight shell >}}
sensuctl prune checks,entities
{{< /highlight >}}

{{< highlight shell >}}
sensuctl prune core/v2.CheckConfig,core/v2.Entity
{{< /highlight >}}

Use the `all` qualifier to prune all supported resources:

{{< highlight shell >}}
sensuctl prune all
{{< /highlight >}}

## Response filtering

**COMMERCIAL FEATURE**: Access sensuctl response filters in the packaged Sensu Go distribution.
For more information, see [Get started with commercial features][30].

Sensuctl supports response filtering for all [commands using the `list` verb][23].
For information about response filtering methods and available label and field selectors, see [API response filtering][28].

### Sensuctl-specific syntax

You can use the same methods, selectors, and examples for sensuctl response filtering as for [API response filtering][28], except you'll format your requests with the `--label-selector` and `--field-selector` flags instead of cURL.

The standard sensuctl response filtering syntax is:

{{< highlight shell >}}
sensuctl RESOURCE_TYPE list --SELECTOR 'FILTER_STATEMENT'
{{< /highlight >}}

To create a sensuctl response filtering command:

- Replace `RESOURCE_TYPE` with the resource your filter is based on.
- Replace `SELECTOR` with either `label-selector` or `field-selector`, depending on which selector you want to use.
- Replace `FILTER_STATEMENT` with the filter to apply.

For example:

{{< highlight shell >}}
sensuctl event list --field-selector 'linux notin event.entity.subscriptions'
{{< /highlight >}}

Sensuctl response filtering commands will also work with a single equals sign between the selector flag and the filter statement:

{{< highlight shell >}}
sensuctl event list --field-selector='linux notin event.entity.subscriptions'
{{< /highlight >}}

The [examples][49] demonstrate how to construct sensuctl filter statements for different selectors and operators.

### Operators quick reference

Sensuctl response filtering supports two equality-based operators, two set-based operators, one substring matching operator, and one logical operator.

| operator  | description        | example                |
| --------- | ------------------ | ---------------------- |
| `==`      | Equality           | `check.publish == true`
| `!=`      | Inequality         | `check.namespace != "default"`
| `in`      | Included in        | `linux in check.subscriptions`
| `notin`   | Not included in    | `slack notin check.handlers`
| `matches` | Substring matching | `check.name matches "linux-"`
| `&&`      | Logical AND        | `check.publish == true && slack in check.handlers`

For details about operators, see [API response filtering operators][47].

### Examples

#### Filter responses with label selectors

Use the `--label-selector` flag to filter responses using custom labels.

For example, to return entities with the `proxy_type` label set to `switch`:

{{< highlight shell >}}
sensuctl entity list --label-selector 'proxy_type == switch'
{{< /highlight >}}

#### Filter responses with field selectors

Use the `--field-selector` flag to filter responses using specific [resource attributes][29].

For example, to return entities with the `switches` subscription:

{{< highlight shell >}}
sensuctl entity list --field-selector 'switches in entity.subscriptions'
{{< /highlight >}}

To retrieve all events that equal a status of `2` (CRITICAL):

{{< highlight shell >}}
sensuctl event list --field-selector 'event.check.status == "2"'
{{< /highlight >}}

To retrieve all entities whose name includes the substring `webserver`:

{{< highlight shell >}}
sensuctl entity list --fieldSelector 'entity.name matches "webserver"'
{{< /highlight >}}

#### Use the logical AND operator

To use the logical AND operator (`&&`) to return checks that include a `linux` subscription in the `dev` namespace:

{{< highlight shell >}}
sensuctl check list --field-selector 'linux in check.subscriptions && dev in check.namespace'
{{< /highlight >}}

#### Combine label and field selectors

You can combine the `--label-selector` and `--field-selector` flags in a single command.

For example, this command returns checks with the `region` label set to `us-west-1` that also use the `slack` handler:

{{< highlight shell >}}
sensuctl check list --label-selector 'region == "us-west-1"' --field-selector 'slack in check.handlers'
{{< /highlight >}}

## Time formats

Sensuctl supports multiple time formats depending on the manipulated resource.
Supported canonical time zone IDs are defined in the [tz database][2].

{{% notice warning %}}
**WARNING**: Windows does not support canonical zone IDs (for example, `America/Vancouver`).
{{% /notice %}}

### Dates with time

Use full dates with time to specify an exact point in time.
This is useful for setting silences, for example.

Sensuctl supports the following formats:

* RFC3339 with numeric zone offset: `2018-05-10T07:04:00-08:00` or
  `2018-05-10T15:04:00Z`
* RFC3339 with space delimiters and numeric zone offset: `2018-05-10 07:04:00
  -08:00`
* Sensu alpha legacy format with canonical zone ID: `May 10 2018 7:04AM
  America/Vancouver`

## Shell auto-completion

### Installation (Bash shell)

Make sure bash-completion is installed.
If you use a current Linux in a non-minimal installation, bash-completion should be available.

On macOS, install with:

{{< highlight shell >}}
brew install bash-completion
{{< /highlight >}}

Then add this to your `~/.bash_profile`:

{{< highlight shell >}}
if [ -f $(brew --prefix)/etc/bash_completion ]; then
. $(brew --prefix)/etc/bash_completion
fi
{{< /highlight >}}

After bash-completion is installed, add this to your `~/.bash_profile`:

{{< highlight shell >}}
source <(sensuctl completion bash)
{{< /highlight >}}

Now you can source your `~/.bash_profile` or launch a new terminal to use shell auto-completion.

{{< highlight shell >}}
source ~/.bash_profile
{{< /highlight >}}

### Installation (ZSH)

Add this to your `~/.zshrc`:

{{< highlight shell >}}
source <(sensuctl completion zsh)
{{< /highlight >}}

Now you can source your `~/.zshrc` or launch a new terminal to use shell auto-completion.

{{< highlight shell >}}
source ~/.zshrc
{{< /highlight >}}

### Usage

`sensuctl` <kbd>Tab</kbd>

{{< highlight shell >}}
check       configure   event       user
asset       completion  entity      handler
{{< /highlight >}}

`sensuctl check` <kbd>Tab</kbd>

{{< highlight shell >}}
create  delete  import  list
{{< /highlight >}}

## Environment variables

Sensuctl includes the `sensuctl env` command to help export and set environment variables on your systems.

{{< highlight text >}}
SENSU_API_URL                    URL of the Sensu backend API in sensuctl
SENSU_NAMESPACE                  Name of the current namespace in sensuctl
SENSU_FORMAT                     Set output format in sensuctl (e.g. JSON, YAML, etc.)
SENSU_ACCESS_TOKEN               Current API access token in sensuctl
SENSU_ACCESS_TOKEN_EXPIRES_AT    Timestamp specifying when the current API access token expires
SENSU_REFRESH_TOKEN              Refresh token used to obtain a new access token
SENSU_TRUSTED_CA_FILE            Path to a trusted CA file if set in sensuctl
SENSU_INSECURE_SKIP_TLS_VERIFY   Boolean value that can be set to skip TLS verification
{{< /highlight >}}

### Usage

{{< language-toggle >}}

{{< highlight bash >}}
sensuctl env
export SENSU_API_URL="http://127.0.0.1:8080"
export SENSU_NAMESPACE="default"
export SENSU_FORMAT="tabular"
export SENSU_ACCESS_TOKEN="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.x.x"
export SENSU_ACCESS_TOKEN_EXPIRES_AT="1567716187"
export SENSU_REFRESH_TOKEN="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.x.x"
export SENSU_TRUSTED_CA_FILE=""
export SENSU_INSECURE_SKIP_TLS_VERIFY="true"

# Run this command to configure your shell:
# eval $(sensuctl env)
{{< /highlight >}}

{{< highlight cmd >}}
sensuctl env --shell cmd
SET SENSU_API_URL=http://127.0.0.1:8080
SET SENSU_NAMESPACE=default
SET SENSU_FORMAT=tabular
SET SENSU_ACCESS_TOKEN=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.x.x
SET SENSU_ACCESS_TOKEN_EXPIRES_AT=1567716676
SET SENSU_REFRESH_TOKEN=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.x.x
SET SENSU_TRUSTED_CA_FILE=
SET SENSU_INSECURE_SKIP_TLS_VERIFY=true
REM Run this command to configure your shell:
REM   @FOR /f "tokens=*" %i IN ('sensuctl env --shell cmd') DO @%i
{{< /highlight >}}

{{< highlight powershell >}}
sensuctl env --shell powershell
$Env:SENSU_API_URL = "http://127.0.0.1:8080"
$Env:SENSU_NAMESPACE = "default"
$Env:SENSU_FORMAT = "tabular"
$Env:SENSU_ACCESS_TOKEN = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.x.x"
$Env:SENSU_ACCESS_TOKEN_EXPIRES_AT = "1567716738"
$Env:SENSU_REFRESH_TOKEN = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.x.x"
$Env:SENSU_TRUSTED_CA_FILE = ""
$Env:SENSU_INSECURE_SKIP_TLS_VERIFY = "true"

# Run this command to configure your shell:
# & sensuctl env --shell powershell | Invoke-Expression
{{< /highlight >}}

{{< /language-toggle >}}

## Configuration files

During configuration, sensuctl creates configuration files that contain information for connecting to your Sensu Go deployment.
You can find these files at `$HOME/.config/sensu/sensuctl/profile` and `$HOME/.config/sensu/sensuctl/cluster`.

For example:

{{< highlight shell >}}
cat .config/sensu/sensuctl/profile
{
  "format": "tabular",
  "namespace": "demo",
  "username": "admin"
}
{{< /highlight >}}

{{< highlight shell >}}
cat .config/sensu/sensuctl/cluster 
{
  "api-url": "http://localhost:8080",
  "trusted-ca-file": "",
  "insecure-skip-tls-verify": false,
  "access_token": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  "expires_at": 1550082282,
  "refresh_token": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
}
{{< /highlight >}}

These configuration files are useful if you want to know which cluster you're connecting to or which namespace or username you're currently configured to use.

## Interact with Bonsai

Sensuctl supports installing asset definitions directly from [Bonsai, the Sensu asset index][34], and checking your Sensu backend for outdated assets.
You can also use `sensuctl command` to install, execute, list, and delete commands from Bonsai or a URL.

### Install asset definitions

To install an asset definition directly from Bonsai, use `sensuctl asset add [NAMESPACE/NAME][:VERSION]`.
`[:VERSION]` is only required if you require a specific version or are pinning to a specific version.

Replace `[NAMESPACE/NAME]` with the namespace and name of the asset from Bonsai:

![Bonsai page for InfluxDB handler showing namespace and name][36]

{{< highlight shell >}}
sensuctl asset add sensu/sensu-influxdb-handler:3.1.1
fetching bonsai asset: sensu/sensu-influxdb-handler:3.1.1
added asset: sensu/sensu-influxdb-handler:3.1.1
{{< /highlight >}}

You can also use the `--rename` flag to rename the asset on install:

{{< highlight shell >}}
sensuctl asset add sensu/sensu-slack-handler --rename slack-handler
no version specified, using latest: 1.0.3
fetching bonsai asset: sensu/sensu-slack-handler:1.0.3
added asset: sensu/sensu-slack-handler:1.0.3
{{< /highlight >}}

### Check your Sensu backend for outdated assets

To check your Sensu backend for assets that have newer versions available on Bonsai, use `sensuctl asset outdated`.
This will print a list of assets installed in the backend whose version is older than the newest version available on Bonsai:

{{< highlight shell >}}
sensuctl asset outdated
          Asset Name                  Bonsai Asset          Current Version  Latest Version
----------------------------  ----------------------------  ---------------  --------------
sensu/sensu-influxdb-handler  sensu/sensu-influxdb-handler       3.1.1            3.1.2
{{< /highlight >}}

### Extend sensuctl with commands

Use `sensuctl command` to install, execute, list, and delete commands from Bonsai or a URL.

#### Install commands

To install a sensuctl command from Bonsai or a URL:

{{< highlight shell >}}
sensuctl command install [ALIAS] ([NAMESPACE/NAME]:[VERSION] | --url [ARCHIVE_URL] --checksum [ARCHIVE_CHECKSUM]) [flags]
{{< /highlight >}}

To install a command plugin, use the Bonsai asset name or specify a URL and SHA512 checksum.

**To install a command using the Bonsai asset name**, replace `[NAMESPACE/NAME]` with the name of the asset from Bonsai.
`[:VERSION]` is only required if you require a specific version or are pinning to a specific version.
If you do not specify a version, sensuctl will fetch the latest version from Bonsai.

Replace `[ALIAS]` with a unique name for the command.
For example, for the [Sensu EC2 Discovery Plugin][37], you might use the alias `sensu-ec2-discovery`. 
`[ALIAS]` is required.

Replace `[flags]` with the flags you want to use.
Run `sensuctl command install -h` to view flags.
Flags are optional and apply only to the `install` command &mdash; they are not saved as part of the command you are installing.

To install a command from the [Sensu EC2 Discovery Plugin][37] with no flags:

{{< highlight shell >}}
sensuctl command install sensu-ec2-discovery portertech/sensu-ec2-discovery:0.3.0
{{< /highlight >}}

**To install a command from a URL**, replace `[ARCHIVE_URL]` with a command URL that points to a tarball (e.g. https://path/to/asset.tar.gz).
Replace `[ARCHIVE_CHECKSUM]` with the checksum you want to use.
Replace `[ALIAS]` with a unique name for the command.

Replace `[flags]` with the flags you want to use.
Run `sensuctl command install -h` to view flags.
Flags are optional and apply only to the `install` command &mdash; they are not saved as part of the command you are installing.

For example, to install a command-test asset via URL with no flags:

{{< highlight shell >}}
sensuctl command install command-test --url https://github.com/amdprophet/command-test/releases/download/v0.0.4/command-test_0.0.4_darwin_amd64.tar.gz --checksum 8b15a170e091dab42256fe64ca7c4a050ed49a9dbfd6c8129c95506a8a9a91f2762ac1a6d24f4fc545430613fd45abc91d3e5d3605fcfffb270dcf01996caa7f
{{< /highlight >}}

{{% notice note %}}
**NOTE**: Asset definitions with multiple asset builds are only supported via Bonsai.
{{% /notice %}}

#### Execute commands

To execute a sensuctl command plugin via its asset's bin/entrypoint executable:

{{< highlight shell >}}
sensuctl command exec [ALIAS] [args] [flags]
{{< /highlight >}}

Replace `[ALIAS]` with a unique name for the command.
For example, for the [Sensu EC2 Discovery Plugin][37], you might use the alias `sensu-ec2-discovery`. 
`[ALIAS]` is required.

Replace `[flags]` with the flags you want to use.
Run `sensuctl command exec -h` to view flags.
Flags are optional and apply only to the `exec` command &mdash; they are not saved as part of the command you are executing.

Replace `[args]` with the globlal flags you want to use.
Run `sensuctl command exec -h` to view global flags.
To pass `[args]` flags to the bin/entrypoint executable, make sure to specify them after a double dash surrounded by spaces.

{{% notice note %}}
**NOTE**: When you use `sensuctl command exec`, the [environment variables](#environment-variables) are passed to the command.
{{% /notice %}}

For example:

{{< highlight shell >}}
sensuctl command exec mycommand arg1 arg2 --cache-dir /tmp -- --flag1 --flag2=value
{{< /highlight >}}

Sensuctl will parse the --cache-dir flag, but bin/entrypoint will parse all flags after the ` -- `.

In this example, the full command run by sensuctl exec would be:

{{< highlight shell >}}
bin/entrypoint arg1 arg2 --flag1 --flag2=value
{{< /highlight >}}

#### List commands

To list installed sensuctl commands: 

{{< highlight shell >}}
sensuctl command list [flags]
{{< /highlight >}}

Replace `[flags]` with the flags you want to use.
Run `sensuctl command list -h` to view flags.
Flags are optional and apply only to the `list` command.

#### Delete commands

To delete sensuctl commands:

{{< highlight shell >}}
sensuctl command delete [ALIAS] [flags]
{{< /highlight >}}

Replace `[ALIAS]` with a unique name for the command.
For example, for the [Sensu EC2 Discovery Plugin][37], you might use the alias `sensu-ec2-discovery`. 
`[ALIAS]` is required.

Replace `[flags]` with the flags you want to use.
Run `sensuctl command delete -h` to view flags.
Flags are optional and apply only to the `delete` command.

[1]: ../../reference/rbac/
[2]: https://en.wikipedia.org/wiki/List_of_tz_database_time_zones
[3]: #sensuctl-create-resource-types
[4]: ../../installation/install-sensu/#install-sensuctl
[5]: ../../reference/agent/#general-configuration-flags
[6]: ../../reference/
[7]: ../../guides/clustering/
[8]: #create-resources
[9]: #sensu-backend-url
[10]: #preferred-output-format
[11]: #username-password-and-namespace
[12]: ../../reference/assets/
[13]: ../../reference/checks/
[14]: ../../reference/entities/
[15]: ../../reference/events/
[16]: ../../reference/filters/
[17]: ../../reference/handlers/
[18]: ../../reference/hooks/
[19]: ../../reference/mutators/
[20]: ../../reference/silencing/
[21]: ../../reference/rbac#namespaces
[22]: ../../reference/rbac#users
[23]: #subcommands
[24]: #sensuctl-edit-resource-types
[25]: ../../api/overview/
[26]: ../../installation/auth/#ldap-authentication
[27]: ../../reference/tessen/
[28]: ../../api/overview#response-filtering
[29]: ../../api/overview#field-selector
[30]: ../../getting-started/enterprise/
[31]: #manage-sensuctl
[32]: ../../reference/datastore/
[33]: #create-resources-across-namespaces
[34]: https://bonsai.sensu.io/
[35]: ../../reference/etcdreplicators/
[36]: /images/sensu-influxdb-handler-namespace.png
[37]: https://bonsai.sensu.io/assets/portertech/sensu-ec2-discovery
[39]: #wrapped-json-format
[40]: ../../installation/install-sensu/#install-the-sensu-backend
[41]: ../../reference/secrets/
[42]: ../../installation/auth/#ad-authentication
[43]: ../../reference/secrets-providers/
[44]: ../../installation/auth#use-built-in-basic-authentication
[45]: ../../installation/install-sensu/#2-configure-and-start
[46]: #first-time-setup
[47]: ../../api/overview/#operators
[48]: #sensuctl-prune-resource-types
[49]: #examples
