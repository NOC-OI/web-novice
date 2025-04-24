---
title: "Elements of Web Scraping with BeautifulSoup"
teaching: 25
exercises: 15
questions:
- "How can I obtain data in a programmatic way from the web without an API?"
objectives:
- "Have an idea about how to navigate the HTML element tree with Beautiful Soup and extract relevant information."
keypoints:
- "A `BeautifulSoup` object can be navigated in many ways:" 
- "Use `find` to look for the first element that matches the given criteria in a subtree"
- "Use `find_all` to obtain a list of elements that matches the given criteria in a subtree"
- "Use `find_parents` to get the list of ancestor of the given element"
---

Sometimes, 
the data we are looking for is not available from an API,
but it is available on web pages that we can view with our browser.
As an example task,
in this episode we are going 
to use
the Beautiful Soup Python package
for web scraping
to find all the relevant information
about Software Carpentry lessons.

## Exploring HTML code in the browser

Navigate to [The Software Carpentry Lessons][software-carpentry-lessons].
The page we see has been rendered by the browser
from the HTML, CSS (Cascading Style Sheets) and JavaScript code
that is available or linked in the page in some way.

In many browsers 
(for example, Chrome, Chromium, and Firefox), 
we can look at the HTML source code
of the page we are viewing
with the `CTRL+u` shortcut
(alternatively, you can 
right click on the page 
and choose "View Source"
from the context menu).

Things to notice:
- HTML elements can be nested, 
  and form (approximately) a tree. 
- Most elements have an opening tag `<tagname>`, 
  and a corresponding closing one `</tagname>`.
  For examples, see the
  [reference on the Mozilla Developer Network][mdn-elements-reference]
- Each element can have __attributes__, 
  defined in the opening tag.

Another way to explore the HTML code
is to use the Developer Tools.
In most browser,
(Chrome, Chromium and Firefox), 
you can use the `CTRL+Shift+I` key combination 
to open the Developer Tools
(alternatively, find the right option
in your browser menu).

> ## Developer Tools in Safari
>
> In Safari on macOS, the Developer Tools are hidden by default. To enable them,
> open the Preferences window, go to the Advanced tab, and enable the "Show
> Develop menu in menu bar" option.
{: .callout}

By using these, 
by pressing the combination `CTRL+Shift+C`
(or clicking on the mouse pointer icon 
in the top left of the window)
you can hover with the mouse 
on the elements in the rendered page
and view their properties.
If you click on one of these,
the relevant part of the HTML code 
will be shown to you.

By using these techniques, 
we can understand how to locate the elements 
that we want 
when using Beautiful Soup 
later on.

## Relevant HTML tags for this lessons

There is a number of tags 
that may be interesting in general,
but specifically for what follows, 
we need to notice:
- the `<table>` tag, which starts a table,
  which is composed of rows;
- the `<tr>` tag, which starts a Table Row
  inside a `<table> ... </table>` element;
- the `<td>` tag, which starts a Table Data cell
  inside a `<tr> ... </tr>` element;
- the `<a>` flag, meaning an "anchor" object,
  used for __hyperlinks__, 
  usually in the form 
  `<a href="http://somewhere.com/an/u/r/i">`,
  i.e., with an `href` attribute.

## Scraping the page with Beautiful Soup

From the [BeautifulSoup documentaion][bs4-docs]:

> Beautiful Soup is a Python library for pulling data out of HTML and XML files.
> It works with your favorite parser to provide idiomatic ways of navigating,
> searching, and modifying the parse tree. It commonly saves programmers hours
> or days of work.
{: .quote}

First of all, let's verify that we have BeautifulSoup installed:
~~~
python -c "import bs4"
~~~
{: .language-bash}
If there is no output, then we are all set.
If instead you see something along the lines of 
~~~
Traceback (most recent call last):
  File "<string>", line 1, in <module>
ModuleNotFoundError: No module named 'bs4'
~~~
{: .output}
Then you have to install the package.
One way of doing that is via `pip`, 
with
~~~
pip install beautifulsoup4
~~~
{: .language-bash}

Once we are sure the BeautifulSoup is available,
we can import the necessary libraries in Python
and use `requests` to GET 
the Software Carpentries website content:

