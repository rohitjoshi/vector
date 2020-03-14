---
component_title: "Dedupe events"
description: "The Vector `dedupe` transform accepts and outputs `log` events allowing you to prevent duplicate Events from being outputted by using an LRU cache."
event_types: ["log"]
function_category: "filter"
issues_url: https://github.com/timberio/vector/issues?q=is%3Aopen+is%3Aissue+label%3A%22transform%3A+dedupe%22
min_version: null
service_name: "Dedupe events"
sidebar_label: "dedupe|[\"log\"]"
source_url: https://github.com/timberio/vector/tree/master/src/transforms/dedupe.rs
status: "prod-ready"
title: "Dedupe events Transform"
---

The Vector `dedupe` transform
accepts and outputs [`log`][docs.data-model.log] events allowing you to prevent
duplicate Events from being outputted by using an LRU cache.

<!--
     THIS FILE IS AUTOGENERATED!

     To make changes please edit the template located at:

     website/docs/reference/transforms/dedupe.md.erb
-->

## Configuration

import Tabs from '@theme/Tabs';

<Tabs
  block={true}
  defaultValue="common"
  values={[{"label":"Common","value":"common"},{"label":"Advanced","value":"advanced"}]}>

import TabItem from '@theme/TabItem';

<TabItem value="common">

import CodeHeader from '@site/src/components/CodeHeader';

<CodeHeader fileName="vector.toml" learnMoreUrl="/docs/setup/configuration/"/ >

```toml
[transforms.my_transform_id]
  # General
  type = "dedupe" # required
  inputs = ["my-source-id"] # required

  # Fields
  fields.match = ["timestamp", "host", "message"] # optional, default
```

</TabItem>
<TabItem value="advanced">

<CodeHeader fileName="vector.toml" learnMoreUrl="/docs/setup/configuration/"/ >

```toml
[transforms.my_transform_id]
  # General
  type = "dedupe" # required
  inputs = ["my-source-id"] # required

  # Cache
  cache.num_events = 5000 # optional, default

  # Fields
  fields.ignore = ["field1", "parent.child_field"] # optional, no default
  fields.match = ["timestamp", "host", "message"] # optional, default
```

</TabItem>
</Tabs>

## Options

import Fields from '@site/src/components/Fields';

import Field from '@site/src/components/Field';

<Fields filters={true}>


<Field
  common={false}
  defaultValue={null}
  enumValues={null}
  examples={[]}
  groups={[]}
  name={"cache"}
  path={null}
  relevantWhen={null}
  required={false}
  templateable={false}
  type={"table"}
  unit={null}
  >

### cache

Options controlling how we cache recent Events for future duplicate checking.



<Fields filters={false}>


<Field
  common={true}
  defaultValue={5000}
  enumValues={null}
  examples={[5000]}
  groups={[]}
  name={"num_events"}
  path={"cache"}
  relevantWhen={null}
  required={false}
  templateable={false}
  type={"int"}
  unit={null}
  >

#### num_events

The number of recent Events to cache and compare new incoming Events against.




</Field>


</Fields>

</Field>


<Field
  common={true}
  defaultValue={null}
  enumValues={null}
  examples={[]}
  groups={[]}
  name={"fields"}
  path={null}
  relevantWhen={null}
  required={true}
  templateable={false}
  type={"table"}
  unit={null}
  >

### fields

Options controlling what fields to match against



<Fields filters={false}>


<Field
  common={false}
  defaultValue={null}
  enumValues={null}
  examples={[["field1","parent.child_field"]]}
  groups={[]}
  name={"ignore"}
  path={"fields"}
  relevantWhen={null}
  required={false}
  templateable={false}
  type={"[string]"}
  unit={null}
  >

#### ignore

The field names to ignore when deciding if an Event is a duplicate.
Incompatible with the `fields.match` option.




</Field>


<Field
  common={true}
  defaultValue={["timestamp","host","message"]}
  enumValues={null}
  examples={[["field1","parent.child_field"]]}
  groups={[]}
  name={"match"}
  path={"fields"}
  relevantWhen={null}
  required={false}
  templateable={false}
  type={"[string]"}
  unit={null}
  >

#### match

The field names considered when deciding if an Event is a duplicate. This can
also be globally set via the [global `log_schema`
options][docs.reference.global-options#log_schema].Incompatible with the
`fields.ignore` option.




</Field>


</Fields>

</Field>


</Fields>

## How It Works

### Cache Behavior

This transform is backed by an LRU cache of size `cache.num_events`.
That means that this transform will cache information in memory for the last
`cache.num_events` Events that it has processed.  Entries will be removed from
the cache in the order they were inserted.  If an Event is received that is
considered a duplicate of an Event already in the cache that will put that event
back to the head of the cache and reset its place in line, making it once again
last entry in line to be evicted.

### Complex Processing

If you encounter limitations with the `dedupe`
transform then we recommend using a [runtime transform][urls.vector_programmable_transforms].
These transforms are designed for complex processing and give you the power of
full programming runtime.

### Environment Variables

Environment variables are supported through all of Vector's configuration.
Simply add `${MY_ENV_VAR}` in your Vector configuration file and the variable
will be replaced before being evaluated.

You can learn more in the
[Environment Variables][docs.configuration#environment-variables] section.

### Memory Usage Details

Each entry in the cache corresponds to an incoming Event and contains a copy of
the 'value' data for all fields in the Event being considered for matching.
When using `fields.match` this will be the list of fields specified in that
config option. When using `fields.ignore` that will include all fields present
in the incoming event *except* those specified in `fields.ignore`. Each entry
also uses a single byte per field to store the type information of that field.
When using `fields.ignore` each cache entry additionally stores a copy of each
field name being considered for matching.  When using `fields.match` storing
the field names is not necessary.

### Memory Utilization Estimation

If you want to estimate the memory requirements of this transform
for your dataset, you can do so with these formulas:

When using `fields.match`:

```text
Sum(the average size of the *data* (but not including the field name) for each field in `fields.match`) * `cache.num_events`
```

When using `fields.ignore`:

```text
(Sum(the average size of each incoming Event) - (the average size of the field name *and* value for each field in `fields.ignore`)) * `cache.num_events`
```

### Missing Fields

Fields with explicit null values will always be considered different than if
that field was omitted entirely.  For example, if you run this transform with
`fields.match = ["a"]`, the event "{a: null, b:5}" will be considered different
than the event "{b:5}".


[docs.configuration#environment-variables]: /docs/setup/configuration/#environment-variables
[docs.data-model.log]: /docs/about/data-model/log/
[docs.reference.global-options#log_schema]: /docs/reference/global-options/#log_schema
[urls.vector_programmable_transforms]: https://vector.dev/components?functions%5B%5D=program