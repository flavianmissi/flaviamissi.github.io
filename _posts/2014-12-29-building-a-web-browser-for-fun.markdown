---
layout: post
title:  "Building a web browser for fun"
date:   2014-12-29
categories: c++ libcurl
---


First things first, let's list some basic browsing features:

    - get content of webpages given a url using HTTP or HTTPS
    - interpret front-end stack, html, css, javascript..
    - is that all?


We're going to do this in C++ so things get a little more exciting. Let's have a look
at C++'s http[s] built-in lib: a quick google search showed lots of opensource implementations
of HTTP clients to C++, but a lot of people also seem to like to simply use libcURL for the job.
That'll be our choice for now.

##Loading an url with libcURL

There are some details I will not include here, but you can check them in the source code later, or
read the [libcurl tutorial](http://curl.haxx.se/libcurl/c/libcurl-tutorial.html) yourself.

Perform a request:

```c++
curl_global_init(CURL_GLOBAL_ALL);

CURL *handle = curl_easy_init();

    curl_easy_setopt(handle, CURLOPT_URL, "http://flaviamissi.com.br");
    curl_easy_setopt(handle, CURLOPT_FOLLOWLOCATION, 1L);

    CURLcode resp = curl_easy_perform(handle);
    if(resp != CURLE_OK)
        std::cerr << "curl_easy_perform() failed: " << curl_easy_strerror(resp) << std::endl;

curl_global_cleanup();
```

You can check the complete code on my [github repository](https://github.com/flaviamissi/webbrowser-experiment/blob/cc7b2e5a0bea22157707a0636af38a45bac1f999/main.cpp).

I refactored the code a bit on master, so its prettier to perform a request.

The next step is to render the requested page's html.

##Rendering the html

I looked that up a bit online, only to find great libraries that could a LOT for me, that's why I
decided to approach the problem in a really simplistic way: we translate the HTML to ANSI escape
sequences. This is far from an ideal approach, but I chose this approach beucase I wanted to deal myself
with the requests made to the browser, which is not possible given all the available tools I found, for example,
the simplest tool I found to do the job was [KHTML](http://en.wikipedia.org/wiki/KHTML), which has been forked by
Google and Apple to build their own engines.

I could've also used [Apple WebKit](http://en.wikipedia.org/wiki/WebKit) (Safari) or
[Blink](http://en.wikipedia.org/wiki/Blink_(layout_engine)) (Google, Opera, etc), but they also solve all of my
problems, giving my almost nothing else to do.

###Translating the HTML to ANSI escape sequences

This is the hardest part of the job. A simple search into google returns various results of people asking how to do
the opposite, I'm going to use my own approach here.
Starting simple, let's colour all links to blue:

```C++
std::string blue = "\33\[34m";
std::size_t pos = html.find("<a>");
if (pos != std::string::npos) {
    html.replace(pos, 3, blue);
    pos = html.find("</a>");
    html.replace(pos, 4, "");
}
```

This is a very simple case that just works when the tag has no attributes, not very useful huh?
We need to generalize this, so let's create a map to keep all tags and its closing tags too.

```C++
std::map<std::string,std::string> html_tags = {
    {"<!DOCTYPE", ""},
    {"<!doctype", ""},
    {"<html", "</html>"},
    {"<head", "</head>"},
    {"<title", "</title>"},
    {"<meta", "</meta>"},
    {"<body", "</body>"},
    {"<h1", "</h1>"},
    {"<h2", "</h2>"},
    {"<center", "</center>"},
    {"<a", "</a>"},
    {"<p", "</p>"},
    {"<ol", "</ol>"},
    {"<ul", "</ul>"},
    {"<li", "</li>"},
    {"<table", "</table>"},
    {"<td", "</td>"},
    {"<tr", "</tr>"},
    {"<img", "</img>"},
    {"<svg", "</svg>"},
    {"<div", "</div>"},
    {"<form", "</form>"},
    {"<input", ""},
    {"<nav", "</nav>"},
    {"<header", "</header>"},
    {"<footer", "</footer>"},
    {"<span", "</span>"},
    {"<strong", "</strong>"},
    {"<em", "</em>"},
    {"<link", ""},
    {"<br", ""}
};
```

Now we need to know what to replace those tags with, let's use another map for that:

```C++
std::map<std::string,std::string> conversion_map = {
    {"<title", "\33\[37m\n"},
    {"<a", "\33\[34m"},
    {"<header", ""},
    {"<footer", ""},
    {"<div", "\033\[37m\n"},
    {"<span", " "},
    {"<li", " | "},
    {"<p", "\n"},
    {"<strong", "\033\[1m"},
    {"<em", "\033\[7m"},
    {"<h1", ""},
    {"<h2", ""},
    {"<ul", ""},
    {"<li", ""},
    {"<svg", ""},
    {"<link", ""},
    {"<br", ""}
};
```

It's worth noting that this is C++11 syntax, make sure you're compiling your file with support to it.

Now to the task:

```C++
std::string HTMLToANSI(std::string html) {
    std::map<std::string,std::string>::iterator it;
    std::size_t pos;
    std::size_t close_pos;
    std::string close_entity = ">";
    int shift = close_entity.size();
    std::string opening_tag;
    std::string closing_tag;

    html = removeTagsContent(html);

    for(it = html_tags.begin(); it != html_tags.end(); it++) {
        opening_tag = it->first;
        while ((pos = html.find(opening_tag)) != std::string::npos) {
            close_pos = html.find(close_entity, pos);

            if (close_pos != std::string::npos)
                html.replace(pos, (close_pos - pos) + shift, conversion_map[opening_tag]);
            else
                html.replace(pos, opening_tag.size() + close_entity.size(), conversion_map[opening_tag]);

            closing_tag = it->second;
            pos = html.find(closing_tag);

            if (pos == std::string::npos)
                continue;

            html.replace(pos, closing_tag.size(), "");
        }
    }

    return improveFormatting(html);
}
```


Now that we have replaced the whole tags with our escape sequences, we need to remove contents of certain tags,
like `<style>` and `<script>`.
So before the program replaces the HTML tags we must remove the contents of those tags:

```C++
std::string removeTagsContent(std::string html) {
    std::map<std::string,std::string>::iterator it;
    std::string opening_tag;
    std::string closing_tag;
    std::size_t pos;
    std::size_t close_pos;

    for(it = no_contents_map.begin(); it != no_contents_map.end(); it++) {
        opening_tag = it->first;
        closing_tag = it->second;
        while((pos = html.find(opening_tag)) != std::string::npos) {
            close_pos = html.find(closing_tag, pos);
            html.replace(pos, (close_pos - pos) + closing_tag.size(), "");
        }
    }
    return html;
}
```

After replacing everything, there're lots of new lines and useless whitespaces as leftovers from the formatting,
to fix that I built a simple function:

```C++
std::string improveFormatting(std::string html) {
    std::size_t pos;
    std::string token = "  ";
    while ((pos = html.find(token)) != std::string::npos) {
        html.replace(pos, token.size(), " ");
    }

    token = "\n \n";
    while ((pos = html.find(token)) != std::string::npos) {
        html.erase(pos, token.size());
    }

    token = "\n\n ";
    while ((pos = html.find(token)) != std::string::npos) {
        html.erase(pos, token.size());
    }

    return html;
}
```

Now that everything has been dealt with, to run the program from [my repository](https://github.com/flaviamissi/webbrowser-experiment):

```shell
$ make run url=<your-url.com>
```

Take a look at the code and comment on github if there's anything else you need to know.