~~~
import requests
from bs4 import BeautifulSoup

response = requests.get("https://software-carpentry.org/lessons/")
response
~~~
{: .language-python}

~~~
<Response [200]>
~~~
{: .output}

So, the request was successful.
The HTML of the web page 
is in the `text` member of the response.
We can pass that directly 
the the `BeautifulSoup` constructor,
obtaining a soup object 
that we still need to navigate:

~~~
soup = BeautifulSoup(markup=response.text,
                     features="html.parser")
~~~
{: .language-python}

Looking at the HTML code,
we see that just above the first table 
there is the text "Core Lessons in English"
inside a `<h2>` tag (code reindented for clarity)

~~~
...
<h2 id=core-lessons-in-english>Core Lessons in English</h2>
<div class="table-striped overflow-x-auto">
    <table>
        <thead>
            <tr>
                <th>Lesson</th>
                <th>Site</th>
                <th>Repository</th>
                <th>Reference</th>
                <th>Instructor Notes</th>
                <th>Maintainers</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td>The Unix Shell</td>
                <td style=text-align:center><a href=https://swcarpentry.github.io/shell-novice /><i
                        class="fas fa-window-maximize"></i></a></td>
                <td style=text-align:center><a href=https://github.com/swcarpentry/shell-novice><i
                            class="fab fa-github"></i></a></td>
                <td style=text-align:center><a href=https://swcarpentry.github.io/shell-novice/reference><i
                            class="fas fa-eye"></i></a></td>
                <td style=text-align:center><a
                        href=https://swcarpentry.github.io/shell-novice/instructor/instructor-notes><i
                            class="fas fa-plus"></i></a></td>
                <td>Jacob Deppen, Benson Muite</td>
            </tr>
            <tr>
                <td>Version control with Git</td>
                <td style=text-align:center><a href=https://swcarpentry.github.io/git-novice><i
                            class="fas fa-window-maximize"></i></a></td>
                <td style=text-align:center><a href=https://github.com/swcarpentry/git-novice><i
                            class="fab fa-github"></i></a></td>
                <td style=text-align:center><a href=https://swcarpentry.github.io/git-novice/reference><i
                            class="fas fa-eye"></i></a></td>
                <td style=text-align:center><a
                        href=https://swcarpentry.github.io/git-novice/instructor/instructor-notes><i
                            class="fas fa-plus"></i></a></td>
                <td>Erin Graham, Katherine Koziar, Martino Sorbaro</td>
            </tr>
            <tr>
                <td>Programming with Python</td>
                <td style=text-align:center><a href=https://swcarpentry.github.io/python-novice-inflammation><i
                            class="fas fa-window-maximize"></i></a></td>
                <td style=text-align:center><a href=https://github.com/swcarpentry/python-novice-inflammation><i
                            class="fab fa-github"></i></a></td>
                <td style=text-align:center><a
                        href=https://swcarpentry.github.io/python-novice-inflammation/reference><i
                            class="fas fa-eye"></i></a></td>
                <td style=text-align:center><a
                        href=https://swcarpentry.github.io/python-novice-inflammation/instructor/instructor-notes><i
                            class="fas fa-plus"></i></a></td>
                <td>Indraneel Chakraborty, Toan Phung, Alberto Villagran</td>
            </tr>
            <tr>
                <td>Plotting and programming with Python</td>
                <td style=text-align:center><a href=https://swcarpentry.github.io/python-novice-gapminder><i
                            class="fas fa-window-maximize"></i></a></td>
                <td style=text-align:center><a href=https://github.com/swcarpentry/python-novice-gapminder><i
                            class="fab fa-github"></i></a></td>
                <td style=text-align:center><a href=https://swcarpentry.github.io/python-novice-gapminder/reference><i
                            class="fas fa-eye"></i></a></td>
                <td style=text-align:center><a
                        href=https://swcarpentry.github.io/python-novice-gapminder/instructor/instructor-notes><i
                            class="fas fa-plus"></i></a></td>
                <td>Allen Lee, Sourav Singh, Olav Vahtras</td>
            </tr>
            <tr>
                <td>Programming with R</td>
                <td style=text-align:center><a href=https://swcarpentry.github.io/r-novice-inflammation /><i
                        class="fas fa-window-maximize"></i></a></td>
                <td style=text-align:center><a href=https://github.com/swcarpentry/r-novice-inflammation><i
                            class="fab fa-github"></i></a></td>
                <td style=text-align:center><a href=https://swcarpentry.github.io/r-novice-inflammation/reference><i
                            class="fas fa-eye"></i></a></td>
                <td style=text-align:center><a
                        href=https://swcarpentry.github.io/r-novice-inflammation/instructor/instructor-notes><i
                            class="fas fa-plus"></i></a></td>
                <td>Rohit Goswami, Hugo Gruson, Isaac Jennings</td>
            </tr>
            <tr>
                <td>R for Reproducible Scientific Analysis</td>
                <td style=text-align:center><a href=https://swcarpentry.github.io/r-novice-gapminder><i
                            class="fas fa-window-maximize"></i></a></td>
                <td style=text-align:center><a href=https://github.com/swcarpentry/r-novice-gapminder><i
                            class="fab fa-github"></i></a></td>
                <td style=text-align:center><a href=https://swcarpentry.github.io/r-novice-gapminder/reference><i
                            class="fas fa-eye"></i></a></td>
                <td style=text-align:center><a
                        href=https://swcarpentry.github.io/r-novice-gapminder/instructor/instructor-notes><i
                            class="fas fa-plus"></i></a></td>
                <td>Matthieu Bruneaux, Sehrish Kanwal, Naupaka Zimmerman</td>
            </tr>
        </tbody>
    </table>
