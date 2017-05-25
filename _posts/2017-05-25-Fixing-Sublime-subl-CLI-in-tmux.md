---
title: "Fixing Sublime (subl) CLI in tmux"
date: 2017-05-25
layout: "post"
categories: sublime, tmux, byobu
---

This fix is borrowed from [here](https://github.com/SublimeTextIssues/Core/issues/675), with some minor adjustments.

```
brew install reattach-to-user-namespace
echo "set-option -g default-command "reattach-to-user-namespace -l ${SHELL}" \
    | tee -a ~/.byobu/.tmux.conf
byoby kill-server
```

Then restart your sessions as normal.