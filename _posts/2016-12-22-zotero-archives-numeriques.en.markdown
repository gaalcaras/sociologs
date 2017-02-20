---
layout: post
title:  "A practical approach to online archives"
lang: en
permalink: /en/practical-approach-online-archives/
date:   2016-12-22 18:01:27 +0100
thumbnail: images/archive.jpg
summary: >
  Online activities are increasingly used for fieldwork and data analysis in sociology.
  But their instability and their volume make them difficult to manipulate.
---

Online activities are quickly becoming a major source of both qualitative and quantitative data for sociologists.
From forums to blog posts, email conversations to social networks, these sources complement (and sometimes renew) the traditional issues and methods of the social sciences[^beuscart].

Unfortunately, I had to first deal with a very concrete issue before I could address these more theoretical questions: how do I handle this data effectively?

One of my sources is the Linux Kernel Mailing List (LKML[^lkml]), where thousands of developers have exchanged over 2.5 million emails between 1995 and 2016.
After I read a hundred relevant messages, and with the goal of reading many more, I understood that each additional email made the task more painful if I just took notes in a notebook, be it paper or digital.

Sticking with this process meant I had to choose between losing a considerable amount of time  referencing these emails or accepting to be less rigorous with their management.
These two options being equally unsatisfactory, I began to sketch out a practical system to manage these materials.
It needed to match three criteria:

1. Fast extraction of any email I found relevant
2. Keeping an offline copy of the email, in case the online archives become unavailable somehow
3. Easy and accurate referencing of an email, in a paper or in my thesis

This post will explain in detail how I implemented this system.
I mentioned the LKML above, but I'll use the [MARC](http://marc.info) email archive as an example since its emails are easier to handle.
This system can be adapted to archive any public online activity, provided it's accessible with a URL and it has coherent formatting accross the board.
Forums, tweets, blog posts, message boards, usenet or email archives come to mind.

# The goal: archiving an email with one click

Before we get into the weeds, I want to give you a better idea of what this system looks like.

<center>
![The archival system in action](/assets/img/zotero-archives-numeriques/screencast.gif)

The archival system in action
</center>

All we have to do is to go the webage that hosts the email and click on a button in our browser toolbar.
That's it!
The metadata (author, sent date, subject and so on) are automatically extracted from the page, an exact copy of the page is downloaded in the background and the date of consultation is registered.
From there, we are able to keep track of all extracted emails and to reference them easily in the software of our choice.

## How does it work?

