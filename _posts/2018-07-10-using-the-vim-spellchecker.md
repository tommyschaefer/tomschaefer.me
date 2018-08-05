---
layout: post
title: Using Vim's Built-In Spell Checker
summary: When writing prose, Vim's built-in spell checker can help you out if you know how to use it
author: Tom Schaefer
author_url: http://teecom.com/people/tommy-schaefer/
tags: vim
---

I often find myself writing prose using Vim. Whether it's for project
documentation, a blog post, or commit messages, I always rely on a spell checker
to catch my horrible, horrible spelling errors. I'm a **terrible** speller.

I knew how to enable Vim's spell checker to see where my errors were, but until
recently I didn't really know how to use it. Whenever I saw a spelling mistake,
I would try three or four guesses before getting frustrated, giving up, and
going to Google for help.

This week, I decided enough was enough and spent some time learning how to
really use Vim's built-in spell checker.

## Enabling Spell Check

Vim's spell checker can be enabled a few different ways. To switch it on for the
current file, you can run:

```
:setlocal spell
```

It would be tedious to turn on spell checking for each individual file, though.

Thankfully Vim configurations can be set for an entire file type. I most
frequently require spell checking when editing markdown documents or commit
messages. Because of this, I have the following configuration in my `~/.vimrc`:

```
" Enable spell checking for markdown files
autocmd Filetype markdown setlocal spell

" Enable spell checking for git commit messsages
autocmd Filetype gitcommit setlocal spell
```

## Using Spell Check

Now that spell checking is enabled, it's time to learn how to use it.

#### Navigating the sea of misspelled words

When I'm working on a document, I usually like to get all of my thoughts out
before going back and editing or fixing spelling errors. When I get to this
step, I usually have **a lot** of places that need fixing. Because of this, I
find it really helpful to jump between misspelled words with `]s` (next word)
and `[s` (previous word).

#### Adding unrecognized words to the dictionary

Often times when I'm working on a blog post or bit of markdown documentation, my
text will include words that the spell checker doesn't recognize. For example,
when working on my last blog post, the word "dropdown" appeared a number of
times:

![](https://dl.dropboxusercontent.com/s/7za6zlif9aovrct/2018_07_10_dropdown_misspelled.png?dl=0)

To fix this, you can use `zg` to add the word under the cursor to Vim's
"spellfile".

#### Correcting spelling mistakes

To replace a misspelled word, you have a few options. If you're in normal mode,
you can use `z=` while your cursor is over a misspelled word to see a list of
possible corrections.

![](https://dl.dropboxusercontent.com/s/ab46oumgovpt0f8/2018_07_10_replace_misspelled.png?dl=0)

When in insert mode, you can press `Ctrl-X s` to see a list of suggestions and
`Ctrl-N` or `Ctrl-P` to navigate the list.

![](https://dl.dropboxusercontent.com/s/7ypwo2d966dvo4u/2018_07_10_spelling_suggestions.png?dl=0)

## What's Next?

To learn more about spell checking in vim, check out these resources:

- [:help spell](http://vimdoc.sourceforge.net/htmldoc/spell.html)
- [Using Spell Check in Vim](http://thejakeharding.com/tutorial/2012/06/13/using-spell-check-in-vim.html)
- [Vim Casts #19: Spell Checking](http://vimcasts.org/episodes/spell-checking/)
