---
layout: post
title: '"Unterminated nested statement!" using TYPO3 rector'
date: '2021-08-15T14:13:00.002+02:00'
author: Torben Hansen
tags:
- TYPO3
- rector
modified_time: '2021-08-15T20:04:45.130+02:00'
thumbnail: https://lh3.googleusercontent.com/-ju9KeqNJcpQ/YRDnd8I0cmI/AAAAAAAAu34/r5JJqEegUgM4HOGUuSk6CVG9yMIXgqqvACLcBGAsYHQ/s72-c/Bildschirmfoto%2B2021-08-09%2Bum%2B09.57.50.png
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-480113124188572608
permalink: /2021/08/unterminated-nested-statement-using-typo3-rector.html
---

[TYPO3 rector](https://github.com/sabbelasichon/typo3-rector) is a really helpful application when it comes to TYPO3
major updates. It helps you to identify and refactor TYPO3 deprecations in custom extensions and can save hours of
manual refactoring. I use TYPO3 rector quite a lot and stumbled across the following error recently.

!["Unterminated nested statement!"](/assets/images/2021-08-15/image1.png)

This message is not really helpful, so I digged deeper into the problem. The `Parser.php` throwing the exception is
located in `helmich/typo3-typoscript-parser` package, so I first thought that there was a problem with the TypoScript in
the desired extension, but after checking ever line manually, I could not find any error.

It came out, that the extension I wanted to process by TYPO3 rector had a `node_modules` folder, which contained a lot
of Typescript (not TypoScript) files. Those files where obviously parsed by the TypoScript parser resulting in the shown
error message. After removing (excluding should also work) the `node_modules` folder, everything worked as expected.

If you like rector and/or TYPO3 rector, please consider to support the authors.