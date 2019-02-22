---
title: "{{ replace .TranslationBaseName "-" " " | title }}"
date: {{ .Date }}
draft: true
tags: []
categories: []
type: "post"
body_classes: "blog"
---

Post text<!--more-->

{{< figure figcaption="caption text" >}}

  {{< img src="filenameX" alt="alt text" >}}

{{< /figure >}}
