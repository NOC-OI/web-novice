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
to find information about a buoy from NOAA's National Data Buoy Centre (NDBC).

## Exploring HTML code in the browser

Navigate to [the station page for station 46014][national-data-buoy-centre].
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
the NDBC website content:

~~~
import requests
from bs4 import BeautifulSoup

response = requests.get("https://www.ndbc.noaa.gov/station_page.php?station=46014")
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
there is the text "Conditions at 46014 as of"
inside a `<table>` tag (code reindented for clarity)

~~~
...
<table class="currentobs"><caption class="titleDataHeader">Conditions at 46014 as of<br>(12:30 am PDT)<br>0730 GMT on 07/22/2026:</caption>
<tbody><tr>
	<td class="parmform" colspan="2">
<form name="uom_m" action="/station_page.php" method="get">
<input type="hidden" name="station" value="46014">
<label for="uom">Unit of Measure: </label><select name="uom" id="uom" size="1">
<option value="E" selected="selected"> Imperial</option>
<option value="M"> Metric</option>
</select>&nbsp;&nbsp;
<label for="tz">Time Zone: </label><select name="tz" id="tz" size="1">
<option value="STN">Station Local Time</option>
<option value="GMT">Greenwich Mean Time [GMT]</option>
<option value="BST">British Summer Time [GMT+1]</option>
<option value="EGT">Eastern Greenland [GMT-1]</option>
<option value="AZOT">Azores [GMT-2]</option>
<option value="WGT">Western Greenland [GMT-3]</option>
<option value="AST">Atlantic Standard [GMT-4]</option>
<option value="EST">US/Eastern Standard</option>
<option value="CST">US/Central Standard</option>
<option value="MST">US/Mountain Standard</option>
<option value="PST">US/Pacific Standard</option>
<option value="AKST">Alaska Standard [GMT-9]</option>
<option value="HST">Hawaii Standard [GMT-10]</option>
<option value="HAST">Hawaii-Aleutian Standard [GMT-10]</option>
<option value="SST">Samoa Standard [GMT-11]</option>
<option value="IDLW">International Date Line West [GMT-12]</option>
<option value="WET">Western European [GMT+0]</option>
<option value="CET">Central European [GMT+1]</option>
<option value="EET">Eastern European [GMT+2]</option>
<option value="MSK">Moscow [GMT+3]</option>
<option value="GMT+4">GMT+4</option>
<option value="PKT">Pakistan Standard [GMT+5]</option>
<option value="GMT+6">GMT+6</option>
<option value="ICT">Indochina Time [GMT+7]</option>
<option value="HKT">Hong Kong [GMT+8]</option>
<option value="JST">Japan Standard [GMT+9]</option>
<option value="CHST">Chamorro Standard [GMT+10]</option>
<option value="ZP11">GMT+11</option>
<option value="IDLE">International Date Line East [GMT+12]</option>
</select>&nbsp;&nbsp;<input type="submit" value=" Select "></form>
<p class="smallertext"><i>Click on the graph icon in the table below to see a time series plot of the last five days of that observation.</i></p></td></tr><tr><td><a href="/show_plot.php?station=46014&meas=wdir&uom=E&tz=PST"><img alt="5-day plot - Wind Direction" title="5-day plot - Wind Direction" src="/images/graph04.gif" width="21" height="20"></a> Wind Direction (WDIR):</td><td>ESE ( 120 deg true )</td></tr>
<tr><td><a href="/show_plot.php?station=46014&meas=wspd&uom=E&tz=PST"><img alt="5-day plot - Wind Speed" title="5-day plot - Wind Speed" src="/images/graph04.gif" width="21" height="20"></a> Wind Speed (WSPD):</td><td>  7.8 kts</td>
</tr><tr><td><a href="/show_plot.php?station=46014&meas=wgst&uom=E&tz=PST"><img alt="5-day plot - Wind Gust" title="5-day plot - Wind Gust" src="/images/graph04.gif" width="21" height="20"></a> Wind Gust (GST):</td><td>  9.7 kts</td>
</tr>
<tr><td><a href="/show_plot.php?station=46014&meas=pres&uom=E&tz=PST"><img alt="5-day plot - Atmospheric Pressure" title="5-day plot - Atmospheric Pressure" src="/images/graph04.gif" width="21" height="20"></a> Atmospheric Pressure (PRES):</td><td>29.95 in</td>
</tr>
<tr><td><a href="/show_plot.php?station=46014&meas=wtmp&uom=E&tz=PST"><img alt="5-day plot - Water Temperature" title="5-day plot - Water Temperature" src="/images/graph04.gif" width="21" height="20"></a> Water Temperature (WTMP):</td><td> 58.8 &deg;F</td>
</tr>
<tr><td><a href="/show_plot.php?station=46014&meas=w10m&uom=E&tz=PST"><img alt="5-day plot - Wind Speed at 10 Meters" title="5-day plot - Wind Speed at 10 Meters" src="/images/graph04.gif" width="21" height="20"></a> Wind Speed at 10 meters (WSPD10M):</td><td>  9.7 kts</td>
</tr>
<tr><td><a href="/show_plot.php?station=46014&meas=w20m&uom=E&tz=PST"><img alt="5-day plot - Wind Speed at 20 Meters" title="5-day plot - Wind Speed at 20 Meters" src="/images/graph04.gif" width="21" height="20"></a> Wind Speed at 20 meters (WSPD20M):</td><td>  9.7 kts</td>
</tr>
<tr><td colspan="2"><a href="/show_plot.php?station=46014&meas=wdpr&uom=E&tz=PST"><img alt="5-day plot -  Wind Speed, Wind Gust and Atmospheric Pressure" title="5-day plot -  Wind Speed, Wind Gust and Atmospheric Pressure" src="/images/graph04.gif" width="21" height="20"></a> <a href="/show_plot.php?station=46014&meas=wdpr&uom=E&tz=PST">Combined plot of Wind Speed, Gust, and Air Pressure</a></td>
</tr>
</tbody></table>
...
~~~
{: .language-html}