</div>
...
~~~
{: .language-html}

We can then look for the table 
by finding the HTML element 
that contains that text,
using the `string` keyword argument:

~~~
(soup.find(string="Core Lessons in English"))
~~~
{: .language-python}

~~~
'Core Lessons in English'
~~~
{: .output}

By using the `find` method on a `BeautifulSoup` object,
we look at all of its descendants and 
obtain other `BeautifulSoup` objects
that we can search 
in the same way as the original one.
But how do we get the parent element?
We can use the `find_parents()` method,
which returns a list of 
`BeautifulSoup` objects
that represents the ancestors in the tree
of the given element,
starting from the immediate parent 
of the element itself 
and ending with the element 
at the root of the tree
(`soup` in this case).
The second parent in the list
is the one that also contains 
the table we are interested in:
~~~
(soup
 .find(string = "Core Lessons in English")
 .find_parents()[1])
~~~
{: .language-python}

~~~
<div class="prose h2-wrap max-w-none">
    <p>A Software Carpentry workshop is taught by at least one trained and badged Instructor. Over the course of the
        workshop, Instructors teach our three core topics: the Unix shell, version control with Git, and a programming
        language (Python or R). Curricula for these lessons in English and Spanish (select lessons only) are below.</p>
    <p>You may also enjoy <a href="https://datacarpentry.org/lessons">Data Carpentry’s lessons</a> (which focus on data
        organisation, cleanup, analysis, and visualisation) and <a href="https://librarycarpentry.org/lessons">Library
            Carpentry’s lessons</a> (which apply concepts of software development and data science to library contexts).
    </p>
    <p>Please <a href="https://carpentries.org/contact">contact us</a> with any general questions.</p>
    <h2 id="core-lessons-in-english">Core Lessons in English</h2>
    <div class="table-striped overflow-x-auto">
        <table>
            <thead>
                <tr>
                    <th>Lesson</th>
                    <th>Site</th>
                    <th>Repository</th>
                    <th>Reference</th>
                    <th>Instructor Notes</th>
                    <th>Maintainers</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td>The Unix Shell</td>
                    <td style="text-align:center"><a href="https://swcarpentry.github.io/shell-novice/"><i
                                class="fas fa-window-maximize"></i></a></td>
                    <td style="text-align:center"><a href="https://github.com/swcarpentry/shell-novice"><i
                                class="fab fa-github"></i></a></td>
                    <td style="text-align:center"><a href="https://swcarpentry.github.io/shell-novice/reference"><i
                                class="fas fa-eye"></i></a></td>
                    <td style="text-align:center"><a
                            href="https://swcarpentry.github.io/shell-novice/instructor/instructor-notes"><i
                                class="fas fa-plus"></i></a></td>
                    <td>Jacob Deppen, Benson Muite</td>
                </tr>
                <tr>
                    <td>Version control with Git</td>
                    <td style="text-align:center"><a href="https://swcarpentry.github.io/git-novice"><i
                                class="fas fa-window-maximize"></i></a></td>
                    <td style="text-align:center"><a href="https://github.com/swcarpentry/git-novice"><i
                                class="fab fa-github"></i></a></td>
                    <td style="text-align:center"><a href="https://swcarpentry.github.io/git-novice/reference"><i
                                class="fas fa-eye"></i></a></td>
                    <td style="text-align:center"><a
                            href="https://swcarpentry.github.io/git-novice/instructor/instructor-notes"><i
                                class="fas fa-plus"></i></a></td>
                    <td>Erin Graham, Katherine Koziar, Martino Sorbaro</td>
                </tr>
                <tr>
                    <td>Programming with Python</td>
                    <td style="text-align:center"><a href="https://swcarpentry.github.io/python-novice-inflammation"><i
                                class="fas fa-window-maximize"></i></a></td>
                    <td style="text-align:center"><a href="https://github.com/swcarpentry/python-novice-inflammation"><i
                                class="fab fa-github"></i></a></td>
                    <td style="text-align:center"><a
                            href="https://swcarpentry.github.io/python-novice-inflammation/reference"><i
                                class="fas fa-eye"></i></a></td>
                    <td style="text-align:center"><a
                            href="https://swcarpentry.github.io/python-novice-inflammation/instructor/instructor-notes"><i
                                class="fas fa-plus"></i></a></td>
                    <td>Indraneel Chakraborty, Toan Phung, Alberto Villagran</td>
                </tr>
                <tr>
                    <td>Plotting and programming with Python</td>
                    <td style="text-align:center"><a href="https://swcarpentry.github.io/python-novice-gapminder"><i
                                class="fas fa-window-maximize"></i></a></td>
                    <td style="text-align:center"><a href="https://github.com/swcarpentry/python-novice-gapminder"><i
                                class="fab fa-github"></i></a></td>
                    <td style="text-align:center"><a
                            href="https://swcarpentry.github.io/python-novice-gapminder/reference"><i
                                class="fas fa-eye"></i></a></td>
                    <td style="text-align:center"><a
                            href="https://swcarpentry.github.io/python-novice-gapminder/instructor/instructor-notes"><i
                                class="fas fa-plus"></i></a></td>
                    <td>Allen Lee, Sourav Singh, Olav Vahtras</td>
                </tr>
                <tr>
                    <td>Programming with R</td>
                    <td style="text-align:center"><a href="https://swcarpentry.github.io/r-novice-inflammation/"><i
                                class="fas fa-window-maximize"></i></a></td>
                    <td style="text-align:center"><a href="https://github.com/swcarpentry/r-novice-inflammation"><i
                                class="fab fa-github"></i></a></td>
                    <td style="text-align:center"><a
                            href="https://swcarpentry.github.io/r-novice-inflammation/reference"><i
                                class="fas fa-eye"></i></a></td>
                    <td style="text-align:center"><a
                            href="https://swcarpentry.github.io/r-novice-inflammation/instructor/instructor-notes"><i
                                class="fas fa-plus"></i></a></td>
                    <td>Rohit Goswami, Hugo Gruson, Isaac Jennings</td>
                </tr>
                <tr>
                    <td>R for Reproducible Scientific Analysis</td>
                    <td style="text-align:center"><a href="https://swcarpentry.github.io/r-novice-gapminder"><i
                                class="fas fa-window-maximize"></i></a></td>
                    <td style="text-align:center"><a href="https://github.com/swcarpentry/r-novice-gapminder"><i
                                class="fab fa-github"></i></a></td>
                    <td style="text-align:center"><a
                            href="https://swcarpentry.github.io/r-novice-gapminder/reference"><i
                                class="fas fa-eye"></i></a></td>
                    <td style="text-align:center"><a
                            href="https://swcarpentry.github.io/r-novice-gapminder/instructor/instructor-notes"><i
                                class="fas fa-plus"></i></a></td>
                    <td>Matthieu Bruneaux, Sehrish Kanwal, Naupaka Zimmerman</td>
                </tr>
            </tbody>
        </table>
    </div>
    <h2 id="core-lessons-in-spanish">Core Lessons in Spanish</h2>
    <div class="table-striped overflow-x-auto">
        <table>
            <thead>
                <tr>
                    <th>LecciÃ³n</th>
                    <th>Sitio web</th>
                    <th>Repositorio</th>
                    <th>Referencias</th>
                    <th>Notas para Instructoras/es</th>
                    <th>Reponsable(s) del mantenimiento</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td>La Terminal de Unix</td>
                    <td style="text-align:center"><a href="https://swcarpentry.github.io/shell-novice-es"><i
                                class="fas fa-window-maximize"></i></a></td>
                    <td style="text-align:center"><a href="https://github.com/swcarpentry/shell-novice-es"><i
                                class="fab fa-github"></i></a></td>
                    <td style="text-align:center"><a href="https://swcarpentry.github.io/shell-novice-es/reference"><i
                                class="fas fa-eye"></i></a></td>
                    <td style="text-align:center"><a
                            href="https://swcarpentry.github.io/shell-novice-es/instructor/instructor-notes"><i
                                class="fas fa-plus"></i></a></td>
                    <td>VerÃ³nica JimÃ©nez, Clara Llebot, Heladia Salgado</td>
                </tr>
                <tr>
                    <td>Control de versiones con Git</td>
                    <td style="text-align:center"><a href="https://swcarpentry.github.io/shell-novice-es"><i
                                class="fas fa-window-maximize"></i></a></td>
                    <td style="text-align:center"><a href="https://github.com/swcarpentry/git-novice-es"><i
                                class="fab fa-github"></i></a></td>
                    <td style="text-align:center"><a href="https://swcarpentry.github.io/git-novice-es/reference"><i
                                class="fas fa-eye"></i></a></td>
                    <td style="text-align:center"><a
                            href="https://swcarpentry.github.io/git-novice-es/instructor/instructor-notes"><i
                                class="fas fa-plus"></i></a></td>
                    <td>Jean-Paul Courneya, Clara Llebot, Mariana Patricia Gomez Nicolas</td>
                </tr>
                <tr>
                    <td>R para AnÃ¡lisis CientÃ­ficos Reproducibles</td>
                    <td style="text-align:center"><a href="https://swcarpentry.github.io/r-novice-gapminder-es"><i
                                class="fas fa-window-maximize"></i></a></td>
                    <td style="text-align:center"><a href="https://github.com/swcarpentry/r-novice-gapminder-es"><i
                                class="fab fa-github"></i></a></td>
                    <td style="text-align:center"><a
                            href="https://swcarpentry.github.io/r-novice-gapminder-es/reference"><i
                                class="fas fa-eye"></i></a></td>
                    <td style="text-align:center"><a
                            href="https://swcarpentry.github.io/r-novice-gapminder-es/instructor/instructor-notes"><i
                                class="fas fa-plus"></i></a></td>
                    <td>VerÃ³nica JimÃ©nez, Heladia Salgado, Nelly SÃ©lem</td>
                </tr>
            </tbody>
        </table>
    </div>
    <h2 id="additional-lessons">Additional Lessons</h2>
    <p>These lessons are not part of the core Software Carpentry curriculum but can be offered as supplementary lessons.
        Please <a href="https://carpentries.org/contact">contact us</a> for more information.</p>
    <div class="table-striped overflow-x-auto">
        <table>
            <thead>
                <tr>
                    <th>Lesson</th>
                    <th>Site</th>
                    <th>Repository</th>
                    <th>Reference</th>
                    <th>Instructor Notes</th>
                    <th>Maintainers</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td>Automation and Make</td>
                    <td style="text-align:center"><a href="https://swcarpentry.github.io/make-novice"><i
                                class="fas fa-window-maximize"></i></a></td>
                    <td style="text-align:center"><a href="https://github.com/swcarpentry/make-novice"><i
                                class="fab fa-github"></i></a></td>
                    <td style="text-align:center"><a href="https://swcarpentry.github.io/make-novice/reference"><i
                                class="fas fa-eye"></i></a></td>
                    <td style="text-align:center"><a
                            href="https://swcarpentry.github.io/make-novice/instructor/instructor-notes"><i
                                class="fas fa-plus"></i></a></td>
                    <td>Gerard Capes</td>
                </tr>
                <tr>
                    <td>Programming with MATLAB</td>
                    <td style="text-align:center"><a href="https://swcarpentry.github.io/matlab-novice-inflammation"><i
                                class="fas fa-window-maximize"></i></a></td>
                    <td style="text-align:center"><a href="https://github.com/swcarpentry/matlab-novice-inflammation"><i
                                class="fab fa-github"></i></a></td>
                    <td style="text-align:center"><a
                            href="https://swcarpentry.github.io/matlab-novice-inflammation/reference"><i
                                class="fas fa-eye"></i></a></td>
                    <td style="text-align:center"><a
                            href="https://swcarpentry.github.io/matlab-novice-inflammation/instructor/instructor-notes"><i
                                class="fas fa-plus"></i></a></td>
                    <td>Daniel Cummins, Padem dhar Dwivedi</td>
                </tr>
                <tr>
                    <td>Using Databases and SQL</td>
                    <td style="text-align:center"><a href="https://swcarpentry.github.io/sql-novice-survey"><i
                                class="fas fa-window-maximize"></i></a></td>
                    <td style="text-align:center"><a href="https://github.com/swcarpentry/sql-novice-survey"><i
                                class="fab fa-github"></i></a></td>
                    <td style="text-align:center"><a href="https://swcarpentry.github.io/sql-novice-survey/reference"><i
                                class="fas fa-eye"></i></a></td>
                    <td style="text-align:center"><a
                            href="https://swcarpentry.github.io/sql-novice-survey/instructor/instructor-notes"><i
                                class="fas fa-plus"></i></a></td>
                    <td>Henry Senyondo</td>
                </tr>
            </tbody>
        </table>
    </div>
