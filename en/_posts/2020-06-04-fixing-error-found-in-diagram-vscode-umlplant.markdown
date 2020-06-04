---
published: true
title: Fixing Error found in diagram Picked up _JAVA_OPTIONS -Dawt.useSystemAAFontSettings=gasp
layout: post
description: UMLPlant VSCode extesion will fail to render the diagram code without these steps
---

UMLPlant VSCode extesion will fail to render the diagram code without these steps...
If you're getting "Error found in diagram Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=gasp" 

You'll get a similar warning when running "java -version"

``` shell
$ java -version
Picked up _JAVA_OPTIONS:   -Dawt.useSystemAAFontSettings=gasp
openjdk version "1.8.0_171"
OpenJDK Runtime Environment (build 1.8.0_171-8u171-b11-2-b11)
OpenJDK 64-Bit Server VM (build 25.171-b11, mixed mode)
``` 

To fix this, add these lines to your ~/.bashrc (or the one you use):

``` shell
_SILENT_JAVA_OPTIONS="$_JAVA_OPTIONS"
unset _JAVA_OPTIONS
alias java='java "$_SILENT_JAVA_OPTIONS"'
``` 

