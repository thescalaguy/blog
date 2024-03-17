---
title: Setting up Sphinx Documentation
tags:
  - architecture
date: 2024-03-17 09:59:09
---


In one of my [previous blog posts](/2023/12/11/Software-Architecture-as-Code/) I had written about creating software architecture as code. The primary motivation behind it was to commit the architecture to version control, and keep it close to the source code. The added benefit, that perhaps remained implicit, is that architecture reviews can now happen as a part of the PR review. In the same spirit, I'd also like to keep the documentation close to the source code, and make documentation review a part of the PR review process. This post is about setting up Sphinx documentaion for a Flask microservice. Although everything mentioned in this post is Python-specific, the ideas can hopefully be applied to any language and framework of your choice.  

## Getting Started  

We'll build a simple Flask microservice. It receives an HTTP request, conducts some processing, and then invokes another microservice. For the purpose of this post, the processing is simply `time.sleep`, while the other microservice is HTTPBin. We'll then document our code and render it with Sphinx. This example, while contrived, is analogous to many real-world software projects in which numerous packages within the codebase are used in conjunction to express business logic. The goal of documentation, therefore, is to provide context to anyone working with the source. We will include documentation that gives a broad overview of the project, and covers each package in detail.  

### HttpBin  

We'll begin by adding classes which will let us make requests to HTTPBin.

{% code lang:python %}
class _HttpBinRequest(abc.ABC):

    base_url: str = "https://httpbin.org"

    @abc.abstractmethod
    def execute(self) -> Any:
        raise NotImplemented


@dc.dataclass
class HttpBinPost(_HttpBinRequest):
    params: dict[str, Any] = dc.field(default=dict)
    json: dict[str, Any] = dc.field(default=dict)

    def execute(self) -> Any:
        url = f"{self.base_url}/post"
        headers = {"Content-Type": "application/json"} if self.json else {}
        response = requests.post(
            url=url, 
            json=self.json, 
            params=self.params, 
            headers=headers,
        )
        response.raise_for_status()
        return response.json()
{% endcode %}  

An object of `HttpBinPost` class is a representation of a POST request to HTTPBin, and can include query params, and a JSON body as a part of the request. As you'd have noticed, there is no documentation in the code.  

We'll now add a blueprint which will accept incoming requests, and then make calls to HttpBin. 

{% code lang:python %}
@httpbin.post("/post")
def post():
    params = {"foo": "bar"}
    json = {"foo": "bar"}
    request = HttpBinPost(params=params, json=json)
    time.sleep(0.5)
    return request.execute()
{% endcode %}  

The `httpbin` blueprint configures a `/post` endpoint. We send some query parameters and a json body along with the POST request, and the response is returned directly to the caller. Finally, we make a curl call to the endpoint. 

{% code lang:bash %}
curl -s -XPOST localhost:5000/post | jq .
{% endcode %}  

To summarize, we have a codebase that includes a package for making requests to HttpBin, as well as an API endpoint that makes the request using this package. We will now look at Sphinx and how to document the codebase.  

### Sphinx  

Sphinx is a documentation generator that we'll use to generate HTML files from a directory of [reStructuredText](https://www.sphinx-doc.org/en/master/usage/restructuredtext/index.html) files. We'll also use the autodoc extension to generate documentaion from docstrings of Python classes and functions. The first step, however, is to install Sphinx. We'll do so by using pip.  

{% code lang:bash %}
pip install sphinx
{% endcode %}  

We'll now begin configuring Sphinx. We'll navigate to the `docs` directory, and run the `sphinx-quickstart` script that will run the interactive setup. While following along the setup, we'll make sure to separate the `build` and `source` directories. 

{% code lang:bash %}
cd docs/
sphinx-quickstart
{% endcode %}  

The `source` directory will contain the configuration file along with the rst files. The `build` directory will contain the HTML files that are generated when we build the documentation from the `source` directory. Let's take a quick look at the generated files and directories with `tree`.  

{% code lang:python %}
.
├── build
├── make.bat
├── Makefile
└── source
    ├── conf.py
    ├── index.rst
    ├── _static
    └── _templates
{% endcode %}  

We will update `conf.py` and add our repository to `sys.path`. This is required by the autodoc extension as it imports modules to be documented. We'll add the following lines to the bottom of the file. You will have to update the path appropriately.

{% code lang:python %}
# -- Add our repository to sys.path
from pathlib import Path
import sys

path = Path.home() / "Personal" / "sphinx"
sys.path.insert(0, str(path))
{% endcode %}  