</div>
~~~
{: .language-html}

It seems we are on the right track.
Now let's focus on the first `table` element:

~~~
(soup
 .find(string = "Core Lessons in English")
 .find_parents()[1]
 .find("table"))
~~~
{: .language-python}

~~~
<table>
    <thead>
        <tr>
            <th>Lesson</th>
            <th>Site</th>
            <th>Repository</th>
            <th>Reference</th>
            <th>Instructor Notes</th>
            <th>Maintainers</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>The Unix Shell</td>
            <td style="text-align:center"><a href="https://swcarpentry.github.io/shell-novice/"><i
                        class="fas fa-window-maximize"></i></a></td>
            <td style="text-align:center"><a href="https://github.com/swcarpentry/shell-novice"><i
                        class="fab fa-github"></i></a></td>
            <td style="text-align:center"><a href="https://swcarpentry.github.io/shell-novice/reference"><i
                        class="fas fa-eye"></i></a></td>
            <td style="text-align:center"><a
                    href="https://swcarpentry.github.io/shell-novice/instructor/instructor-notes"><i
                        class="fas fa-plus"></i></a></td>
            <td>Jacob Deppen, Benson Muite</td>
        </tr>
        <tr>
            <td>Version control with Git</td>
            <td style="text-align:center"><a href="https://swcarpentry.github.io/git-novice"><i
                        class="fas fa-window-maximize"></i></a></td>
            <td style="text-align:center"><a href="https://github.com/swcarpentry/git-novice"><i
                        class="fab fa-github"></i></a></td>
            <td style="text-align:center"><a href="https://swcarpentry.github.io/git-novice/reference"><i
                        class="fas fa-eye"></i></a></td>
            <td style="text-align:center"><a
                    href="https://swcarpentry.github.io/git-novice/instructor/instructor-notes"><i
                        class="fas fa-plus"></i></a></td>
            <td>Erin Graham, Katherine Koziar, Martino Sorbaro</td>
        </tr>
        <tr>
            <td>Programming with Python</td>
            <td style="text-align:center"><a href="https://swcarpentry.github.io/python-novice-inflammation"><i
                        class="fas fa-window-maximize"></i></a></td>
            <td style="text-align:center"><a href="https://github.com/swcarpentry/python-novice-inflammation"><i
                        class="fab fa-github"></i></a></td>
            <td style="text-align:center"><a
                    href="https://swcarpentry.github.io/python-novice-inflammation/reference"><i
                        class="fas fa-eye"></i></a></td>
            <td style="text-align:center"><a
                    href="https://swcarpentry.github.io/python-novice-inflammation/instructor/instructor-notes"><i
                        class="fas fa-plus"></i></a></td>
            <td>Indraneel Chakraborty, Toan Phung, Alberto Villagran</td>
        </tr>
        <tr>
            <td>Plotting and programming with Python</td>
            <td style="text-align:center"><a href="https://swcarpentry.github.io/python-novice-gapminder"><i
                        class="fas fa-window-maximize"></i></a></td>
            <td style="text-align:center"><a href="https://github.com/swcarpentry/python-novice-gapminder"><i
                        class="fab fa-github"></i></a></td>
            <td style="text-align:center"><a href="https://swcarpentry.github.io/python-novice-gapminder/reference"><i
                        class="fas fa-eye"></i></a></td>
            <td style="text-align:center"><a
                    href="https://swcarpentry.github.io/python-novice-gapminder/instructor/instructor-notes"><i
                        class="fas fa-plus"></i></a></td>
            <td>Allen Lee, Sourav Singh, Olav Vahtras</td>
        </tr>
        <tr>
            <td>Programming with R</td>
            <td style="text-align:center"><a href="https://swcarpentry.github.io/r-novice-inflammation/"><i
                        class="fas fa-window-maximize"></i></a></td>
            <td style="text-align:center"><a href="https://github.com/swcarpentry/r-novice-inflammation"><i
                        class="fab fa-github"></i></a></td>
            <td style="text-align:center"><a href="https://swcarpentry.github.io/r-novice-inflammation/reference"><i
                        class="fas fa-eye"></i></a></td>
            <td style="text-align:center"><a
                    href="https://swcarpentry.github.io/r-novice-inflammation/instructor/instructor-notes"><i
                        class="fas fa-plus"></i></a></td>
            <td>Rohit Goswami, Hugo Gruson, Isaac Jennings</td>
        </tr>
        <tr>
            <td>R for Reproducible Scientific Analysis</td>
            <td style="text-align:center"><a href="https://swcarpentry.github.io/r-novice-gapminder"><i
                        class="fas fa-window-maximize"></i></a></td>
            <td style="text-align:center"><a href="https://github.com/swcarpentry/r-novice-gapminder"><i
                        class="fab fa-github"></i></a></td>
            <td style="text-align:center"><a href="https://swcarpentry.github.io/r-novice-gapminder/reference"><i
                        class="fas fa-eye"></i></a></td>
            <td style="text-align:center"><a
                    href="https://swcarpentry.github.io/r-novice-gapminder/instructor/instructor-notes"><i
                        class="fas fa-plus"></i></a></td>
            <td>Matthieu Bruneaux, Sehrish Kanwal, Naupaka Zimmerman</td>
        </tr>
    </tbody>
