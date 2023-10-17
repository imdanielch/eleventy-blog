---
templateKey: blog-post
status: published
title: python imports
date: 2017-01-09T20:13:59.817Z
featuredpost: false
featuredimagealt:
featuredimage:
description:
layout: layouts/post.njk
tags:
---

python imports are weird in the sense that you can't just do `import "../../x.py"`.

Depending on your file structure, things can get a bit complicated. Here's my fix.

(based on this http://stackoverflow.com/questions/8951255/import-script-from-a-parent-directory)

```
file structure:

Project_Parent/
├── moduleA.py
└── Application/
|   ├── app.py
|   └── Module/
|       ├── __init__.py
|       └── moduleA.py
└── Tests/
    ├── __init__.py
    └── test_moduleA.py
```

In this scenario, `Tests` is my unit test directory and I want `test_moduleA.py` to import `moduleA.py`

`Application` directory is the parent of the project so we need to add it to `sys.path` for python to be able to find it.

```python
import sys
sys.path.insert(0, os.path.join('Project_Parent', 'Application'))
from Module.moduleA import function_in_moduleA
```