We'll now generate documentation by running the `sphinx-build` command manually. Later we'll write a small bash script to automate the process of regenerating the documentation.   

{% code lang:bash %}
sphinx-build source/ build/html/
open build/html/index.html
{% endcode %}  

We'll get the following HTML page after rendering index.rst. It is extremely basic, and we will add to it to give an overview of the codebase. 

{% asset_img Sphinx-1.png %}  

We'll update the `conf.py` file and enable a couple of [extensions](https://www.sphinx-doc.org/en/master/usage/extensions/index.html) by adding them to the `extensions` array.  

{% code lang:python %}
extensions = ["sphinx.ext.autodoc", "sphinx.ext.napoleon"]
{% endcode %} 

The two extensions we've added allow us to include docstrings as a part of the documentation. For the sake of this post, we'll add lorem ipsum to the `HttpBinPost` class.  

{% code lang:python %}
@dc.dataclass
class HttpBinPost(_HttpBinRequest):
    """
    Lorem ipsum dolor sit amet, consectetur adipiscing elit.
    Fusce laoreet lectus neque, in congue purus dapibus et.
    Sed eros elit, luctus ac ante eget, fermentum imperdiet urna.
    Integer rutrum leo sed quam faucibus rutrum. Suspendisse nulla diam, rhoncus id nisi et, aliquet auctor risus.
    In pellentesque, orci quis molestie dignissim, dui massa posuere lorem, ut suscipit orci libero quis sem.
    Etiam ullamcorper turpis at tempus semper.
    Nunc odio massa, feugiat quis sem nec, hendrerit pretium ex.
    Integer varius volutpat interdum.
    """
{% endcode %}  

We'll now create a new rst file called `httpbin.rst` in the `source` directory. This will contain the overview of the HTTPBin module, and a couple of directives to include the module, and the `HttpBinPost` class as a part of the documentation.  

{% code %}
HttpBin Module
==============

Lorem ipsum dolor sit amet, consectetur adipiscing elit.
Fusce laoreet lectus neque, in congue purus dapibus et.
Sed eros elit, luctus ac ante eget, fermentum imperdiet urna.
Integer rutrum leo sed quam faucibus rutrum. Suspendisse nulla diam, rhoncus id nisi et, aliquet auctor risus.
In pellentesque, orci quis molestie dignissim, dui massa posuere lorem, ut suscipit orci libero quis sem.
Etiam ullamcorper turpis at tempus semper.
Nunc odio massa, feugiat quis sem nec, hendrerit pretium ex.
Integer varius volutpat interdum.

.. automodule:: service.httpbin
.. autoclass:: service.httpbin.HttpBinPost
{% endcode %}  

Finally, we'll update the `index.rst` file. This is where we'll provide the overview of the codebase, and link the `httpbin.rst` file.  

{% code %}
Welcome to Sphinx's documentation!
==================================
Lorem ipsum dolor sit amet, consectetur adipiscing elit.
Fusce laoreet lectus neque, in congue purus dapibus et.
Sed eros elit, luctus ac ante eget, fermentum imperdiet urna.
Integer rutrum leo sed quam faucibus rutrum. Suspendisse nulla diam, rhoncus id nisi et, aliquet auctor risus.
In pellentesque, orci quis molestie dignissim, dui massa posuere lorem, ut suscipit orci libero quis sem.
Etiam ullamcorper turpis at tempus semper.
Nunc odio massa, feugiat quis sem nec, hendrerit pretium ex.
Integer varius volutpat interdum.

.. toctree::
   :maxdepth: 2
   :caption: Contents:

   httpbin.rst

...
{% endcode %}  

The generated documentation now has an overview on the index page, and a link to the HttpBin module. When we click the link to the module, we'll see that the overview, as well as the docstring of the `HttpBinPost` class, are included in the documentation. Screenshots of both of these pages are provided below.

{% asset_img Sphinx-2.png index %}
{% asset_img Sphinx-3.png httpbin %}  

We'll now add a small bash script, called `docs.sh`, to regenerate the documentation for the codebase, and place it at the root of the codebase.  

{% code lang:bash %}
#!/bin/bash

sphinx-build docs/source/ docs/build/html/
open docs/build/html/index.html
{% endcode %}  

## Conclusion  

This post is a basic introduction to using Sphinx to generate documentation. Keeping the documentation within the repository allows keeping the context close to the source. Combining this post with the previous post on architecture as code, we can keep most of the context of a repository within itself. I find this to be more helpful than using Confluence or Notion. Finally, all of the code for this post is available as a [GitHub repository](https://github.com/thescalaguy/sphinx).