</table>
~~~
{: .language-html}

Now we can get a list of row elements with

~~~
rows = (soup
 .find(string = "Core Lessons in English")
 .find_parents()[1]
 .find("table")
 .find_all("tr"))
~~~
{: .language-python}

Let's focus now on the second element (the first contains the column headings):

~~~
rows[1]
~~~
{: .language-python}

~~~
<tr>
    <td>The Unix Shell</td>
    <td style="text-align:center"><a href="https://swcarpentry.github.io/shell-novice/"><i
                class="fas fa-window-maximize"></i></a></td>
    <td style="text-align:center"><a href="https://github.com/swcarpentry/shell-novice"><i
                class="fab fa-github"></i></a></td>
    <td style="text-align:center"><a href="https://swcarpentry.github.io/shell-novice/reference"><i
                class="fas fa-eye"></i></a></td>
    <td style="text-align:center"><a href="https://swcarpentry.github.io/shell-novice/instructor/instructor-notes"><i
                class="fas fa-plus"></i></a></td>
    <td>Jacob Deppen, Benson Muite</td>
</tr>
~~~
{: .language-html}

We can now split the row 
into six table data elements:

~~~
td0, td1, td2, td3, td4, td5 = rows[1].find_all("td")
~~~
{: .language-python}

If we want the link to the lesson page,
we can look at the `<a>` tag in `td1`,
and specifically at its `href` attribute:

