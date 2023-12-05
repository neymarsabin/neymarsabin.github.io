---
layout: post
type: blog
author: neymarsabin
title: EMACS configuration
tags: 
- emacs
- linux
---

### package management
EMACS has its own package managers, which can read from sources like gnu, melpa repositories and install package for you. There are other package managers like use-package, cask, straight.el, but I like using cask. Cask let's you define package you want to install. It is best for running emacs in new setups at once. 

Installing Cask:

	curl -fsSL https://raw.githubusercontent.com/cask/cask/master/go | python


And in the emacs configuration folder like *~/.emacs.d* you can create an empty Cask file. My cask file contains these packages and source, its minimal but has sufficient packages for Rails/React projects.

	(source gnu)
	(source melpa)

	(depends-on "projectile")
	(depends-on "helm")
	(depends-on "helm-projectile")
	(depends-on "org")
	(depends-on "projectile-rails")
	(depends-on "magit")
	(depends-on "multiple-cursors")
	(depends-on "web-mode")
	(depends-on "cask-mode")
	(depends-on "all-the-icons")
	(depends-on "neotree")
	(depends-on "dumb-jump")
	(depends-on "ruby-mode")
	(depends-on "rjsx-mode")
	(depends-on "zenburn-theme")
	(depends-on "markdown-mode")

Then you just have to install all the packages with this command from the repository with *Cask* file was in. 
	
	cask install

There are other bunch of options for updates/upgrades of packages, compiling them, listing unused packages. Cask has help options as well.

	cask help
	

### Configuration

EMACS file configurations resides in *~/.emacs.d* folder with an *~/.emacs.d/init.el* as the main source of configuration for emacs startup. 

	;; setup cask for emacs project management
	(require 'cask "~/.cask/cask.el")
	(cask-initialize)

	;; really hate the title bar, menu options, scrollbar in emacs
	;; no use to me, removing these
	(when (window-system)
		(tool-bar-mode -1)
		(menu-bar-mode -1)
		(scroll-bar-mode -1))

	;; sync system clipboard with emacs <->
	(transient-mark-mode t)
	(setq x-select-enable-clipboard t)
	(setq x-select-enable-primary t)

	;; you would () want parenthesis at once, also hightlight parenthesis
	(electric-pair-mode)
	(show-paren-mode t)

	;; better wrapping mode for each line
	(global-visual-line-mode t)

	;; no need to make backup files
	(setq make-backup-files nil)

	;; also disable auto save
	(setq auto-save-default nil)

	;; default tab width
	(setq-default tab-width 2)

	;; enable auto indentation
	(electric-indent-mode t)

	;; really important for editing, multiple-cursors
	(require  'multiple-cursors)
	(global-set-key (kbd "C-S-c C-S-c") 'mc/edit-lines)
	(global-set-key (kbd "C-{") 'mc/mark-next-like-this)
	(global-set-key (kbd "C-}") 'mc/mark-previous-like-this)
	(global-set-key (kbd "C-c C-{") 'mc/mark-all-like-this)

	;; some important keybindings
	(global-set-key (kbd "RET") 'newline-and-indent)
	(global-set-key (kbd "C-;") 'comment-or-uncomment-region)
	(global-set-key (kbd "M-/") 'hippie-expand)
	(global-set-key (kbd "C-+") 'text-scale-increase)
	(global-set-key (kbd "C--") 'text-scale-decrease)
	(global-set-key (kbd "C-c C-k") 'compile)
	(global-set-key (kbd "C-x g") 'magit-status)

	;; really like dumb-jump for function/definitition navigation
	(global-set-key (kbd "M-g j") 'dumb-jump-go)
	(global-set-key (kbd "M-g p") 'dumb-jump-back)
	(global-set-key (kbd "M-g w") 'dumb-jump-go-other-window)

	;; projectile mode is the best for projects navigation, works really well with rails with projectile-rails
	(projectile-mode +1)
	(define-key projectile-mode-map (kbd "C-c p") 'projectile-command-map)
	(projectile-rails-global-mode)
	(define-key projectile-rails-mode-map (kbd "C-c r") 'projectile-rails-command-map)

	;; using helm for projects/files/everything navigation easily
	(require 'helm)
	(helm-mode 1)
	(global-set-key (kbd "M-x") 'helm-M-x)
	(global-set-key (kbd "C-x C-f") 'helm-find-files)
	(global-set-key (kbd "C-x b") 'helm-buffers-list)
	(require 'helm-projectile)
	(helm-projectile-on)
	
	;; this is for my one and only zenburn theme
	(custom-set-variables
	;; custom-set-variables was added by Custom.
	;; If you edit it by hand, you could mess it up, so be careful.
	;; Your init file should contain only one such instance.
	;; If there is more than one, they won't work right.
	'(custom-safe-themes
	(quote
	("f56eb33cd9f1e49c5df0080a3e8a292e83890a61a89bceeaa481a5f183e8e3ef" default))))
		(custom-set-faces
	;; custom-set-faces was added by Custom.
	;; If you edit it by hand, you could mess it up, so be careful.
	;; Your init file should contain only one such instance.
	;; If there is more than one, they won't work right.
	)
		;; set default theme
		(set-frame-font "Mono-16")

	;; some javascript configurations
	(setq js-indent-level 2)
	(setq js2-basic-offset 2)
	(setq javascript-indent-level 2)
	(setq css-indent-offset 2)
	(setq web-mode-markup-indent-offset 2)

	;; org-agenda-mode
	(global-set-key (kbd "C-c a") 'org-agenda-list)


latest configs: [dotfiles](https://github.com/neymarsabin/dotfiles_reloaded/blob/trail/dots/emacs/init.el)
