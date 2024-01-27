+++
title = 'Vim macros tip: write it out and yank to register'
author = 'by James Lingford'
date = 2024-01-24T19:31:17+11:00
draft = false
toc = true
math = true
+++

## Vim macros refresher

Vim macros (also known as vim recording) are an efficient way to carry out repetitive edits on a text file and are one of the most powerful features of Vim.
Simply record the exact sequence of keystrokes/edits on one part of your text, and then play it on all other parts of your text to make the same edits instantly.
Recording a macro starts with simply hitting `q` followed by any alphabetical lower case letter:
```python
# start recording a macro
q<letter>

# make you edits...

# stop recording the macro
q

# play the macro on any part of your text
@<letter>
```

So for example, here's a simple macro to add checkboxes to a line:
```python
qq
I-[ ]: ^[j
q
```
Will start recording into register "`q`", will go into insert mode at the beggining of the line the cursor is on, will write the checklist boxes `- [ ]: ` while in insert mode,
will then escape to normal mode (the ESC key is represented as `^[` in Vim), then move down one line.

This macro can be played on 12 consecutive lines like so:
```python
12@q
```
Alternatively, it can be played on specific individual lines of your choosing by first hitting `@q` on a line of interest, then repeating that command on other lines by hitting `.`

## Using macros more efficiently

The thing with macros is that they are just keystrokes saved to one of Vims letter registers (type `:reg` to see what is already saved in your Vims registers).
This opens up a lot of potential for using macros better.
A lot of Vim users will try to record a macro "live" and, when playing it back, it doesn't work the way they had intended, record it live all over again and hope to not make the same mistake the first time.
This is innefficient and error prone (and most importantly, annoying).
What one should do instead is:

1. record your macro
2. if it doesn't work, paste it from the register into the text file
3. edit the pasted text directly to fix the error
4. yank that new text back into lettered register

So for example, say you're recording a macro into register "`a`":
```python
# move to start of line, move 2 words right, delete this third word, move down one line
qa
02wdwj
q
```
But you realise this doesn't behave as consistently as you like it, and you want to change it to delete "around" a word with `daw` instead of `dw`.
Edit it like so:

```python
# paste the macro recording
"ap

# make your changes

# yank new macro instructions into the register
"ayy

# play edited macro
@a
```

## Use Ctrl-v to insert useful commands while in insert mode

We can be even more efficient by writing out all our macros first instead of recording them live.
Though we will likely need to represent certain keystrokes as plain text, such as the all important ESC key to exit from insert mode.
To do this, while in insert mode simply type `Ctrl-v` followed by hitting `ESC`.

Examples:

```python
# <C-v>ESC results in the following text:
^[

# <C-v> and hitting the ENTER key results in:
^M

# Pasting from a register while in insert mode:
# <C-v> followed by <C-r> and paste contents from register ID number 1:
^R1

# we can even write a vim macro of a command-mode instruction:
:s/foo/bar/g^M
```

Happy Vimming
