---
layout: post
title: "Steps to install jekyll on windows"
description: "Jekyll installation instruction"
category: [jekyll]
tags: [jekyll, installation]
---

1. Install ruby
     * Download ruby installer and DevKit-mingw
        * Install ruby and set enviroment path
        * Extract Dev-kit-mingw, edit config.yaml, add ruby install path;
        * then open and cmd window,enter the following commands


	chdir --Path where you extract: D:/devkit--
	ruby dk.rb init
	ruby dk.rb install


2. Install jekyll
     * Enter the following code
 
     gem install jekyll -V
     jekyll new my-awesome-site
     cd my-awesome-site
     jekyll serve
 