---
layout: post
title: "Emacs cl-lib madness"
date: 2013-05-27 15:52
comments: true
categories: programming emacs emacs-lisp
---

Emacs 24.3 renamed the Common Lisp emulation package from `cl` to `cl-lib`. The
[release notes](http://www.gnu.org/software/emacs/NEWS.24.3) say that `cl` in
24.3 is now "a bunch of aliases that provide the old, non-prefixed names", but I
encountered some problems with certain packages searching for--as best I can
determine--function names that at some point changed but were not kept around as
aliases. This was particularly problematic when trying to run 24.3 on OS X
10.6.8.

In case anyone else runs into this problem, here's my solution:

``` scheme
;; Require Common Lisp. (cl in <=24.2, cl-lib in >=24.3.)
(if (require 'cl-lib nil t)
  (progn
    (defalias 'cl-block-wrapper 'identity)
    (defalias 'member* 'cl-member)
    (defalias 'adjoin 'cl-adjoin))
  ;; Else we're on an older version so require cl.
  (require 'cl))
```

We try to require `cl-lib`, and when that succeeds, define some aliases so that
packages don't complain about missing `cl-block-wrapper`, `member*`, and
`adjoin`. If it doesn't succeed, we're on an older Emacs, so require the old
`cl`.