We can then look for the table 
by finding the HTML element 
that contains that text,
using the `string` keyword argument:

~~~
(soup.find(string="Conditions at 46014 as of"))
~~~
{: .language-python}

~~~
'Conditions at 46014 as of'
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
 .find(string = "Conditions at 46014 as of")
 .find_parents()[1])
~~~
{: .language-python}

~~~
<table class="currentobs"><caption class="titleDataHeader">Conditions at 46014 as of<br/>(12:40 am PDT)<br/>0740 GMT on 07/22/2026:</caption>
<tbody><tr>
<td class="parmform" colspan="2">
<form action="/station_page.php" method="get" name="uom_m">
<input name="station" type="hidden" value="46014"/>
<label for="uom">Unit of Measure: </label><select id="uom" name="uom" size="1">
<option selected="selected" value="E"> Imperial</option>
<option value="M"> Metric</option>
</select>  
<label for="tz">Time Zone: </label><select id="tz" name="tz" size="1">
<option value="STN">Station Local Time</option>
<option value="GMT">Greenwich Mean Time [GMT]</option>
<option value="BST">British Summer Time [GMT+1]</option>
<option value="EGT">Eastern Greenland [GMT-1]</option>
<option value="AZOT">Azores [GMT-2]</option>
<option value="WGT">Western Greenland [GMT-3]</option>
<option value="AST">Atlantic Standard [GMT-4]</option>
<option value="EST">US/Eastern Standard</option>
<option value="CST">US/Central Standard</option>
<option value="MST">US/Mountain Standard</option>
<option value="PST">US/Pacific Standard</option>
<option value="AKST">Alaska Standard [GMT-9]</option>
<option value="HST">Hawaii Standard [GMT-10]</option>
<option value="HAST">Hawaii-Aleutian Standard [GMT-10]</option>
<option value="SST">Samoa Standard [GMT-11]</option>
<option value="IDLW">International Date Line West [GMT-12]</option>
<option value="WET">Western European [GMT+0]</option>
<option value="CET">Central European [GMT+1]</option>
<option value="EET">Eastern European [GMT+2]</option>
<option value="MSK">Moscow [GMT+3]</option>
<option value="GMT+4">GMT+4</option>
<option value="PKT">Pakistan Standard [GMT+5]</option>
<option value="GMT+6">GMT+6</option>
<option value="ICT">Indochina Time [GMT+7]</option>
<option value="HKT">Hong Kong [GMT+8]</option>
<option value="JST">Japan Standard [GMT+9]</option>
<option value="CHST">Chamorro Standard [GMT+10]</option>
<option value="ZP11">GMT+11</option>
<option value="IDLE">International Date Line East [GMT+12]</option>
</select>  <input type="submit" value=" Select "/></form>
<p class="smallertext"><i>Click on the graph icon in the table below to see a time series plot of the last five days of that observation.</i></p></td></tr><tr><td><a href="/show_plot.php?station=46014&amp;meas=wdir&amp;uom=E&amp;tz=PST"><img alt="5-day plot - Wind Direction" height="20" src="/images/graph04.gif" title="5-day plot - Wind Direction" width="21"/></a> Wind Direction (WDIR):</td><td>SSE ( 150 deg true )</td></tr>
<tr><td><a href="/show_plot.php?station=46014&amp;meas=wspd&amp;uom=E&amp;tz=PST"><img alt="5-day plot - Wind Speed" height="20" src="/images/graph04.gif" title="5-day plot - Wind Speed" width="21"/></a> Wind Speed (WSPD):</td><td> 11.7 kts</td>
</tr><tr><td><a href="/show_plot.php?station=46014&amp;meas=wgst&amp;uom=E&amp;tz=PST"><img alt="5-day plot - Wind Gust" height="20" src="/images/graph04.gif" title="5-day plot - Wind Gust" width="21"/></a> Wind Gust (GST):</td><td> 15.5 kts</td>
</tr>
<tr><td><a href="/show_plot.php?station=46014&amp;meas=pres&amp;uom=E&amp;tz=PST"><img alt="5-day plot - Atmospheric Pressure" height="20" src="/images/graph04.gif" title="5-day plot - Atmospheric Pressure" width="21"/></a> Atmospheric Pressure (PRES):</td><td>29.94 in</td>
</tr>
<tr><td><a href="/show_plot.php?station=46014&amp;meas=wtmp&amp;uom=E&amp;tz=PST"><img alt="5-day plot - Water Temperature" height="20" src="/images/graph04.gif" title="5-day plot - Water Temperature" width="21"/></a> Water Temperature (WTMP):</td><td> 58.6 °F</td>
</tr>
<tr><td><a href="/show_plot.php?station=46014&amp;meas=w10m&amp;uom=E&amp;tz=PST"><img alt="5-day plot - Wind Speed at 10 Meters" height="20" src="/images/graph04.gif" title="5-day plot - Wind Speed at 10 Meters" width="21"/></a> Wind Speed at 10 meters (WSPD10M):</td><td> 11.7 kts</td>
</tr>
<tr><td><a href="/show_plot.php?station=46014&amp;meas=w20m&amp;uom=E&amp;tz=PST"><img alt="5-day plot - Wind Speed at 20 Meters" height="20" src="/images/graph04.gif" title="5-day plot - Wind Speed at 20 Meters" width="21"/></a> Wind Speed at 20 meters (WSPD20M):</td><td> 13.6 kts</td>
</tr>
<tr><td colspan="2"><a href="/show_plot.php?station=46014&amp;meas=wdpr&amp;uom=E&amp;tz=PST"><img alt="5-day plot -  Wind Speed, Wind Gust and Atmospheric Pressure" height="20" src="/images/graph04.gif" title="5-day plot -  Wind Speed, Wind Gust and Atmospheric Pressure" width="21"/></a> <a href="/show_plot.php?station=46014&amp;meas=wdpr&amp;uom=E&amp;tz=PST">Combined plot of Wind Speed, Gust, and Air Pressure</a></td>
</tr>
</tbody></table>
~~~
{: .language-html}