We will use two features of [Zotero](https://www.zotero.org/), which you may already use to manage scientific papers and references.
The first one is Zotero's built-in ability to fetch online resources directly from a browser.
The other one is that Zotero is [*open source*](https://opensource.org/osd) software, making it easy for us to extend its functionalities to suit our own needs.

More to the point, Zotero uses a system of "translators" : each bibliographic source has its own translator, which is actually a simple file.
The translators for various sources can be found on Github, whether it's [Cairn](https://github.com/zotero/translators/blob/master/Cairn.info.js), [Amazon](https://github.com/zotero/translators/blob/master/Amazon.js), [Wikipedia](https://github.com/zotero/translators/blob/master/Wikipedia.js) or [a thousand others](https://github.com/zotero/translators).

## Evaluating what we need to do

Our goal is getting clearer: we need to write a translator for Zotero, that is a few lines of code in a single file.

The quantity of actual work involved is quite low.
However, there *is* some technical knowledge required.
Translators are written in the programming language [JavaScript](https://www.javascript.com/)  and [XPath](https://en.wikipedia.org/wiki/XPath) queries are often unavoidable.
Moreover, being comfortable with [regular expressions](https://fr.wikipedia.org/wiki/Expression_rationnelle) is indeniably useful.

These words may frighten you, but don't give up yet: the barriers to entry are lower than it seems.
You do not have to be proficient in JavaScript to write a translator, far from it.
As to XPath queries, there are [numerous](https://en.wikipedia.org/wiki/XPath) [resources](https://openclassrooms.com/courses/structurez-vos-donnees-avec-xml/xpath-localiser-les-donnees) [online](http://edutechwiki.unige.ch/fr/Tutoriel_XPath) − the same goes for Regex.
Finally, the [Zotero docs](https://www.zotero.org/support/dev/translators) on translators contain some valuable information.
While this post won't teach you JavaScript or XPath themselves, I will try to make each step as clear as I can.

The final code of this tutorial is available on [Github](https://github.com/gaalcaras/translators/).

# Creating a translator: our working example

Before we begin, make sure you have installed [Firefox](https://firefox.com), [Zotero](https://www.zotero.org/) and [Scaffold](https://github.com/zotero/scaffold/releases/latest), a `.xpi` extension for Firefox ([read the docs](https://www.zotero.org/support/dev/translators/scaffold)), which makes it possible to edit, test and debug translators.

## Assess a typical page

The first step is, obviously, to find a page (preferably somewhat representative of other pages) containing the desired metadata.
In this tutorial, I use [this email](https://marc.info/?l=git&m=111490299628887&w=2) as an example.

![Our typical email](/assets/img/zotero-archives-numeriques/email.png)

<center>Our typical page</center>

Start by spotting the relevant metadata you want to extract:

| Field             | Value                                              |
| -----             | ------                                             |
| Title             | **Re: Trying to use AUTHOR_DATE**                  |
| Publication       | **git** - Mailing list ARChive                     |
| Author            | **Linus Torvalds**                                 |
| URL               | **https://marc.info/?l=git&m=111490299628887&w=2** |
| Date              | **2005-04-30**                                     |
| Consultation date | today's date                                       |
| Type              | email                                              |
| Language          | en                                                 |

<center>Values of our metadata fields</center>

The elements in bold can be directly extracted from the web page.
The good new is that we don't have a lot of values to fetch, only 5.

We can now launch Scaffold (in Firefox, Tools > Scafford).

![Scaffold's interface](/assets/img/zotero-archives-numeriques/scaffold.png)

<center>Scaffold's interface v3.2.3</center>

The icons provide quick access to the most frequent file management actions (open and save) and debugging tools.
The left panel serves as a text editor for the various components of the translator.
Each component has its own tab.
The right panel allows you to test and debug the translator, which frankly is the best thing about Scaffold[^neovim].
Read Scaffold's [documentation](https://www.zotero.org/support/dev/translators/scaffold) for a more exhaustive, although a bit outdated, explanation of the interface.

Pro tip: make sure you keep your test page as your active tab in Firefox (see the "Active Tab URL" in Scaffold) while you test the translator, otherwise debugging can sometimes prove to be a very painful task.

## Translator metadata

This is the simplest part of the process.
The metadata provides Zotero with information about the execution context of the translator.
You should edit at least three fields:

+ *Label* : the translator's name.
+ *Target* : a Regex matching the URLs targeted by the translator.
  In my case, `https?://marc.info` will do.
  You should test your Regex with the appropriate button (should return `true`).
+ *Translator type* : choose "Web".

Save your translator before switching to the Code tab.

## The code

The Code tab display the part of the translator that does most of the heavy lifting.
A web translator has to contain at least two functions: `detectWeb()` and `doWeb()`.

### What is `detectWeb()` for?

`detectWeb` is the first function to be executed.
Its role is to check if the active page matches the data source.
It has to return specific values:

1. If a single source is detected, return its type ("journalArticle" or in our case "email").
2. If several sources are detected, return the "multiple" type.
3. If no metadata can be detected, nothing is returned.

For now, we'll skip the detection part for later and code a function that always returns "email".

```javascript
function detectWeb (doc, url) {
  return 'email'
}
```

You can check that `detectWeb()` does return "email" every time using the "Run detectWeb" button in Scaffold.

### `doWeb()`, where most of the work happens

The second function `doWeb()` extracts the metadata and saves them in the Zotero database.

To do so, you have to create a new instance of the object `item`, edit its properties with the matching metadat from the page and then save it to the database.
The following gives you a broad idea of the process:

```javascript
function doWeb (doc, url) {
  /* Instanciating a new item object (email type) */
  var email = new Zotero.item('email')

  /* Edit its properties */
  email.title = 'Titre'
  /* And so on... */

  /* At last, save the object */
  email.complete()
}
```

As you can see, the process is quite linear.
Let's retrieve the data from the page.

#### Metadata extraction

In our working examble, the metadata all live at the top of the page.

![The desired metadata](/assets/img/zotero-archives-numeriques/marc-header.png)

<center>The metadata to fetch</center>

We're in luck: in some other case, we would have to extract metadata from all over the page.

The next step is to check the source code of the relevant parts of the webpage.
In Firefox, select the metadata, right-click and choose "Inspect".
Below is the HTML code resulting in the screenshot just above:

```html
<font size="+1">
List:       <a href="?l=git&amp;r=1&amp;w=2">git</a>
Subject:    <a href="?t=111481650200002&amp;r=1&amp;w=2">Re: Trying to use AUTHOR_DATE</a>
From:       <a href="?a=105701892400001&amp;r=1&amp;w=2">Linus Torvalds &lt;torvalds () osdl ! org&gt;</a>
Date:       <a href="https://marc.info/?l=git&amp;r=1&amp;b=200504&amp;w=2">2005-04-30 23:18:11</a>
Message-ID: <a href="?i=Pine.LNX.4.58.0504301607570.2296%20()%20ppc970%20!%20osdl%20!%20org">Pine.LNX.4.58.0504301607570.2296 () ppc970 ! osdl ! org</a>
</font>
```

We note that the data share the same parent [node](https://en.wikipedia.org/wiki/Node_\(computer_science\)) (or tag), `font`.
Once again our task is quite simple, since each piece of data is contained in a `a` node, that is a hypertext link.
First, we will extract the text of these nodes and store them in as many variables as needed.

```javascript
var listTxt = ZU.xpathText(doc, '//font/a[1]')
var subTxt = ZU.xpathText(doc, '//font/a[2]')
var fromTxt = ZU.xpathText(doc, '//font/a[3]')
var dateTxt = ZU.xpathText(doc, '//font/a[4]')
```

`ZU` points to `Zotero.Utilities` ([documentation](https://www.zotero.org/support/dev/translators/functions)) which is automatically loaded when a translator is run.

`xpathText(elt, req)` is one of the many methods in this `Utilities` object.
It's a function that runs a XPath query `req` on the XML element `elt`, and returns the `textContent` of the node (just like `/text()` in XPath).
In our working example above:

  + `//font/a[1]` means "among all the `font` nodes, select the first child node `a`"
  + the XML element is `doc`, provided as an argument to `doWeb()`.
    It contains the HTML code of the page.

I recommend frequently testing[^debug] your XPath requests, especially if you're new to this syntax.

Now that the text content of the nodes is extracted, we have to select the relevant parts of these strings.

#### For a fistful of Regex

The name of the list and the email subject are ready to use.
For the other metadata, we have to use regular expressions to get the name of the author (and get rid of the email address) and to separate the date and the time.

```javascript
/* Name Extraction */
var namePattern = new RegExp(/(.*)\s</)
var authorName = namePattern.exec(fromTxt)[1]

/* Date Extraction */
var datePattern = new RegExp(/(\d{4}-\d{2}-\d{2})/)
var sentDate = datePattern.exec(dateTxt)[1]
```

And *voilà*!
Our variables `authorName` and `sentDate` now contain the desired values.

#### Storing the metadata in Zotero

All we have to do is to edit the properties[^shorttitle] of the `email` object.

```javascript
email.title = subTxt
email.shortTitle = listTxt + ' - Mailing list ARChives'
email.libraryCatalog = email.shortTitle

email.creators.push(ZU.cleanAuthor(authorName))
email.date = sentDate

email.language = 'en'
email.accessDate = 'CURRENT_TIMESTAMP'
email.url = url
```

We use another function of the utilities, `cleanAuthor()`, to take care of the formatting of the author's name (identifying first and last name on its own).
Storing the date and the time of access to the email in `accessDate` is an essential part of the process: this property is then used to properly cite your archive in your document.

All we have to do is to signal to Zotero that we have successfully changed the object's properties, so that it records them in the database:

```javascript
email.complete()
```

We now have completed the bulk of the work in `doWeb()`.

### Improving `detectWeb()`

Remember `detectWeb()`?
We previously set it aside a few steps ago.
For now, it only returns "email" whatever the page is, which defeats the purpose of the function in the first place.
One of the checks we could do is that the page contains the text "Message-ID: ", which is a distinctive mark of the targeted pages.

```javascript
var msgId = ZU.xpathText(doc, '//font/text()')

if (msgId !== null && msgId.indexOf('Message-ID: ')) {
  return 'email'
}
```

## Complete translator code

Our translator is now ready.
Here's what the code looks like:

```javascript
function detectWeb (doc, url) {
  var msgId = ZU.xpathText(doc, '//font/text()')

  if (msgId !== null && msgId.indexOf('Message-ID: ')) {
    return 'email'
  }
}

function doWeb (doc, url) {
  var email = new Zotero.Item('email')

  /* Elements extraction */
  var listTxt = ZU.xpathText(doc, '//font/a[1]')
  var subTxt = ZU.xpathText(doc, '//font/a[2]')
  var fromTxt = ZU.xpathText(doc, '//font/a[3]')
  var dateTxt = ZU.xpathText(doc, '//font/a[4]')

  /* Name Extraction */
  var namePattern = new RegExp(/(.*)\s</)
  var authorName = namePattern.exec(fromTxt)[1]

  /* Date Extraction */
  var datePattern = new RegExp(/(\d{4}-\d{2}-\d{2})/)
  var sentDate = datePattern.exec(dateTxt)[1]

  email.title = subTxt
  email.shortTitle = listTxt + ' - Mailing list ARChives'

  email.creators.push(ZU.cleanAuthor(authorName))
  email.date = sentDate

  email.language = 'en'
  email.accessDate = 'CURRENT_TIMESTAMP'
  email.libraryCatalog = email.shortTitle
  email.url = url

  email.complete()
}
```

In the end, less than 40 lines of code enable us to archive, download and reference millions of emails in the hundreds of the MARC archive mailing lists.
I don't know about you, but I'd say it's worth the work!

# Conclusion

You can now harvest the results of your hard work.
Save your translator[^dossier], close Scaffold and reload the target page.
Zotero now should be able to recognize the data source:

<center>
![Zotero's icon shows you the data source](/assets/img/zotero-archives-numeriques/zotero-icon.png)

Zotero now recognizes the data source
</center>

You can find all of my custom translators in this [Github repo](https://github.com/gaalcaras/translators).

Of course, the code above is a Minimum Working Example.
I encourage you to read the code of more complex translators in the [official repository](https://github.com/zotero/translators).
In a future post, I'll tackle the matter of robust testing of translators to improve their reliability.


[^beuscart]: A French reference: Jean-Samuel Beuscart, Éric Dagiral, Sylvain Parasie, *Sociologie d'internet*, Armand Colin (2016),
  URL : http://www.armand-colin.com/sociologie-dinternet-9782200612429.
[^lkml]: The Linux Kernel Mailing List is the development list of the Linux Kernel.
  One of the many available archives : http://lkml.iu.edu/hypermail/linux/kernel/index.html.
[^neovim]: The Scaffold code editor is kind of awkward, to say the least, especially if your translator is getting long.
  I highly recommend using a better text editor ([neovim](https://neovim.io) is dear to my heart) once you get beyond the basics.
[^dossier]: The entire file of the translator (metadata, functions and test cases) are stored in a `translators` subfolder of your Zotero directory (see the [documentation](https://www.zotero.org/support/zotero_data)).
[^debug]:  If you're used to using `console.log()` to debug your JS code, you can use `Zotero.debug()` instead in Scaffold to get about the same result.
  For instance, by adding `Zotero.debug(listTxt)` to your code and clicking the "Run doWeb" button, the word "git" should appear in the left panel.
[^shorttitle]: For some reason related to bibliography management in LateX, I keep the name of the email archive in both `shortTitle` and `libraryCatalog`.
  That's a personal hack that you can (and maybe should) get rid of.

