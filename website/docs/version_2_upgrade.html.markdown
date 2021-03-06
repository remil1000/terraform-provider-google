---
layout: "google"
page_title: "Terraform Google Provider Version 2 Upgrade Guide"
sidebar_current: "docs-google-provider-version-2-upgrade"
description: |-
  Terraform Google Provider Version 2 Upgrade Guide
---

# Terraform Google Provider Version 2 Upgrade Guide

Version 2.0.0 of the Google provider for Terraform is a major release and includes some changes that you will need to consider when upgrading. This guide is intended to help with that process and focuses only on changes from version 1.19.1 to version 2.0.0.

Most of the changes outlined in this guide have been previously marked as deprecated in the Terraform plan/apply output throughout previous provider releases, up to and including 1.19.1. These changes, such as deprecation notices, can always be found in the [Terraform Google Provider CHANGELOG](https://github.com/terraform-providers/terraform-provider-google/blob/master/CHANGELOG.md).

Upgrade topics:

<!-- TOC depthFrom:2 depthTo:2 -->

- [Provider Version Configuration](#provider-version-configuration)
- [`google-beta` provider](#google-beta-provider)
- [Open in Cloud Shell](#open-in-cloud-shell)
- [Resource: `google_bigtable_instance`](#resource-google_bigtable_instance)
- [Resource: `google_compute_instance`](#resource-google_compute_instance)
- [Resource: `google_compute_instance_from_template`](#resource-google_compute_instance_from_template)
- [Resource: `google_compute_project_metadata`](#resource-google_compute_project_metadata)
- [Resource: `google_compute_url_map`](#resource-google_compute_url_map)
- [Resource: `google_project_iam_policy`](#resource-google_project_iam_policy)
- [Resource: `google_sql_database_instance`](#resource-google_sql_database_instance)
- [Resource: `google_storage_default_object_acl`](#resource-google_storage_default_object_acl)
- [Resource: `google_storage_object_acl`](#resource-google_storage_object_acl)

<!-- /TOC -->

## Provider Version Configuration

!> **WARNING:** This topic is placeholder documentation until version 2.0.0 is released later this year.

-> Before upgrading to version 2.0.0, it is recommended to upgrade to the most recent version of the provider (1.19.1) and ensure that your environment successfully runs [`terraform plan`](https://www.terraform.io/docs/commands/plan.html) without unexpected changes or deprecation notices.

It is recommended to use [version constraints when configuring Terraform providers](https://www.terraform.io/docs/configuration/providers.html#provider-versions). If you are following that recommendation, update the version constraints in your Terraform configuration and run [`terraform init`](https://www.terraform.io/docs/commands/init.html) to download the new version.

For example, given this previous configuration:

```hcl
provider "google" {
  # ... other configuration ...

  version = "~> 1.19.0"
}
```

An updated configuration:

```hcl
provider "google" {
  # ... other configuration ...

  version = "~> 2.0.0"
}
```

## `google-beta` provider

The `google-beta` provider is now necessary to be able to configure resources with beta features.
This new provider enables full import support of beta features and gives users who
wish to use only the most stable APIs and features more confidence that they are doing so
by continuing to use the `google` provider.

Beta GCP Features have no deprecation policy and no SLA, but are otherwise considered to be feature-complete
with only minor outstanding issues after their Alpha period. Beta is when GCP
features are publicly announced, and is when they generally become publicly
available. For more information see [the official documentation on GCP launch stages](https://cloud.google.com/terms/launch-stages).

Because the API for beta features can change before their GA launch, there may be breaking changes
in the `google-beta` provider in minor release versions. These changes will be announced in the
[Terraform `google-beta` Provider CHANGELOG](https://github.com/terraform-providers/terraform-provider-google-beta/blob/master/CHANGELOG.md).

To have resources at different API versions, set up provider blocks for each version:

```hcl
provider "google" {
  credentials = "${file("account.json")}"
  project     = "my-project-id"
  region      = "us-central1"
}

provider "google-beta" {
  credentials = "${file("account.json")}"
  project     = "my-project-id"
  region      = "us-central1"
}
```

In each resource, state which provider that resource should be used with:

```hcl
resource "google_compute_instance" "ga-instance" {
  provider = "google"

  # ...
}

resource "google_compute_instance" "beta-instance" {
  provider = "google-beta"

  # ...
}
```

See [Provider Versions](https://terraform.io/docs/providers/google/provider_versions.html)
for more details on how to use `google-beta`.

## Open in Cloud Shell

2.0.0 is the first release including Open in Cloud Shell. Examples in the documentation for
Magic Modules resources now have Open in Cloud Shell links in their documentation that open
them in an interactive editor and shell - all without leaving the browser. See the
[blog post announcing the feature](https://www.hashicorp.com/blog/kickstart-terraform-on-gcp-with-google-cloud-shell)
for more details.

## Resource: `google_bigtable_instance`

### `cluster_id`, `zone`, `num_nodes`, and `storage_type` have moved into a `cluster` block

Example previous configuration:

```hcl
resource "google_bigtable_instance" "instance" {
  name         = "tf-instance"
  cluster_id   = "tf-instance-cluster"
  zone         = "us-central1-b"
  num_nodes    = 3
  storage_type = "HDD"
}
```

Example updated configuration:

```hcl
resource "google_bigtable_instance" "instance" {
  name = "tf-instance"
  cluster {
    cluster_id   = "tf-instance-cluster"
    zone         = "us-central1-b"
    num_nodes    = 3
    storage_type = "HDD"
  }
}
```

### `zone` is no longer inferred from the provider

`cluster.zone` is now required, even if the provider block has a zone set.

## Resource: `google_compute_instance`

### `metadata` is now authoritative

Terraform will remove values not explicitly set in this field. Any `metadata` values
that were added outside of Terraform should be added to the config.

## Resource: `google_compute_instance_from_template`

### `metadata` is now authoritative

Terraform will remove values not explicitly set in this field. Any `metadata` values
that were added outside of Terraform should be added to the config.

## Resource: `google_compute_project_metadata`

### `metadata` is now authoritative

Terraform will remove values not explicitly set in this field. Any `metadata` values
that were added outside of Terraform should be added to the config.

## Resource: `google_compute_url_map`

### `host_rule`, `path_matcher`, and `test` are now authoritative

Terraform will remove values not explicitly set in these fields. Any `host_rule`, `path_matcher`, or `test`
values that were added outside of Terraform should be added to the config.

## Resource: `google_project_iam_policy`

### `policy_data` is now authoritative

Terraform will remove values not explicitly set in this field. Any `policy_data`
values that were added outside of Terraform should be added to the config.

### `authoritative`, `restore_policy`, and `disable_project` have been removed

Remove these fields from your config. Ensure that `policy_data` contains all
policy values that exist on the project.

This resource is very dangerous. Consider using `google_project_iam_binding` or
`google_project_iam_member` instead.

## Resource: `google_sql_database_instance`

### `settings` is now authoritative

Terraform will remove values not explicitly set in this field. Any settings
values that were added outside of Terraform should be added to the config.

## Resource: `google_storage_default_object_acl`

### `role_entity` is now authoritative

Terraform will remove values not explicitly set in this field. Any `role_entity`
values that were added outside of Terraform should be added to the config.

## Resource: `google_storage_object_acl`

### `role_entity` is now authoritative

Terraform will remove values not explicitly set in this field. Any `role_entity`
values that were added outside of Terraform should be added to the config.
For fine-grained management, use `google_storage_object_access_control`.
