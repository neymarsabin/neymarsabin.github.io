+++
layout = 'posts'
title = 'TermStock'
date = 2024-08-19T12:00:00+05:45
draft = false
+++

<!--more-->

Check prices of Nepse symbols/stocks in the terminal. Quick and easy way to know how's your stock doing today.

#### how it works?
The project uses merolagani(credit to them) to fetch latest data and display it. The symbols you add are stored in a SQLite database and everytime you refresh or restart the application symbols data gets updated.

#### running the project
Build and run the repo using golang. 
    
    go run main.go

A menu is available with shortcut keys when pressed will present you the menu. For example to add a new symbol use the 'a' shortcut and enter your symbol. If the symbol is available data fetch works in the next update.

#### Next Steps:
- [ ] build a boxed dashboard to show the symbols and their performance.
- [ ] build a chatbox system to interact in groups, people who share the env key will have access to the group and the groups will be anonymous
