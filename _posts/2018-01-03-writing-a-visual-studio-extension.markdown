---
layout: post
title:  "Writing a Visual Studio Extension"
date:   2018-01-03 20:02:15 +0000
categories: jekyll update
---

The git integration within Visual Studio has its critics, but I find it generally does most of the
things I need it to do in my day to day work. However, the lack of an ability to prune has been something
of an inconvenience. I know that I can configure git to prune on fetch, but I saw this as an opportunity
to experiment with building a basic extension to Visual Studio.

# Creating the project

![New project]({{ "../assets/vs-new-vsix-project.png" | absolute_url }})

First things first, we need to create a new extension project. This can be done by clicking the new project
button, choosing 'Extensibility' as the project type, and selecting 'VSIX Project'. This will create a new
project using a template that brings in all things necessary to get started on our extension.