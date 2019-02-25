---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
draft: true
tags: []
categories: []
type: "post"
toc: true
body_classes: "blog"
---

Post text<!--more-->

{{< figure figcaption="caption text" >}}

  {{< img src="filenameX" alt="alt text" >}}

{{< /figure >}}
