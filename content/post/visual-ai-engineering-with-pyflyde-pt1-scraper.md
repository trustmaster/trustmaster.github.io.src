---
title:       "AI Engineering Goes Visual: Web Scraping & Data Prep with PyFlyde"
subtitle:    "Learn how to create a web scraper and local article database using PyFlyde, BeautifulSoup4, and Playwright"
description: "In this tutorial, we explore how to use Flyde, a visual programming tool, to build a web scraper that feeds into a Retrieval Augmentation Generation (RAG) system. We will cover the process of scraping web content and storing it locally, setting the stage for more advanced AI engineering tasks."
date:        2025-02-22
tags:        ["fbp", "data", "python"]
categories:  ["Flow-based Programming" ]
draft:       false
---

## Introduction

Building data pipelines often involves integrating multiple tools and services into complex workflows. While traditional programming approaches are effective, visual tools can offer a more intuitive and streamlined way to design and manage these workflows.

In this tutorial, we will explore how to use [Flyde](https://flyde.dev), a visual programming tool, to construct a web scraper that feeds into a Retrieval Augmentation Generation (RAG) system. This tutorial aims to go beyond basic examples to demonstrate a more complex application, providing insights into the advantages and trade-offs of visual programming.

Our tutorial is divided into two parts. In Part 1 (this article), we will focus on building the web scraper using Flyde and Python. In [Part 2](/post/visual-ai-engineering-with-pyflyde-pt2-rag), we will extend the project by transforming the scraped content into a RAG system, utilizing LangChain, a vector database, LLaMA3 or OpenAI, and Flyde to integrate the components.

By the end of this tutorial, you will have a comprehensive understanding of how to leverage visual programming tools to build and manage modern data and AI pipelines.

## Prerequisites for this tutorial part

Here is what you will need today:

- Python 3.10 or newer
- Basic Python programming experience
- Familiarity with web concepts (HTML, HTTP requests)
- Visual Studio Code
- Basic understanding of web scraping (helpful but not required)

Some useful links to the tools and libraries we are going to use:

- [Flyde](https://flyde.dev) - visual programming tool and VSCode extension.
- [PyFlyde](https://github.com/trustmaster/pyflyde) - Python library and runtime for Flyde.
- [BeautifulSoup4](https://www.crummy.com/software/BeautifulSoup/) - Python library for web scraping.
- [Playwright](https://playwright.dev/) - browser automation for end-to-end testing and more.
- [Markdownify](https://pypi.org/project/markdownify/) - Markdown conversion library for Python.

## The concept of our app

As an Engineering Manager, I've found the [Software Leads Weekly](https://softwareleadweekly.com/) newsletter invaluable. Often, while mentoring other EMs, I think, "What was that article about this topic?". Sometimes I find it, sometimes not.

This inspired our project: we'll build a Software Leads Weekly index (SWLWi) with an LLM interface. The complete system will have four components:

1. **Scraper** (part 1) - fetches the newsletter issues and retrieves the articles from third-party blogs referenced in those issues. It converts each article to Markdown and then cleans it up to reduce noise in our data. The result is a local catalog of Markdown files containing the articles from the newsletter.
2. **Indexer** (part 2) - creates a searchable index of tokens generated from the scraped articles in a Vector Store. It splits the articles into smaller chunks, converts them to tokens that an LLM can understand, and saves them in a Vector database.
3. **RAG** (part 2) - receives a query/prompt, searches the Vector Store for matching token sequences, and provides them as context along with the prompt to a LLaMA3.
4. **UI** (part 2) - a simple Streamlit app that wraps our RAG and allows a user to query the database and get responses.

The resulting application is available in a [SWLWi Github repository](https://github.com/trustmaster/swlwi).

This might be a very niche application, so consider this as a tutorial for building similar applications for whatever domain and data source you might be interested in. Our goal here is to learn how to build simple applications around LLMs and text processing, and how visual tools like Flyde can help make this task more comprehensive.

## Understanding Flow-based Programming and Flyde

Before diving into implementation, let's understand the key concepts we'll be using.

### What is Flow-based Programming?

Flow-based Programming (FBP) is a programming paradigm that defines applications as networks of "black box" processes exchanging data across predefined connections. Think of it as building with Lego blocks - each block has a specific function, and you connect them to create something larger.

Learn more about Flow-based Programming from [What is Flow-based Programming](https://blog.kodigy.com/post/state-of-flow-based-programming/#what-is-flow-based-programming) section in my blog.

### Key concepts in Flyde

[Flyde](http://flyde.dev) is a modern implementation of FBP that provides a [Visual Studio Code extension](https://marketplace.visualstudio.com/items?itemName=flyde.flyde-vscode) to edit the diagrams in your project. It is built in TypeScript and for TypeScript by default. We will Flyde's VSCode extension in combination with the [PyFlyde](https://github.com/trustmaster/pyflyde) Python runtime to write and run our Flyde code with Python.

Here are the core concepts of Flyde that we will use in this article:

{{< figure src="/img/post/pyflyde/flyde-concepts.png" alt="Core Concepts of Flyde overview diagram" class="w50" >}}

- **Node** is the main building block. Everything consists of nodes that process data and pass it along. There are two types of Nodes:
  - __Code Node__ (aka __Component__ in PyFlyde and FBP) - nodes that are defined in a textual programming language like Python, TypeScript, etc.
  - __Visual Node__ (aka __Graph__ in PyFlyde and FBP) - nodes that consist of other nodes and connections, defined as Flyde flows. Visual Nodes can contain other Visual Nodes, and so on, allowing you to use as many layers of nesting and reuse your flows as needed.
- **Inputs and outputs** are ports that connect the nodes with the world around them.
- **Connection** connects outputs to inputs and wires the Nodes together. Typically, it acts like a queue, delivering messages one by one.
- **Flow** is a visual representation of an application or its part. A Flow consists of Nodes wired together with Connections. A Flow can have its own Inputs and Outputs as well, so that it can be used from the outside.

### Built-in features

Flyde comes with a standard library of commonly used low-level nodes and an extended library of components for TypeScript. PyFlyde currently implements only a minimal subset of standard nodes out of the box:

- `InlineValue` is a node that outputs a static value. It is typically used to kick-start the application or set up configuration variables.
- `Conditional` is a simple logical branching node that routes the input based on a boolean condition.
- `GetAttribute` is used to extract an attribute from an object.

Another limitation of PyFlyde is that the runtime currently only works from the CLI; it doesn't have runtime integration with the visual editor that TypeScript code in Flyde has. Additionally, for the editor to recognize your components, an extra step is required to generate TypeScript stubs. Don't worry, we'll cover these bits below.

### Why use Flyde for web scraping?

Traditional web scraping code can become complex and hard to maintain. Flyde offers several advantages:

1. Visual representation makes the data flow clear
2. Easy to modify and extend the pipeline
3. Natural parallelization of tasks

## Project setup

### Creating the project structure

Let's set up our project with a clean structure:

```bash
swlwi
├── .venv                        # Virtual environment for Python
├── swlwi
│   ├── __init__.py
│   ├── <our Python modules>
│
├── tests
│   ├── test_<module_name>.py
│
├── index
│   ├── issue-<issue number> /   # Subfolder for newsletter issue
│   │   ├── article-<number>.md  # Scraped article files
│   ├── vectors /                # Folder for the vector database files
│
├── pyproject.toml               # Python project setup and deps
├── Makefile                     # Makefile
├── <root-level Python files>
├── <Flyde flow files>
```

### Setting up dependencies

Create a `pyproject.toml` with our dependencies:

```toml
[build-system]
# This part is needed to install and manage dependencies
requires = ["setuptools", "setuptools-scm"]
build-backend = "setuptools.build_meta"

[project]
# General project setup
name = "swlwi"
version = "0.0.1"
requires-python = ">= 3.10"
authors = [{ name = "Your name here" }]
description = "Software Leads Weekly Intelligent Index"

# All our dependencies are listed here
dependencies = [
    "beautifulsoup4",               # HTML parsing
    "markdownify",                  # HTML to Markdown conversion
    "playwright",                   # JavaScript-enabled scraping
    "pyflyde >= 0.0.11",            # Flyde runtime for Python
    "requests",                     # Basic HTTP requests
]

[tool.setuptools.packages.find]
# Tells Python where to find our local package
where = ["swlwi"]
```
### Creating the environment

Set up a Python virtual environment to isolate the dependencies for our project:

```bash
python3 -m venv .venv     # Create a virtual environment
source .venv/bin/activate # On Windows: .venv\Scripts\activate
pip install .             # Install the dependencies
```

## Building the Scraper

### Developer workflow with Flyde

When building applications with Flyde, follow this workflow:

1. Design the high-level flow
2. Create component interfaces
3. Build the visual flow
4. Implement component behavior
5. Test and iterate

This approach helps catch design issues early and makes the development process more structured.

### Designing and implementing our first Flyde flow

Let's start with a simple flow that handles basic web pages:

{{< figure src="/img/post/pyflyde/scrape_v1.flyde.avif" alt="Scrape.flyde version 1" class="w60" caption="The first scraper flow" >}}

Following our developer workflow defined above, at first just define the interface parts of the nodes on the graph, and then complete the behavior implementation.

#### Defining data structures

First, let's define our core data structures in `scrape.py`:

```python
@dataclass
class Issue:
    """Represents a newsletter issue"""

    num: int
    url: str
    date: date
    item_of: tuple[int, int] = (0, 0)  # (current, total)


@dataclass
class Article:
    """Represents an article within an issue"""

    title: str
    url: str
    issue_num: int = 0
    reading_time: int = 0  # minutes
    summary: str = ""
    html: bytes = b""
    markdown: str = ""
    item_of: tuple[int, int] = (0, 0)  # (current, total)
    parent_item_of: tuple[int, int] = (0, 0)  # parent issue's item_of
```

These data structures are all we need: Issues and Articles. The `item_of` and `parent_item_of` tuples might be a bit tricky. They act as pointers, like _"issue 3 out of 100"_ and _"article 5 out of 7"_. This is useful for operations like fan-out and fan-in, so the fan-in component knows which items in a batch it has received and how many more to expect.

#### Defining Component interfaces

We've skipped the imports section of our scrape module in the previous snippet. Here it is with some more imports that we are going to use below:

```python
from dataclasses import dataclass # needed for our data structures
from datetime import date         # needed for Issue.date

# PyFlyde imports of core classes
from flyde.io import Input, InputMode, Output, Requiredness
from flyde.node import Component
```

For now, let's code all the nodes of the above flow as black boxes with just inputs and outputs:

```python
class ListIssues(Component):
    """Fetches the index by the URL, parses it and returns a stream of issues and their URLs."""

    inputs = {
        "url": Input(description="URL of the index page", type=str),
        "limit": Input(description="Limit the number of issues to fetch",
            type=int, mode=InputMode.STICKY),
    }

    outputs = {
        "issue": Output(description="List of issues", type=Issue),
    }

    def process(self, url: str, limit: int):
        pass


class ExtractArticles(Component):
    """Extracts articles from the issue page."""

    inputs = {
        "issue": Input(description="Issue", type=Issue),
    }

    outputs = {
        "article": Output(description="Article", type=Article),
    }

    def process(self, issue: Issue):
        pass


class FetchArticle(Component):
    """Fetches the article HTML from the Internet."""

    inputs = {
        "article": Input(description="Article", type=Article),
    }

    outputs = {
        "complete": Output(description="Article with HTML content", type=Article),
    }

    def process(self, article: Article):
        pass


class ExtractArticleContent(Component):
    """Extracts the article content from the HTML and converts it to Markdown."""

    inputs = {
        "article": Input(description="Article", type=Article),
    }

    outputs = {
        "article": Output(description="Article with content", type=Article),
    }

    def process(self, article: Article):
        pass


class SaveArticle(Component):
    """Saves the article to a file."""

    inputs = {
        "article": Input(description="Article", type=Article),
        "path": Input(description="Path prefix to save the article",
            type=str, mode=InputMode.STICKY),
    }

    def process(self, article: Article, path: str):
        pass
```

As you can see, each PyFlyde Component defines its `inputs` and `outputs` as dictionaries, mapping the names to `Input` or `Output` objects from the `flyde.io` module. All inputs/outputs in our examples have optional descriptions and type hints.

What does this `mode=InputMode.STICKY` argument do? By default, inputs in Flyde act as FIFO queues. What `InputMode.STICKY` does is make the input remember the last value it received from the queue and not block its execution until the next value arrives. This is useful when working with Inline Values, for example, as we only receive them once and keep using them with other input data. We'll see this feature in use when we implement the full `process()` code.

#### Adding nodes to the flow

Although some of our nodes don't do anything yet, it is becoming clearer what they are going to do and, more importantly, how they are going to be used together. Before we add them to a visual Flyde flow, we need to generate TypeScript stubs:

```bash
pyflyde gen swlwi/scrape.py
```

Then we can go to VSCode, press `Cmd + Shift + P`, and choose the `Flyde: New Visual Flow` command to create a new flow diagram. Let's call it `Scrape.flyde`.

In the Flyde node toolbar on the right side, we need two items for now:

{{< figure alt="Flyde node selection toolbar" src="/img/post/pyflyde/flyde-nodebar-1.avif" class="w40" >}}

We need to add our constant values for the URL, issues limit, and directory path using the `Essentials` -> `Inline Value` button.

For the first one, enter `https://softwareleadweekly.com/issues/` as the value, click `Change type`, and select `String` in the dropdown. For the limit of the number of issues, add another Inline Value, enter `20` as the value, and choose `Number` as its type. Now you know how to add constants to the diagram, so you can add them whenever needed like this.

{{< figure alt="Flyde node selection toolbar" src="/img/post/pyflyde/flyde-nodebar-2.avif" class="w40" >}}

Adding the nodes that we defined in our Python code above requires hitting that small `View all` button at the bottom right of the toolbar. It opens a node browser window:

{{< figure src="/img/post/pyflyde/flyde-add-node.avif" alt="Add node dialog in Flyde editor" class="w75" >}}

Select our `swlwi/scrape.flyde.ts` under the _Current Project_, and add the nodes by selecting them in the right panel and clicking the _Add_ button. Add all the necessary nodes to the flow and drag them to the desired positions on the canvas.

When you hover over a node, all of its inputs and outputs pop up. To connect two nodes, click on the output of the first node and then on the input on the second node (or vice versa, the order does not matter here). To connect an Inline Value with an input, hover over it, and connect its `value` output with a target input.

This way, you can draw the whole flow and get a `Scrape.flyde` diagram that looks like the picture we already saw above:

{{< figure src="/img/post/pyflyde/scrape_v1.flyde.avif" alt="Scrape.flyde version 1" class="w60" >}}

Yay! We've created our first Flyde flow!

Our flow doesn't do anything yet, so we need to add behavior logic next.

### Implementing the node logic

The `ListIssues` and `ExtractArticles` contain a lot of HTML parsing with BeautifulSoup, which are very specific to the website that we are scraping. If you are writing a scraper for a different site, the implementation is going to be very different.

#### ListIssues component

{{< figure src="/img/post/pyflyde/scrape_list_issues.avif" alt="ListIssues component view in Flyde" class="w25" >}}

Below is the main logic of the `ListIssues` component. The HTML parsing logic using BeautifulSoap is omitted. You can find the full source in [scrape.py](https://github.com/trustmaster/swlwi/blob/main/swlwi/scrape.py).

```python
class ListIssues(Component):
    """Fetches the index by the URL, parses it and returns a stream of issues and their URLs."""

    inputs = {
        "url": Input(description="URL of the index page", type=str),
        "limit": Input(description="Limit the number of issues to fetch",
            type=int, mode=InputMode.STICKY),
    }

    outputs = {
        "issue": Output(description="List of issues", type=Issue),
    }

    def process(self, url: str, limit: int):
        # Get the page content using the requests library
        response = requests.get(url)

        # Parse the HTML content using BeautifulSoup
        soup = BeautifulSoup(response.content, "html.parser")
        issue_elements = self._find_issue_elements(soup)
        total_issues = len(issue_elements)
        item_count = 0

        # Get the base URL
        base_url = "/".join(url.split("/")[:3])

        # Process each issue
        for element in issue_elements:
            issue = self._parse_issue_element(element, base_url, item_count, total_issues)

            # Send the issue to the next component
            self.send("issue", issue)

            # Increment the item count and exit if the limit is reached
            item_count += 1
            if limit > 0 and item_count >= limit:
                break
```

The `process()` method is invoked when the node receives a new `url` value. Since `limit` input is _sticky_ and it has received an `InlineValue`, it keeps using just the same value with different URLs.

After fetching and parsing the newsletter index page, we find all issue elements on it. For each issue found we send a new `Issue` object to the output using `self.send(output_name, value)` method.

#### ExtractArticles component

{{< figure src="/img/post/pyflyde/scrape_extract_articles.avif" alt="ExtractArticles component view in Flyde" class="w25" >}}

The `ExtractArticles` logic is going to be similar to the previous component and it is also actively using BeautifulSoup.

```python
class ExtractArticles(Component):
    """Extracts articles from the issue page."""

    inputs = {
        "issue": Input(description="Issue", type=Issue),
    }

    outputs = {
        "article": Output(description="Article", type=Article),
    }

    def process(self, issue: Issue):
        # Get the page content and parse it
        response = requests.get(issue.url)
        soup = BeautifulSoup(response.content, "html.parser")

        # Find the topic sections
        topic_sections = self._find_topic_sections(soup)
        # Count the total number of articles across all sections
        total_articles = self._count_total_articles(topic_sections)

        # Loop through each topic section and extract articles
        item_count = 0
        for section in topic_sections:
            article_divs = section.find_next_siblings("div")
            for div in article_divs:
                article = self._extract_article(div, issue, item_count, total_articles)
                if article:
                    self.send("article", article)
                    item_count += 1
```

This component is also loop-based. It fetches and parses an issue page. Then it loops over all sections in the issue to find and extract article entries, which it sends to the output as it extracts them successfully.

See the full source in the [scrape.py on Github](https://github.com/trustmaster/swlwi/blob/main/swlwi/scrape.py).

#### FetchArticle component

{{< figure src="/img/post/pyflyde/scrape_fetch_article.avif" alt="FetchArticle component view in Flyde" class="w25" >}}

The `FetchArticle` is generic enough, so let's write the complete code:

```python
class FetchArticle(Component):
    """Fetches the article HTML from the Internet."""

    inputs = {
        "article": Input(description="Article", type=Article),
    }

    outputs = {
        "complete": Output(description="Article with HTML content", type=Article),
    }

    def process(self, article: Article) -> dict[str, Article]:
        try:
            response = requests.get(article.url)
            article.html = response.content

        except Exception as e:
            logger.error(f"Failed to fetch article '{article.title}' at {article.url}: {e}")
            # We still want to save at least the metadata
            return {"complete": article}

        # Return the article
        return {"complete": article}
```

In this component, we simply use the `requests.get()` function to fetch the remote page and add its HTML contents to the article object. In case of an error, it still passes the article along with an empty body.

In this case, the `process()` signature includes a return value of type `dict`:

```python
def process(self, article: Article) -> dict[str, Article]:
    article.html = fetch_html(article.url)
    return {"complete": article}
```

The returned `dict` maps an output name to a value that needs to be sent to that output. If a node has multiple outputs, you can return all of them at once, or just a subset. This kind of signature is common for nodes that output one value per each input.

#### ExtractArticleContent component

{{< figure src="/img/post/pyflyde/scrape_extract_article_content.avif" alt="ExtractArticleContent component view in Flyde"  class="w25" >}}

The next component in our scraper module is `ExtractArticleContent`.

```python
class ExtractArticleContent(Component):
    """Extracts the article content from the HTML and converts it to Markdown."""

    inputs = {
        "article": Input(description="Article", type=Article),
    }

    outputs = {
        "article": Output(description="Article with content", type=Article),
    }

    def process(self, article: Article) -> dict[str, Article]:
        if not article.html:
            logger.error(f"No HTML content found for article '{article.title}' at {article.url}")
            return {"article": article}

        markdown = html_to_markdown(article.html)

        # Clean up the Markdown content
        markdown = clean_markdown(markdown)

        # Update the article with the Markdown content
        article.markdown = markdown

        # Return the article
        return {"article": article}
```

The implementation of this `process` makes use of `markdownify` package and our simple cleaner functions located in [parser.py](https://github.com/trustmaster/swlwi/blob/main/swlwi/parser.py) module.

#### SaveArtile component

{{< figure src="/img/post/pyflyde/scrape_save_article.avif" alt="SaveArticle component view in Flyde" class="w25" >}}

Let's implement `SaveArtile` which saves the result to the disk, as this is quite standard Python code:

```python
class SaveArticle(Component):
    """Saves the article to a file."""

    inputs = {
        "article": Input(description="Article", type=Article),
        "path": Input(description="Path prefix to save the article", type=str,
            mode=InputMode.STICKY),
    }

    def process(self, article: Article, path: str):
        # Save the article to a file
        dir_name = f"{path}/issue-{article.issue_num}"
        os.makedirs(dir_name, exist_ok=True)
        filename = f"{dir_name}/article-{article.item_of[0]}.md"

        logger.info(f"Saving article '{article.title}' to {filename}")

        header = f"""# {article.title}

Source: [{article.url}]({article.url})
Reading time: {article.reading_time} minutes

{article.summary}

---

"""

        with open(filename, "w") as file:
            file.write(header + article.markdown)
```

Note that this component doesn't have any output ports. It acts as a sink node, so it doesn't need to output anything.

In this component we construct the directory path for the issue and create it if it doesn't exists yet. Then we create the path string of the article itself. Note that the prefix for the directory path comes from a sticky input that is configured with inline value.

For each article we always save the header containing its title, URL, reading time, and summary. If markdown body is non-empty, it follows the header.

In case you're looking for the full list of imports that we need so far, here it is:

```python
import logging
import os
import re
from dataclasses import dataclass
from datetime import date, datetime

import requests
from bs4 import BeautifulSoup
from bs4.element import PageElement

from flyde.io import Input, InputMode, Output, Requiredness
from flyde.node import Component
from swlwi.parser import clean_markdown, html_to_markdown
```

### Running the flow

Once you have your flow wired up and ready, you can run it with PyFlyde:

```bash
pyflyde Scrape.flyde
```

The logging we added in the code will output something like this:

```bash
INFO:swlwi.scrape:Processing issue #633
INFO:swlwi.scrape:Processing issue #632
INFO:swlwi.scrape:Processing issue #631
INFO:swlwi.scrape:Processing issue #630
...
INFO:swlwi.scrape:Found article 'AI and Cognitive Offloading: Sharing the Thinking Process With Machines' at https://uxdesign.cc/ai-and-cognitive-offloading-sharing-the-thinking-process-with-machines-2d27e66e0f31 in issue #633. Reading time: 11 minutes.
INFO:swlwi.scrape:Found article '“Yes I Feel Fully Recharged After a Restful Holiday”' at https://x.com/mattturck/status/1876256566963568660 in issue #633. Reading time: 1 minute.
INFO:swlwi.scrape:Found article 'What Happens When AI Joins the Org Chart?' at https://every.to/working-overtime/what-happens-when-ai-joins-the-org-chart in issue #633. Reading time: 7 minutes.
INFO:swlwi.scrape:Found article 'Explore vs Execute' at https://longform.asmartbear.com/explore-execute/ in issue #633. Reading time: 9 minutes.
...
INFO:swlwi.scrape:Fetched article 'Leveraging AI for Efficient Incident Response' at https://engineering.fb.com/2024/06/24/data-infrastructure/leveraging-ai-for-efficient-incident-response/
INFO:swlwi.scrape:Extracted article content from 'Leveraging AI for Efficient Incident Response' at https://engineering.fb.com/2024/06/24/data-infrastructure/leveraging-ai-for-efficient-incident-response/
INFO:swlwi.scrape:Saving article 'Leveraging AI for Efficient Incident Response' to ./index/issue-614/article-4.md
ERROR:swlwi.scrape:No HTML content found for article 'Action Is a Skill. How to Become High-Action' at https://x.com/ShaanVP/status/1827777391294308640
INFO:swlwi.scrape:Saving article 'Action Is a Skill. How to Become High-Action' to ./index/issue-614/article-6.md
```

Apart from some errors from Twitter and YouTube, there is another problem. If we skim through the Markdown files, we will notice that some of the contents look like:

```markdown
Enable JavaScript and cookies to view this page
```

Oops, we forgot that some Single Page Applications (SPA) are not so easy to access these days. No worries, there are tools that make it easy.

### Adding JavaScript support

To address the problem of fetching pages that require JavaScript to render, we are going to make the following modifications to our scraper:

{{< figure src="/img/post/pyflyde/scrape_v2.flyde.avif" alt="Scrape.flyde version 2" class="w60" caption="Modified Scrape.flyde flow with SPA handling" >}}

1. Modify `FetchArticle` to check if the response HTML contains errors related to JavaScript or cookies. Create an alternative output path if the article requires JavaScript to fetch. We keep trying to fetch a page with just `requests` before switching to Playwright to keep our scraper fast by default.
2. Create a new node called `FetchArticleWithJavascript` which fetches the article HTML using Playwright. We are only going to use it for pages that failed plain request retrieval because Playwright uses headless Chromium under the hood and works much slower.
3. Create a new branch in the flow that uses `FetchArticleWithJavascript` before parsing the page.

#### Installing Playwright

Playwright is a tool that allows you to automate browsers. It is primarily used to test frontend code, but it is also a great tool for scraping pages that require JavaScript to render.

Let's install the Playwright package in our environment and download the browser binaries for Playwright:

```bash
pip install playwright
playwright install
```

#### Checking for JavaScript errors in the page HTML

We'll send the `Article` object to a second output called `needs_javascript` instead of `complete` if a JavaScript problem is detected. A simple regex check in the HTML body should do the trick.

Here is the complete updated code for `FetchArticle`. See comments for the new code:

```python
class FetchArticle(Component):
    """Fetches the article HTML from the Internet."""

    inputs = {
        "article": Input(description="Article", type=Article),
    }

    outputs = {
        "complete": Output(description="Article with HTML content", type=Article),
        # ADDED: Second input to send the article to if it requires JS
        "needs_javascript": Output(description="Article that needs JavaScript to fetch", type=Article),
    }

    def process(self, article: Article) -> dict[str, Article]:
        try:
            response = requests.get(article.url)
            article.html = response.content

            # ADDED: Check if the article needs JavaScript to fetch
            if re.findall(r"enable.+javascript", response.text, re.IGNORECASE):
                logger.debug(f"Article '{article.title}' at {article.url} needs JavaScript to fetch")
                return {"needs_javascript": article}

        except Exception as e:
            logger.error(f"Failed to fetch article '{article.title}' at {article.url}: {e}")
            # We still want to save at least the metadata
            return {"complete": article}

        # Return the article
        return {"complete": article}
```

#### Fetching the page contents with Playwright

Next, we create a separate component that fetches an article using Playwright.

{{< figure src="/img/post/pyflyde/scrape_fetch_article_with_javascript.avif" alt="FetchArticleWithJavascript component view in Flyde" class="w25" >}}

The implementation is similar to the original `FetchArticle`, but we use the `sync_playwright` API to launch a Chromium instance, navigate to the article URL, and retrieve the page contents.

```python
from playwright.sync_api import sync_playwright # add this to the imports at the top


class FetchArticleWithJavaScript(Component):
    """Fetches the article SPA contents using Playwright."""

    inputs = {
        "article": Input(description="Article", type=Article),
    }

    outputs = {
        "article": Output(description="Article with content", type=Article),
    }

    def process(self, article: Article) -> dict[str, Article]:
        try:
            # With block cleans up resources when done
            with sync_playwright() as p:
                # Open the URL in a new tab in headless Chromium
                browser = p.chromium.launch(headless=True)
                page = browser.new_page()
                page.goto(article.url)

                # Wait for the article content to load
                page.wait_for_selector("body")

                # Extract the page content
                page_content = page.content()
                browser.close()

                article.html = page_content.encode("utf-8")

        except Exception as e:
            logger.error(f"Failed to fetch '{article.title}' at {article.url}: {e}")
            return {"article": article}

        # Return the article
        return {"article": article}
```

We need to update TypeScript stubs after adding new nodes:

```bash
pyflyde gen swlwi/scrape.py
```

#### Updating the flow

Updating the visual flow is as simple as connecting the new `needs_javascript` output of `FetchArticle` to the newly added `FetchArticleWithJavascript` node, which then outputs to the same `ExtractArticleContents` node. Refer to the screenshot above for guidance.

Now we can run our flow again:

```bash
pyflyde Scrape.flyde
```

### Improving speed and robustness of the scraper

We could stop with our scraper as it is probably good enough at this point. However, with resources like Medium, we are hitting a rate limit if we run the flow as is. So, we might want to add a rate limiter to our script to avoid that. Another simple improvement we can make is to skip issues that have already been fetched. That's what we will do next.

#### Skipping issues that have already been fetched

The new node we are adding here is called `SkipExistingIssues`, and in the graph, it looks like this:

{{< figure src="/img/post/pyflyde/scrape_skip_existing_issues.avif" alt="SkipExistingIssues component view in Flyde" class="w25" >}}

This node checks if the issue already exists in the directory under a given path. If it does, the processing of the issue stops at this stage.

```python
class SkipExistingIssues(Component):
    """Checks if an issue already exists in the index and skips it if it does."""

    inputs = {
        "issue": Input(description="Issue", type=Issue),
        "path": Input(description="Path to the index", type=str, mode=InputMode.STICKY),
        "force_all": Input(
            description="Force processing all issues",
            type=bool,
            mode=InputMode.STICKY,
            value=False,
            required=Requiredness.REQUIRED_IF_CONNECTED,
        ),
    }

    outputs = {
        "issue": Output(description="Issue", type=Issue),
    }

    def process(self, issue: Issue, path: str, force_all: bool = False):
        if force_all:
            # Ignore the cache and force re-fetching the issue
            self.send("issue", issue)
            return

        # Check if the issue exists
        issue_path = os.path.join(path, f"issue-{issue.num}")
        if not os.path.exists(issue_path):
            # No cache found, proceeding
            self.send("issue", issue)
        else:
            # The issue already exists, skipping
            pass
```

What is interesting about this component is that we used the combination of `mode=InputMode.STICKY`, `value=False`, and `required=Requiredness.REQUIRED_IF_CONNECTED` to make the `force_all` input completely optional, defaulting to a `False` value. If connected to some output, it will use the values from that output. We will leave it with the default in our flow.

After adding new components we need to update the Flyde TypeScript stubs:

```bash
pyflyde gen swlwi/scrape.py
```

Then we can update `Scrape.flyde` in VSCode. Here is the scraper flow after adding this new node to it:

{{< figure src="/img/post/pyflyde/scrape_v3.flyde.avif" alt="Scrape.flyde version 3" class="w60" >}}

#### Adding a rate limiter

If we run our flow as it is, we would be hitting the websites at the quickest rate possible, and some of them wouldn't be happy about that, especially ones like Medium and Substack, which we have quite a few links to. To address this problem, we can implement a simple rate limiter and use it in our fetcher nodes.

Here is the `RateLimiter` implementation that I went for:

```python
class _RateLimiter:
    """Singleton class to handle rate limiting per domain."""

    _instance = None
    _last_request_time: dict[str, datetime] = {}
    _timeout: float = 1

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super(_RateLimiter, cls).__new__(cls)
        return cls._instance

    def wait(self, domain: str):
        now = datetime.now()
        if domain in self._last_request_time:
            elapsed = (now - self._last_request_time[domain]).total_seconds()
            if elapsed < self._timeout:
                sleep_time = self._timeout - elapsed
                logger.debug(f"Rate limiting: sleeping for {sleep_time:.2f} seconds before fetching from {domain}")
                time.sleep(sleep_time)
        self._last_request_time[domain] = datetime.now()
```

It's a private class since we don't want to export it outside of the scraper module. It contains `_instance`, which holds the only instance of the class, and the `__new__()` constructor implements the singleton pattern by making sure we reuse an existing `_instance` instead of creating a new one.

The main work is happening in the `wait()` method. It tracks `_last_request_time` for each domain, and if the current `domain` was last requested less than `_timeout` seconds ago, it makes the script sleep for the remainder of that time.

To check which domain the URL belongs to, we use a simple regex function:

```python
def _extract_domain_from_url(url: str) -> str:
    m = re.findall(r"https?://(?:[^./]+\.)*([^./]+\.[a-zA-Z][^./:]+)(?:/|$|:)", url)
    return m[0] if m else "unknown"
```

Of course, this is a very simple implementation of rate limiting per domain, and if you're serious about your scraping, you may want to improve it. For the sake of our tutorial, we move on to applying this limiter to our fetchers.

In the `FetchArticle` class, we instantiate the rate limiter as a class variable:

```python
# Adding it right after the outputs definition
outputs = {
    "complete": Output(description="Article with HTML content", type=Article),
    "needs_javascript": Output(description="Article that needs JavaScript to fetch", type=Article),
}

_rate_limiter = _RateLimiter()
# ...
```

And then we use it in the `process()`:

```python
logger.debug(f"Fetching article '{article.title}' at {article.url}")
# Extract the 2nd level domain from the URL
domain = _extract_domain_from_url(article.url)

if domain == "x.com" or domain == "youtube.com":
    # Ignore X/Twitter/Youtube as it does not allow scraping and we already have tweet summary
    return {"complete": article}

# Use the rate limiter to wait if necessary
self._rate_limiter.wait(domain)

# The rest of the code remains the same as before
response = requests.get(article.url)
# ...
```

We've added filtering for X/Twitter and YouTube since we are extracting a domain from the URL anyway, and these sites don't let us scrape the content.

The `FetchArticleWithJavascript` class can be modified in a similar way.

## Wrapping up the Scraper flow and Next steps

In this tutorial, we successfully developed a scraper for Software Leads Weekly, a curated newsletter focused on managing Software Engineering teams. Using PyFlyde, we created an efficient scraping workflow and gained valuable experience with the Flyde visual editor, key concepts, and the process of building and running nodes and flows in Python.

However, this is just the beginning. There are several ways to enhance and customize your scraper flow:

- _Use your data source_: replace the `ListIssues` and `ExtractArticles` components to target your specific sites or sources, making the scraper useful for various applications.
- _Robustness and conciseness_: add a retry mechanism and more scraping techniques to `FetchArticle` and `FetchArticleWithJavaScript`. You could also use a single node that employs Playwright from the start if the volume of pages to scrape is not high and scraping speed is not a concern.
- _Cleaner dataset_: enhance `parser.py` to clean up the scraped documents even better by preserving meaningful content and removing all remaining navigation elements, media links, decorative leftovers, etc. We didn't focus on this part in the article because it is not central to the data flow, but there is actually a lot of room for improvement there.

[In the second part of the tutorial](/post/visual-ai-engineering-with-pyflyde-pt2-rag), we will focus less on Flyde specifics and dive deeper on the AI engineering side. Here is what we will focus on:

1. **Creating a search index:** we will build a search index for our database and store it in a Vector Store.
2. **Implementing RAG:** we will implement Retrieval Augmented Generation (RAG) using our Vector Store and Ollama.
3. **Building a UI:** we will add a simple, user-friendly web-based UI for our application using Streamlit.

Can't wait to try the whole application out? Check out the [SWLWi Github repo](https://github.com/trustmaster/swlwi).

Thanks for your time and attention, and see you in the second part!
