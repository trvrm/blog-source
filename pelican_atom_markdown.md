Title:Pelican, Atom and Markdown
Category: Software
Tags: pelican, markdown
Author: Trevor
Date: 2015-06-26

I've been using reStructured text in general to write this blog, but I think I'm
going to be switching to Markdown.  As an experiment, I'm writing this post in Markdown.

I'm also writing it in the [Atom text editor](https://atom.io/), which has really come on a
long way since I last tried it.  Specifically, it includes a Markdown preview function, so
I can see the effects of the markup that I'm writing as I write it.

Mostly, I want a rapid way of creating and publishing code snippets, without the mental
overhead of switching between markup languages. Although reStructured text and markdown
are broadly similar, there are subtle differences between them when it comes to things
like syntax highlighting.  But I've discovered today that if I use the triple-backtick
syntax, I can get the same output from Pelican, Atom, and IPython notebooks.

So

```markdown
    ```python
    def syntax(highlighting=True):
        return "cool huh?"
    ```
```

yields

```python
 def syntax(highlighting=True):
   return "cool huh?"
```

And if I paste that into a markdown cell in an IPython notebook, I get the same effect, as
can be seen [here](https://github.com/trvrm/notebooks/blob/master/Markdown%20Demo.ipynb)


So this seems to be the general way that the open-source ecosystem is going: Markdown
allows me to use the same syntax for my GitHub documentation, my IPython notebooks, and
my blog posts.  

I do use **Sphinx** in various places for Python code documentation, so that will still
require reStructured text, but elsewhere I think Markdown is the way to go
