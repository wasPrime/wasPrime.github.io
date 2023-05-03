---
title: Configure clang-format
date: 2023-05-04 01:41:30
categories:
- [configuration, clang]
tags:
- configuration
- clang
- clang-format
---

A `.clang-format` configuration template I prefer to use.

```yaml
# .clang-format
# Run manually to reformat a file:
# clang-format -i --style=file <file>
# find . -iname '*.cc' -o -iname '*.h' -o -iname '*.h.in' | xargs clang-format -i --style=file

Language: Cpp
BasedOnStyle: Google
# The offset of Access specifier(public/protected/private)
AccessModifierOffset: -4
AllowShortBlocksOnASingleLine: false
AllowShortFunctionsOnASingleLine: false
AllowShortLambdasOnASingleLine: false
ColumnLimit: 120
DerivePointerAlignment: false
IndentWidth: 4
```
