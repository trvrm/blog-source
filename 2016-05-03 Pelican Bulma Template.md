Title: New Blog Theme
Category: Software
Tags: Python, Pelican, Bulma
Author: Trevor
Date: 2016-05-13
 
 
Yesterday I discovered the [Bulma](http://bulma.io/) CSS library.  It seems to be basically 'bootstrap for the flexbox world.'

Given that Bootstrap version 4 has been promising us Flexbox support for nearly a year now, I think Bulma could be my
new best CSS friend.  Of course, I won't be able to use it anywhere where I have to support even reasonably old
browsers, but so far it's been very pleasant to work with.

I also used this opportunity to learn how to [create themes for Pelican](http://docs.getpelican.com/en/3.1.1/themes.html).  I basically took the 'simple' theme from the Pelican distribution and systematically rewrote each template
to use Bulma classes.  Here's an example from the `article.html` template

    <section class="section">
        <div class="container">
            <p class="subtitle is-4">
                {{ article.locale_date }}
            </p>
            
            <h2 class="title is-2">
                <a href="{{ SITEURL }}/{{ article.url }}" rel="bookmark" title="Permalink to {{ article.title|striptags }}">
                    {{ article.title }}
                </a>
            </h2>
            ...
    