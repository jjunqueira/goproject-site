---
title: "Configuration"
date: 2017-10-17T15:26:15Z
lastmod: 2019-10-26T15:26:15Z
draft: false
weight: 20
---

As of right now there isn't much to configure. As the project expands this will change.

As a user the primary thing you would want to configure is a custom template. A custom template requires a name and a path. It is OK if a custom template has the same name as a default template. However keep in mind that a custom template will take precedence over a default one.

```
[sourcecontrol]
uri = "https://github.com"

[go]
vendor = false

[[custom_templates]]
name = "example1"
path = "/path/to/custom/templates/example1"

[[custom_templates]]
name = "example2"
path = "/path/to/custom/templates/example2"
```