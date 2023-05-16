---
layout: post
title:  "Markdown Guide"
date:   2023-05-16
author: Tobashi
excerpt: This post is a brief cheatsheet of markdown capabilities.
categories: Other
---

* ToC
{:toc}

_This [post]( {{site.baseurl }}{% post_url 2023-05-16-markdown-guide %}) contains a cheatsheet with a few capabilities of markdown._

# Topic
This is a section for a new topic.

## Sub Topic
This is a section for a new sub topic with a source or addendum[^src].

[^src]: This is for a source/addendum <acronym title="Exempli gratia">e.g.</acronym> references or additional information.

# Code Snippets
Here is a simple example of a code snippet with and without `c` highlights:
``` c
#include <stdio.h>

// main function
int main()
{
    // prints hello world
    printf("Hello World");
  
    return 0;
}
```
Which when executed prints the following:
```
$ ./helloworld.o
$ Hello World
```

# Tables
Here is a simple example of a table covering the following list:
1. Item 1
    * Text.
2. Item 2
    * Text.
3. Item 3
    - **Test.**

|--------------------------------------+-----------+
| List                                 | Text      |
|:-------------------------------------|:----------|
| …                                    | …         |
| Item 1                               | Text.     |
| Item 2                               | Text.     |
| Item 3                               | **Test.** |
| …                                    | …         |
|--------------------------------------+-----------+
{: .decorated-table }

{% comment %} vim: set syntax=markdown spell tw=79 fo+=a: {% endcomment %}
