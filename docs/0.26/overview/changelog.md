---
title: "0.26 changelog"
description: "Release notes for Sensu Core and Sensu Enterprise releases based
  on Sensu Core version 0.26.x"
version: 0.26
weight: 5
---

# Changelog

## Releases

- [Core 0.26.0 Release Notes](#core-v0-26-0)

## Core 0.26.0 Release Notes {#core-v0-26-0}

Source: [GitHub.com][1]

**August 24, 2016** &mdash; Sensu Core version 0.26.0 has been released and is
available for immediate download. Please note the following improvements:

### IMPORTANT {#core-v0-26-0-important}

This release includes potentially breaking, backwards-incompatible changes:

- Event silencing is now built into Sensu Core, and a new `/silenced` API is
  now available, which may conflict with the legacy "silence stash" filtering
  provided by Sensu Plugin handlers that are based on the `sensu-plugin` gem. A
  new [`"handle_silenced": true` attribute][12] is available to disable this
  new built-in silencing functionality, which should be added to handler
  definitions for handlers that rely on "silence stashes".

  _NOTE: Event filtering is moving out of the Sensu Plugins project and into
  Sensu Core. Though "silence stashes" are still supported in the `sensu-plugin`
  gem version 1.4.2 (included in Sensu Core 0.26), this functionality will be
  deprecated in an upcoming release, along with all built-in filtering in the
  `sensu-plugin` gem. In the interim, the Sensu embedded Ruby environment in
  versions 0.26.x will includes versions of the `sensu-plugin` library that will
  print deprecation warnings by default. Set the check attribute
  `enable_deprecated_filtering: false` to disable the deprecated filtering
  behavior. Please refer to the [Deprecating Event Filtering in sensu-plugin][10]
  blog post for more information._

- The handler definition `subdue` attribute is no longer supported. Time-based
  filtering is now supported by the new [filter `when` attribute][5]. Please
  update your handler definitions accordingly.

- Check `subdue` definitions no longer support the `"at": "handler"`
  configuration setting.

### CHANGES {#core-v0-26-0-changes}

- **NEW**: Event silencing is now built into Sensu Core! The Sensu API now
  provides a set of [`/silenced` endpoints][11], for silencing one or more
  checks and/or subscriptions (including the NEW client-specific subscriptions,
  below). Silencing applies to all event handlers by default, however a new
  `handle_silenced` handler definition attribute can be used to disable this
  functionality. Metric check events (OK) bypass event silencing.

  _NOTE: this improvement is very closely related to the impending deprecation
  of event filtering in the `sensu-plugin` gem. See the recent [Deprecating
  Event Filtering in sensu-plugin][10] blog post for more information._

- **NEW**: Every Sensu client now creates and subscribes to a unique client
  subscription (e.g. `client:i-424242`). Unique client subscriptions allow Sensu
  checks to target a single client (host) and enable silencing events for a
  single client. **Fixes: [#1327][1327]**.

- **NEW**: Introducing **Subdue 2.0**! Sensu `subdue` rules have a brand new
  configuration syntax, adding support for a broader number of applications, and
  `subdue` definitions are now supported by standalone checks.

  By way of comparison, the legacy `subdue` definition specification was
  limited to a single time window rule, with an array of exceptions. This was
  not only confusing, it made it very difficult to apply a simple "don't execute
  this check outside of 9-5, M-F" rule.

  ~~~ json
  {
    "checks": {
      "example_check": {
        "command": "check_something.rb",
        "...": "...",
        "subdue": {
          "begin": "12:00:00 AM",
          "end": "11:59:00 PM",
          "days": ["monday", "tuesday", "wednesday", "thursday", "friday"],
          "exceptions": [
            {
              "begin": "9:00:00 AM",
              "end": "5:00:00 PM"
            }
          ]
        }
      }
    }
  }
  ~~~

  The new syntax is more verbose, but by doing away with the need for
  `exceptions` and adding support for defining an array of subdue time windows,
  it is much easier to configure.

  ~~~ json
  {
    "checks": {
      "example_check": {
        "command": "check_something.rb",
        "...": "...",
        "subdue": {
          "days": {
            "monday": [
              {
                "begin": "12:00:00 AM PST",
                "end": "9:00:00 AM PST"
              },
              {
                "begin": "5:00:00 PM PST",
                "end": "11:59:59 PM PST"
              }
            ],
            "tuesday": [
              {
                "begin": "12:00:00 AM PST",
                "end": "9:00:00 AM PST"
              },
              {
                "begin": "5:00:00 PM PST",
                "end": "11:59:59 PM PST"
              }
            ],
            "wednesday": [
              {
                "begin": "12:00:00 AM PST",
                "end": "9:00:00 AM PST"
              },
              {
                "begin": "5:00:00 PM PST",
                "end": "11:59:59 PM PST"
              }
            ],
            "thursday": [
              {
                "begin": "12:00:00 AM PST",
                "end": "9:00:00 AM PST"
              },
              {
                "begin": "5:00:00 PM PST",
                "end": "11:59:59 PM PST"
              }
            ],
            "friday": [
              {
                "begin": "12:00:00 AM PST",
                "end": "9:00:00 AM PST"
              },
              {
                "begin": "5:00:00 PM PST",
                "end": "11:59:59 PM PST"
              }
            ],
            "saturday": [
              {
                "begin": "12:00:00 AM PST",
                "end": "11:59:59 PM PST"
              }
            ],
            "sunday": [
              {
                "begin": "12:00:00 AM PST",
                "end": "11:59:59 PM PST"
              }
            ]
          }
        }
      }
    }
  }
  ~~~

  _NOTE: Subdue rules now apply to check publishing, **ONLY** (i.e. `subdue`
  definitions no longer support the `"at": "handler"` definition attribute,
  among other changes). Prior to this release, `subdue` rules could be provided
  via [check definition `subdue` attribute][2] (i.e. `"at": "publisher"`) or the
  [handler definition `subdue` attribute][3] (i.e. `"at": "handler"`).
  Time-based filtering for handlers is now provided by Sensu filters (see
  below). Please refer to the new [`subdue` reference documentation][4] for more
  information._ **See: [#1415][1415]**.

- **NEW**: Event filters now support time-based rules, via a new `"when": {}`
  filter definition attribute. The filter `when` specification uses the same
  syntax as the new Subdue 2.0 specification, simplifying time-based event
  filtering. Please refer to the [filer `when` reference documentation][5] for
  more information. *See [#1415][1415]**.

- **NEW**: Sensu Extensions can now be loaded from Rubygems and enabled/disabled
  via configuration! The `sensu-install` has also added support for installing
  Sensu Extensions (e.g. `sensu-install -e system-profile`). Extensions gems
  must be enabled via configuration, please refer to the [Sensu extension
  reference documentation][6] for more information. **See: [#1394][1394]**.

- **NEW**: A check can now be a member of more than one [named aggregate][7],
  via a new check definition `"aggregates": []` attribute. **See: [#1379][1379];
  fixes [#1342][1342]**.

- **NEW**: Added support for setting Redis Sentinel configuration via a new
  `REDIS_SENTINEL_URLS` environment variable. Please refer to the [Sensu
  environment variables reference documentation][8] for more information. **See
  [#1411][1411]; fixes [#1361][1361].

- **NEW**: Added support for automatically discovering and setting client `name`
  and `address` attributes (two of the few required attributes for a valid
  Sensu client definition). **See: [#1379][1379]; fixes [#1362][1362]**.

- **IMPROVEMENT**: Added support for a new `occurrences_watermark` attribute,
  which is used by the built-in [sensu-occurrences-extension][9] filter to
  prevent sending resolve notifications for events that were not handled due to
  occurrence filtering. **See: [#1419][1419] and [#1427][1427]**.

- **IMPROVEMENT**: Only attempt to schedule standalone checks that have an
  interval. **See: [#1384][1384]; fixes [#1286][1286]**.

- **IMPROVEMENT**: Locally configured standalone checks (e.g. on a Sensu server)
  are no longer accessible via the Sensu API `/checks` endpoint. **See:
  [#1417][1417]; fixes [#1416][1416]**.

- **IMPROVEMENT**: Check TTL events are no longer created if the associated
  Sensu client has a current keepalive event. **See [#1428][1428]; fixes [#861][861]
  and [#1282][1282]**.

- **IMPROVEMENT**: Increased the maximum number of EventMachine timers from 100k
  to 200k, to accommodate very large Sensu installations that execute over 100k
  checks. **See [#1370][1370]; fixes [#1368][1368]**.

- **BUGFIX**: Fixed a Sensu API `/results` endpoint race condition that
  caused incomplete response content. **See [#1372][1372]; fixes [#1356][1356]**.

[1]:    https://github.com/sensu/sensu/blob/master/CHANGELOG.md
[2]:    ../../0.25/reference/checks.html#subdue-attributes
[3]:    ../../0.25/reference/handlers.html#subdue-attributes
[4]:    ../reference/checks.html#subdue-attributes
[5]:    ../reference/filters.html#when-attributes
[6]:    ../reference/extensions.html
[7]:    ../reference/aggregates.html
[8]:    ../reference/configuration.html#sensu-environment-variables
[9]:    https://github.com/sensu-extensions/sensu-extensions-occurrences/
[10]:   https://sensuapp.org/blog/2016/07/07/sensu-plugin-filter-deprecation.html
[11]:   ../api/silenced-api.html
[12]:   ../reference/handlers.html#handler-attributes
[861]:  https://github.com/sensu/sensu/issues/861
[1282]: https://github.com/sensu/sensu/issues/1282
[1286]: https://github.com/sensu/sensu/issues/1286
[1327]: https://github.com/sensu/sensu/issues/1327
[1342]: https://github.com/sensu/sensu/issues/1342
[1356]: https://github.com/sensu/sensu/issues/1356
[1361]: https://github.com/sensu/sensu/issues/1361
[1362]: https://github.com/sensu/sensu/issues/1362
[1368]: https://github.com/sensu/sensu/issues/1368
[1370]: https://github.com/sensu/sensu/pull/1370
[1372]: https://github.com/sensu/sensu/pull/1372
[1379]: https://github.com/sensu/sensu/pull/1379
[1384]: https://github.com/sensu/sensu/pull/1384
[1394]: https://github.com/sensu/sensu/pull/1394
[1411]: https://github.com/sensu/sensu/pull/1411
[1415]: https://github.com/sensu/sensu/pull/1415
[1417]: https://github.com/sensu/sensu/pull/1417
[1416]: https://github.com/sensu/sensu/issues/1416
[1419]: https://github.com/sensu/sensu/pull/1419
[1427]: https://github.com/sensu/sensu/pull/1427
[1428]: https://github.com/sensu/sensu/pull/1428