It seems we are on the right track - we've got the `table` element we were interested.

Now we can get a list of row elements with

~~~
rows = (soup
 .find(string = "Conditions at 46014 as of")
 .find_parents()[1]
 .find_all("tr"))
~~~
{: .language-python}

Let's focus now on the second element (the first contains the unit and time zone choices):

~~~
rows[1]
~~~
{: .language-python}

~~~
<tr><td><a href="/show_plot.php?station=46014&amp;meas=wdir&amp;uom=E&amp;tz=PST"><img alt="5-day plot - Wind Direction" height="20" src="/images/graph04.gif" title="5-day plot - Wind Direction" width="21"/></a> Wind Direction (WDIR):</td><td>SSE ( 150 deg true )</td></tr>
~~~
{: .language-html}

We can now split the row 
into two table data elements:

~~~
td0, td1 = rows[1].find_all("td")
~~~
{: .language-python}

If we want the link to the graph,
we can look at the `<a>` tag in `td1`,
and specifically at its `href` attribute:

~~~
link = td0.find("a")["href"]
link
~~~
{: .language-python}

~~~
'/show_plot.php?station=46014&meas=wdir&uom=E&tz=PST'
~~~
{: .output}

This is not particularly useful to us in this case because it's a relative link.

We can get, for example, the general wind direction the text content of `td1`:

~~~
wind_directions = td1.text.split()

print(wind_directions[0])
~~~
{: .language-python}

~~~
'SSE'
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
> > soup.find_all("table")[0]
> > ~~~
> > {: .language-python}
> > can be used to access the first table.
> {: .solution}
{: .challenge}

> ## List the Lessons
>
> Create a list of tuples for each time available containing:
> - wind speed
> - water_temperature
> 
> > ## Solution
> >
> > ~~~
> > rows = soup.find_all("table")[3].find_all("tr")
> > # Remove the first row that only contains headings
> > rows.pop(0)
> > 
> > def process_row(row):
> >     _,td1, _, _, _, _, _, _, _, _, td10, _, _, _, _ = row.find_all("td")
> >     wind_speed = td1.text.strip()
> >     water_temperature = td10.text.strip()
> >     return (wind_speed, water_temperature) 
> > 
> > wind_speed_and_water_temperature = []
> > for row in rows:
> >     wind_speed_and_water_temperature.append(process_row(row))
> > print(wind_speed_and_water_temperature)
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
> How would you extract these data as a Pandas dataframe? (Which is probably a much more 
> useful way to end up with the data than in a list, as we did above.)
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
[national-data-buoy-centre]: https://www.ndbc.noaa.gov/station_page.php?station=46014
[psmsl]: https://psmsl.org/data/obtaining/
[mdn-elements-reference]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element
[selenium]: https://www.selenium.dev/documentation/en/webdriver/