~~~
link = td1.find("a")["href"]
link
~~~
{: .language-python}

~~~
'https://swcarpentry.github.io/shell-novice/'
~~~
{: .output}

We can get a list of maintainer names 
from the text content of `td5`:

~~~
maintainers = td5.text.split(",")

print(maintainers)
~~~
{: .language-python}

~~~
['Jacob Deppen', 'Benson Muite'] 
~~~
{: .output}

> ## A more direct way
>
> Can we look directly for table elements in the soup?
> How would you do that?
> Would that work?
>
> > ## Solution
> >
> > We can check how many 
> > `table` elements are in the soup 
> > with
> >
> > ~~~
> > len(soup.find_all("table"))
> > ~~~
> > {: .language-python}
> >
> > We gather that there are three tables in the soup.
> > `find_all` returns a list of them, so we can index
> > into the list to access the one we want. For example:
> > ~~~
> > soup.find_all("table")[1]
> > ~~~
> > {: .language-python}
> > can be used to access the second table.
> {: .solution}
{: .challenge}

> ## List the Lessons
>
> Create a list of all the lessons,
> reporting for each one:
> - lesson name
> - link
> - names of maintainers
> 
> > ## Solution
> >
> > ~~~
> > rows = soup.find("table").find_all("tr")
> >  # Remove the first row that only contains headings
> > rows.pop(0)
> > 
> > def process_row(row):
> >     td0,td1, _, _, _,td5 = row.find_all("td")
> >     link = td1.find("a")["href"]
> >     lesson = td0.text
> >     maintainers = td5.text.split(",")
> >     return dict(
> >         lesson = lesson,
> >         link = link,
> >         maintainers = maintainers
> >     ) 
> > 
> > lessons = []
> > for row in rows:
> >     lessons.append(process_row(row))
> > print(lessons)
> > ~~~
> > {: .language-python}
> {: .solution}
{: .challenge}


