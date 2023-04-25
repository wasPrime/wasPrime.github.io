---
title: Visual Studio freezing (crashing) while loading solution
date: 2023-04-25 02:27:53
categories:
- [configuration, environment]
tags:
- configuration
- environment
- visual_studio
---

## Problem

When opening a solution, Visual Studio is not responding after displaying "reading the file...".

## Solution

1. Close the Visual Studio process.
2. Remove the `.vs` folder in the folder where the solution file in.
3. Try to reopen the solution file.

## References

- [Stackoverflow - Visual Studio freezing (crashing) while loading solution](https://stackoverflow.com/questions/39703475/visual-studio-freezing-crashing-while-loading-solution)
