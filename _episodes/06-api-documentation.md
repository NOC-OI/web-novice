---
title: "API Documentation"
teaching: 15
exercises: 0
questions:
- "How are APIs documented?"
objectives:
- ""
keypoints:
- ""
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

Navigate to [The Software Carpentry Lessons][eds-citation-swagger].
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
  For examples, see the [reference on the Mozilla Developer Network][mdn-elements-reference]
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

> ## EDS Citation API
>
> Look at [BODC's EDS citation API][eds-citation-swagger]
> (See this [page][eds-citation-background] page for background information.)
> Can you find out the total number of citation count for the Polar Data Centre (PDC),
> without leaving the page?
>
> > ## Solution
> > There are multiple ways to do this. One is to use the `/centre` endpoint. You could
> > also use the `/centre/{centre_name}` endpoint, where `centre_name` is 
> > `Polar Data Centre (PDC)`. To interact with the documentation on the page, click
> > the `Try it out` button, enter any desired parameters, then click the `Execute` 
> > button.
> {: .solution}
{: .challenge}



[eds-citation-swagger]: https://www.bodc.ac.uk/eds-citation/docs
[eds-citation-background]: https://eds.ukri.org/news/impacts/who-has-used-my-data-our-brand-new-citation-api
