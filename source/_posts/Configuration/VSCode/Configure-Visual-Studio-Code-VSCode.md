---
title: Configure Visual Studio Code (VSCode)
date: 2023-05-06 21:01:57
categories:
- [configuration, vscode]
tags:
- configuration
- vscode
- extension
---

List some extensions I get used to using.

## General

### Theme

- `One Dark Pro`: An awesome theme!
- `vscode-icons`: A set of good-looking icons for folders and files.

### Comments

- `Better Comments`: Highlight comments.
  
### Run code

- `Code Runner`: Run a single file on the spot conveniently with shortcut key `control + option + N` in MacOS.

    {% note info %}
    **How to edit its executor map:**
    1. Click the gear icon in the lower right corner
    2. Click `Settings - Open Settings (JSON)` icon
    3. Edit in `settings.json`
    {% endnote %}

    {% note info %}
    **An available executor map:**

    ```json
    {
        "code-runner.executorMap": {
            "javascript": "node",
            "java": "cd $dir && javac $fileName && java $fileNameWithoutExt",
            "c": "cd $dir && gcc $fileName -o $fileNameWithoutExt && $dir$fileNameWithoutExt",
            "cpp": "cd $dir && g++ -std=c++2a $fileName -o $fileNameWithoutExt && $dir$fileNameWithoutExt",
            "objective-c": "cd $dir && gcc -framework Cocoa $fileName -o $fileNameWithoutExt && $dir$fileNameWithoutExt",
            "php": "php",
            "python": "python3 -u",
            "perl": "perl",
            "perl6": "perl6",
            "ruby": "ruby",
            "go": "go run",
            "lua": "lua",
            "groovy": "groovy",
            "powershell": "powershell -ExecutionPolicy ByPass -File",
            "bat": "cmd /c",
            "shellscript": "bash",
            "fsharp": "fsi",
            "csharp": "scriptcs",
            "vbscript": "cscript //Nologo",
            "typescript": "ts-node",
            "coffeescript": "coffee",
            "scala": "scala",
            "swift": "swift",
            "julia": "julia",
            "crystal": "crystal",
            "ocaml": "ocaml",
            "r": "Rscript",
            "applescript": "osascript",
            "clojure": "lein exec",
            "haxe": "haxe --cwd $dirWithoutTrailingSlash --run $fileNameWithoutExt",
            "rust": "cd $dir && rustc $fileName && $dir$fileNameWithoutExt",
            "racket": "racket",
            "scheme": "csi -script",
            "ahk": "autohotkey",
            "autoit": "autoit3",
            "dart": "dart",
            "pascal": "cd $dir && fpc $fileName && $dir$fileNameWithoutExt",
            "d": "cd $dir && dmd $fileName && $dir$fileNameWithoutExt",
            "haskell": "runhaskell",
            "nim": "nim compile --verbosity:0 --hints:off --run",
            "lisp": "sbcl --script",
            "kit": "kitc --run",
            "v": "v run",
            "sass": "sass --style expanded",
            "scss": "scss --style expanded",
            "less": "cd $dir && lessc $fileName $fileNameWithoutExt.css",
            "FortranFreeForm": "cd $dir && gfortran $fileName -o $fileNameWithoutExt && $dir$fileNameWithoutExt",
            "fortran-modern": "cd $dir && gfortran $fileName -o $fileNameWithoutExt && $dir$fileNameWithoutExt",
            "fortran_fixed-form": "cd $dir && gfortran $fileName -o $fileNameWithoutExt && $dir$fileNameWithoutExt",
            "fortran": "cd $dir && gfortran $fileName -o $fileNameWithoutExt && $dir$fileNameWithoutExt",
            "sml": "cd $dir && sml $fileName"
        }
    }
    ```

    {% endnote %}

### Git

- `git-commit-plugin`: Type git commit with some templates and emojis.
- `GitLens — Git supercharged`: Reveiw git log graphically.

### Markdown

- `Markdown All in One`: Markdown format especially for filling in spaces in tables
- `markdownlint`: Markdown lint and format.
- `Markdown Preview Enhanced`: Markdown preview.

### Autocomplete

- `Tabnine`: An AI autocomplete tool whose full name is `Tabnine AI Autocomplete for Javascript, Python, Typescript, PHP, Go, Java, Ruby & more`.

## Containers

- `Dev Containers`: Create containers environment such as Linux for development quickly.

## Language

### C++

- `clangd`: Awesome clang server with clang-format and clang-tidy.
- `Cmake`
- `Cmake Tools`

### Python

- `Python`: IntelliSense (Pylance), Linting, Debugging (multi-threaded, remote), Jupyter Notebooks, code formatting, refactoring, unit tests, and more.
- `Pylance`: A performant, feature-rich language server for Python in VS Code
- `isort`: Sort imports automatically.
