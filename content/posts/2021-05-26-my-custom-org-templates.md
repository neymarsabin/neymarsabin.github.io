---
layout: post
title: Org Templates
type: blog
author: neymarsabin
tags: 
- org-mode
- emacs
---

These templates are already merged in [#pr-260](https://github.com/AndreaCrotti/yasnippet-snippets/pull/260)

# Using Org Templates
Using orgTemplates is very simple.Just clone this repository.Then copy all files inside orgTemplates to your emacs configuration folder excluding hidden files.

``
cd orgTemplatse
``

``
rsync -av --exclude=".*" . ~/.emacs.d/snippets/org-mode/
``

Then, tell emacs to reload all the snippets.For that do *M-x yas-reload-all* 

# Depends On
	Yasnippet = [yasnippet](https://github.com/joaotavora/yasnippet)
	Installing yasnippet is easy and instructions is available in it's github page.

# Keywords and Templates
	#+CAPTION: Keywords and names
	#+ATTR_HTML: :width 100%
	| Keywords | names         |   | Keywords | names       |
	|----------|---------------|---|----------|-------------|
	| <au      | Author        |   | <ex      | Export      |
	| <c       | center        |   | <ht      | html        |
	| <da      | date          |   | <im      | Images      |
	| desc     | Description   |   | <i       | Includes    |
	| <em      | Email         |   | <ke      | Keywords    |
	| <e       | Example block |   | <lan     | Language    |
	| <l       | Latex         |   | <li      | Links       |
	| <op      | Options       |   | <q       | Quote       |
	| set      | Setup         |   | <s       | Source Code |
	| <st      | Style         |   | <ta      | Table       |
	| <ti      | Title         |   | <v       | Verse       |
	| <vi      | Videos        |   |          |             |

	For Revealjs templates
	#+CAPTION: Keywords and names 
	#+ATTR_HTML: :width 100%
	| Keywords | names                            |
	|----------|----------------------------------|
	| <rvl     | Reveal all options               |
	| <rsb     | Reveal single colored background |
	| <rib     | Reveal image background          |



