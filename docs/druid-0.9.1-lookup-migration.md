# Migration to Druid 0.9.1 lookups

IAP 1.3.0 that ships with Druid 0.9.1 introduces a new way of making query time lookups (QTLs).
The [new](http://druid.io/docs/0.9.1/development/extensions-core/lookups-cached-global.html) and [old](http://druid.io/docs/0.9.1/development/extensions-core/namespaced-lookup.html) lookups are incompatible.
To facilitate a rolling migration [Druid 0.9.1 supports both styles of lookup](https://github.com/druid-io/druid/issues/2999).

If you are using Pivot with QTLs (`.lookup('...')` actions) and desire to have uninterrupted service,
you will need to follow these steps.

**Note:** these steps assume that Pivot is collocated with the broker node as it is configured in the [Imply Analytics Platform](http://imply.io/download).

### Step 1

Update the IAP Data Nodes to 1.3.0 first.

### Step 2

For the Query Node, as you roll out 1.3.0, update the Pivot config as follows:

Your config will go from looking like:

```yaml
# Old style config
druidHost: localhost:8082
timeout: 30000
sourceListScan: auto
sourceListRefreshOnLoad: true

# ...
```

To this:

```yaml
# New style config
clusters:
  - name: druid
    type: druid
    host: localhost:8082           # 'druidHost' becomes 'host'
    version: 0.9.1-legacy-lookups  # <- continue using old lookups for now
    timeout: 30000
    sourceListScan: auto
    sourceListRefreshOnLoad: true

# ...
```

Setting `version` to `0.9.1-legacy-lookups` instructs Pivot to use all the Druid 0.9.1 features except the new lookups.
Read more about this in the [general migration instructions](./pivot-0.9.x-migration.md).

### Step 3

Follow the [instructions](http://druid.io/docs/0.9.1/development/extensions-core/namespaced-lookup.html#transitioning-to-lookups-cached-global) and add the new style lookups alongside the legacy lookups to Druid.

### Step 4

Remove the explicit version pinning from the Pivot config.
Pivot will now auto detect your druid version as `0.9.1`.
Make sure to restart the Query Node.

### Step 5

Set a reminder to remove old lookup definitions from Druid next time you update.

