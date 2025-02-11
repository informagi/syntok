======
syntok
======

(a.k.a. segtok_ v2)

.. image:: https://img.shields.io/pypi/v/syntok.svg
    :target: https://pypi.python.org/pypi/syntok

.. image:: https://travis-ci.org/fnl/syntok.svg?branch=master
    :target: https://travis-ci.org/fnl/syntok

-------------------------------------------
Sentence segmentation and word tokenization
-------------------------------------------

The syntok package provides two modules, ``syntok.segmenter`` and ``syntok.tokenizer``.
The tokenizer provides functionality for splitting (Indo-European) text into words and symbols (collectively called *tokens*).
The segmenter provides functionality for splitting (Indo-European) token streams (from the tokenizer) into sentences and for pre-processing documents by splitting them into paragraphs.
Both modules can also be used from the command-line to split either a given text file (argument) or by reading from STDIN.
While other Indo-European languages could work, it has only been designed with the languages Spanish, English, and German in mind (the author's main languages).

``segtok``
==========

Syntok is the successor of an earlier, very similar tool, segtok_, but has evolved significantly in terms of providing better segmentation and tokenization performance and throughput (syntok can segment documents at a rate of about 100k tokens per second without problems).
For example, if a sentence terminal marker is not followed by a spacing character, segtok is unable to detect that as a terminal marker, while syntok has no problem segmenting that case (as it uses tokenization first, and does segmentation afterwards).
In fact, I feel confident enough to just boldly claim syntok is the world's best sentence segmenter for at least English, Spanish, and German.

Install
=======

To use this package, you minimally should have Python 3.5 or installed.
As it uses the typing package, earlier versions are not supported.
The easiest way to get ``syntok`` installed is using ``pip`` or any other package manager that works with PyPI::

    pip3 install syntok

*Important*: If you are on a Linux machine and have problems installing the ``regex`` dependency of ``syntok``, make sure you have the ``python-dev`` and/or ``python3-dev`` packages installed to get the necessary headers to compile that package.

Then try the command line tools on some plain-text files (e.g., this README) to see if ``syntok`` meets your needs::

    python3 -m syntok.segmenter README.rst
    python3 -m syntok.tokenizer README.rst

Test Suite
==========

To run the test suite, you have to have flake8, pytest, and mypy installed (``pip3 install flake8 pytest mypy``).

The testing environment works by running ``make`` targets (i.e., you need GNU Make or something equivalent around) or have to call the three commands by hand::

   make check

   # OR
   flake8 syntok  # make lint
   mypy syntok    # make type
   pytest syntok  # make test

Usage
=====

For details, please refer to the code documentation; This README only provides an overview of the provided functionality.

Command-line
------------

After installing the package, two command-line usages will be available, ``python -m syntok.segmenter`` and ``python -m syntok.tokenizer``.
Each takes [UTF-8 encoded] plain-text files (or STDIN) as input and transforms that into newline-separated sentences or space-separated tokens, respectively.
You can control Python3's file ``open`` encoding by `configuring the environment variable`_ ``PYTHONIOENCODING`` to your needs (e.g. ``export PYTHONIOENCODING="utf-16-be"``).
The tokenizer produces single-space separated tokens for each input line.
The segmenter produces line-segmented sentences for each input file (or after STDIN closes).

``syntok.tokenizer``
--------------------

This module provides the ``Tokenizer`` class to tokenize input text into words and symbols (**value** Tokens), prefixed with (possibly empty) **spacing** strings, while recording their **offset** positions.
The Tokenizer comes with utility static functions, to join hyphenated words across line-breaks, and to reproduce the original string from a sequence of tokens.
The Tokenizer considers camelCase words as individual tokens (here: camel and Case) and by default considers underscores and Unicode hyphens *inside* words as spacing characters (not Token values).
It does not split numeric tokens (without letters) if they contain symbols (e.g. maintaining "2018-11-11", "12:30:21", "1_000_000", "1,000.00", or "1..3" all as single tokens)
Finally, as it splits English negation contractions (such as "don't") into their root and "not" (here: do and not), it can be configured to refrain from replacing this special "n't" token with "not", and instead emit the actual "n't" value.

To track the spacing and offset of tokens, the module contains the ``Token`` class, which is a ``str`` wrapper class where the token **value** itself is available from the ``value`` property and adding a ``spacing`` and a ``offset`` property that will hold the **spacing** prefix and the **offset** position of the token, respectively.

Basic example::

   from syntok.tokenizer import Tokenizer

   document = open('README.rst').read()
   tok = Tokenizer()  # optional: keep "n't" contractions and "-", "_" inside words as tokens

   for token in tok.tokenize(document):
       print(repr(token))

``syntok.segmenter``
--------------------

This module provides several functions to segment documents into iterators over paragraphs, sentences, and tokens (functions ``analyze`` and ``process``) or simply sentences and tokens (functions ``split`` and ``segment``).
The analytic segmenter can even keep track of the original offset of each token in the document while processing (but does not join hyphen-separated words across line-breaks).
All segmenter functions accept arbitrary Token streams as input (typically as generated by the ``Tokenizer.tokenize`` method).
Due to how ``syntok.tokenizer.Token`` objects "work", it is possible to establish the exact sentence content (with the original spacing between the tokens).
The pre-processing functions and paragraph-based segmentation splits paragraphs, i.e., chunks of text separated by at least two consecutive linebreaks (``\\r?\\n``).

Basic example::

   import syntok.segmenter as segmenter

   document = open('README.rst').read()

   # choose the segmentation function you need/prefer

   for paragraph in segmenter.process(document):
       for sentence in paragraph:
           for token in sentence:
               # roughly reproduce the input,
               # except for hyphenated word-breaks
               # and replacing "n't" contractions with "not",
               # separating tokens by single spaces
               print(token.value, end=' ')
           print()  # print one sentence per line
       print()  # separate paragraphs with newlines

   for paragraph in segmenter.analyze(document):
       for sentence in paragraph:
           for token in sentence:
               # exactly reproduce the input
               # and do not remove "imperfections"
               print(token.spacing, token.value, sep='', end='')
       print("\n")  # reinsert paragraph separators

Legal
=====

License: `MIT <http://opensource.org/licenses/MIT>`_

Copyright (c) 2017-2021, Florian Leitner. All rights reserved.

Contributors
============

- Koen Dercksen, @KDercksen, https://koendercksen.com/  
- Sergiusz Bleja, @svenski 

Thank you!

History
=======

- **1.3.2** bugfix for offset of not contractions; discussion in Issue `#15`_
- **1.3.1** segmenting now occurs at semi-colons, too; discussion in Issue `#9`_
- **1.2.2** bugfix for offsets in multi-nonword prefix tokens; Issue `#6`_
- **1.2.1** added a generic rule for catching more uncommon uses of "." without space suffix as abbreviation marker
- **1.2.0** added support for skipping and handling text in brackets (e.g., citations)
- **1.1.1** fixed non-trivial segmentation in sci. text and refactored splitting logic to one place only
- **1.1.0** added support for ellipses (back - from segtok) in
- **1.0.2** hyphen joining only should happen when letters are present; squash escape warnings
- **1.0.1** fixing segmenter.analyze to preserve "n't" contractions, and improved the README and Tokenizer constructor API
- **1.0.0** initial release

.. _segtok: https://github.com/fnl/segtok
.. _configuring the environment variable: https://docs.python.org/3/using/cmdline.html
.. _#6: https://github.com/fnl/syntok/issues/6
.. _#9: https://github.com/fnl/syntok/issues/9
.. _#15: https://github.com/fnl/syntok/issues/15
