# Sublundo [![Build Status](https://travis-ci.org/libundo/Sublundo.svg?branch=master)](https://travis-ci.org/libundo/Sublundo) [![Build status](https://ci.appveyor.com/api/projects/status/2hs94fgrhds5dh5a?svg=true)](https://ci.appveyor.com/project/libundo/sublundo) [![PEP8](https://img.shields.io/badge/code%20style-pep8-orange.svg)](https://www.python.org/dev/peps/pep-0008/) [![Sublime Text version](https://img.shields.io/badge/sublime%20text-v3103%2B-blue.svg)](https://www.sublimetext.com/3) [![codebeat badge](https://codebeat.co/badges/fdbbebac-f1fd-411d-adee-4cc00cc55412)](https://codebeat.co/projects/github-com-libundo-sublundo-master) [![Package Control](https://img.shields.io/packagecontrol/dt/Sublundo.svg)](https://packagecontrol.io/packages/Sublundo)

<p align="center">
  <img alt="sublundo" src="https://user-images.githubusercontent.com/8785025/29885633-627d89ae-8d6c-11e7-9886-ccd389d18317.png">
</p>

Sublundo brings Vim-like persistent, branching undo/redo to Sublime Text 3. It was inspired by [Gundo](https://sjl.bitbucket.io/gundo.vim/) (and its successor, [Mundo](http://simnalamburt.github.io/vim-mundo/dist/)).

However, since Sublime Text doesn't have native support for branching undo like Vim, we had to build our own data structure&mdash;the [`UndoTree`](https://github.com/libundo/Sublundo/blob/master/lib/tree.py#L41). An `UndoTree` is an [N-ary tree](https://en.wikipedia.org/wiki/K-ary_tree) containing nodes that represent a particular buffer state:

<p align="center">
  <img width="320" alt="tree" src="https://user-images.githubusercontent.com/8785025/29885744-baac1096-8d6c-11e7-8105-f17b3bbe20d0.png">
</p>

Each [node](https://github.com/libundo/Sublundo/blob/master/lib/tree.py#L12) contains, among other attributes, a map associating node IDs to [patches](https://en.wikipedia.org/wiki/Patch_(Unix)). This means that, instead of having to store the entire buffer for each insertion (which often consists of small changes), we only need to store the information necessary to travel back and forth (in both the `parent → children` and `child → parent` directions). For example: if `A = 'Hello, world!'` and `B = 'Bye, world!'`, the `A → B` translation would be `[(-1, 'H'), (1, 'By'), (0, 'e'), (-1, 'llo'), (0, ', wo')]`. In Python terms, we'd have:

```python
>>> t = UndoTree()
>>> t.insert('Hello, world!')
>>> t.insert('Bye, world!')
>>> t.text()
'Bye, world!'
>>> t.undo()
# (buffer, patch, cursor position)
('Hello, world!', '@@ -1,7 +1,9 @@\n+H\n-By\n e\n+llo\n , wo\n', None)
>>> t.text()
'Hello, world!'
```

## Installation

1. Install [Package Control][pck-ctrl].
3. Bring up the Command Palette
   (<kbd>Command-Shift-P</kbd> on macOS and <kbd>Ctrl-Shift-P</kbd> on Linux/Windows).
4. Select `Package Control: Install Package`
   and then select `Sublundo` when the list appears.

## Usage

This package completely overrides the built-in `undo` and `redo` commands: whenever you undo or redo an edit, the `sublundo` command is run instead. So, you should be able to edit, undo, and redo text as you normally would.

When you want to either visualize or navigate the `UndoTree`, you invoke the `Sublundo: Visualize` command and then use the following keys to move around:

- <kbd>up</kbd> (or <kbd>k</kbd>): Move up the current branch (i.e., invoke `redo`).
- <kbd>down</kbd> (or <kbd>j</kbd>): Move down the current branch (i.e., invoke `undo`).
- <kbd>left</kbd> (or <kbd>h</kbd>): Move to the next branch on the left.
- <kbd>right</kbd> (or <kbd>l</kbd>): Move to the next branch on the right.

For information on the available settings, see the [default settings file](https://github.com/libundo/Sublundo/blob/master/Sublundo.sublime-settings).

[pck-ctrl]: https://packagecontrol.io/installation "Package Control"