## Additional material

Beautiful Soup is a rich library
that has a lot of powerful features 
that we are unable to discuss here.

A close look at [the official documentation][bs4-docs]
is worth the time 
for anyone seriously interested 
in web scraping.

> ## Scraping the locations for tide gauge stations into a Pandas dataframe
>
> Look at [the locations for tide gauge stations][psmsl].
> How would you extract these data as a Pandas dataframe?
>
> > ## Solution
> > ~~~
> > import requests
> > import pandas
> > from bs4 import BeautifulSoup
> >
> > # From the url displayed in the browser in the address bar 
> > response = requests.get("https://psmsl.org/data/obtaining/")
> >
> > soup = BeautifulSoup(response.text,"html.parser")
> >
> > rows = soup.find_all("table")
> > ~~~
> > {: .language-python}
> >
> > Then we can convert the string to a pandas dataframe:
> > ~~~
> > df = pandas.read_html(str(rows))[0]
> > ~~~
> > {: .language-python}
> > we now have the station location data inside a pandas dataframe ready for
> > processing, graphing etc.
> {: .solution}
{: .challenge}


## Javascript code, the DOM and Selenium

The JavaScript code running on the page 
can actively change the structure of the HTML document.
For some web pages, 
this is a crucial part
of the rendering process:
in some of those cases 
the JavaScript code must be run
to download 
the data you are looking for 
from another URL,
and populate the web page with that data
and any additional element of the page design.

In those cases, 
using `requests` and `BeautifulSoup`
might not be enough
(as `requests` gets the HTML
without running the JavaScript code on the page),
but you can use the [Selenium WebDriver][selenium]
to load the page in a fully-fledged browser 
and automate the interaction with it.


[bs4-docs]: https://www.crummy.com/software/BeautifulSoup/bs4/doc/
[software-carpentry-lessons]: https://software-carpentry.org/lessons/
[psmsl]: https://psmsl.org/data/obtaining/
[mdn-elements-reference]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element
[selenium]: https://www.selenium.dev/documentation/en/webdriver/
