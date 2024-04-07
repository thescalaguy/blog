---
title: Hosting Python Packages
tags:
  - architecture
date: 2024-03-22 22:04:29
---


In the [previous post](/2024/03/21/Setting-up-a-Python-microservice/) we saw how to create Python microservices. It's likely that these microservices will share code. For example libraries for logging, accessing the database, etc. In this post we'll see how to create and host Python packages.  

## Before We Begin

The setup is intentionally chosen for simplicity. We'll separate the common code into a repository of its own, and use Poetry to package it. We'll deploy it to S3, and use `dumb-pypi` to generate a static site which can be used to list the hosted packages.  

The setup can be expanded to accommodate larger organizations with more teams, but it is primarily intended for smaller organizations with fewer teams that need to ramp up quickly. The main idea behind package distribution for Python is to arrange the files in a particular directory structure that can be accessed by a web server. While I am utilizing S3, you are free to use any comparable technology. You may even host everything on virtual machines (VMs) that are placed behind a load balancer. 

## Getting Started

Let's start with a package called "common" which contains the OTel code we saw previously. In essence, we've simply copied the contents of package over into a new repository. Next we'll intialize a Poetry project and enter the relevant information interactively.

{% code lang:bash %}
poetry init --name="common" --description="A collection of utilities" --python="^3.12"
{% endcode %}

Next we'll add the relevant dependencies with Poetry. The first three are for the common code, and the fourth is for generating static pages.

{% code lang:bash %}
poetry add opentelemetry-sdk@latest
poetry add opentelemetry-api@latest
poetry add setuptools@latest
poetry add --group dev dumb-pypi@latest
{% endcode %}

Next we'll write a small bash script which will create the package, generate a static site to display hosted packages, and upload them to S3.

{% code lang:bash deploy.sh %}
#!/bin/sh
PROJECT="common"
BUCKET="... bucket name ..."
TEAM="devtools"

# -- Generate the package file for dumb-pypi
PACKAGE=$(poetry build | awk '{print $3}' | sed -n '3p')
echo "$PACKAGE" >> packages
sort -u packages -o packages

# -- Upload package
aws s3 cp "dist/$PACKAGE" "s3://$BUCKET/packages/$TEAM/$PROJECT/$PACKAGE"

# -- Generate static files for PyPi
dumb-pypi \
    --package-list packages \
    --packages-url "https://$BUCKET.s3.us-west-2.amazonaws.com/packages/$TEAM/$PROJECT" \
    --output-dir index

# -- Upload the files in S3
aws s3 cp --recursive index/ "s3://$BUCKET/dumb-pypi/$TEAM/index/"

# -- Open static pages in the browser
open "https://$BUCKET.s3.us-west-2.amazonaws.com/dumb-pypi/$TEAM/index/index.html"

# -- Serve static pages locally
# -- python -m http.server -b 0.0.0.0 -d index 8080
{% endcode %}

There's a lot going on in the script. First we build the package using Poetry on line #7. The output contains the name of the zip file and we extract that into a variable. This is then stored into a file on line #8 and will be used by `dumb-pypi` to generate the static files. We sort and deduplicate the packages on line #9. On line #12 we store copy the package to S3. On line #15 we generate the static files into a folder named `index` which we then copy to S3 on line #21. On line #24 we open the documentation in the browser.  

The S3 bucket contains a directory structure as mentioned in the Python packaging docs.<sup>[1]</sup>

Before we run the script, however, we will have to configure the S3 bucket to be publicly accessible. We do so by enabling it to serve static content, and adding a policy which enables access to the content. The policy is given below.  

{% code lang:json %}
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "ReadIndex",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::yourbucketnamehere/*"
        }
    ]
}
{% endcode %}

Now we'll run the script.  

{% code lang;bash %}
./deploy.sh
{% endcode %}

The browser displays the following page.  

{% asset_img Package-1.png Dumb PyPi Homepage %}

We can now test the installation of the package with both Poetry and Pip. We will do this in a different Poetry project so that the install does not conflict with the existing "common" package.

{% code lang:bash %}
pip install --extra-index-url https://selfhostedpackages.s3.us-west-2.amazonaws.com/packages/devtools/ common

Looking in indexes: https://pypi.org/simple, https://selfhostedpackages.s3.us-west-2.amazonaws.com/packages/devtools/
Collecting common
  Using cached common-0.1.2.tar.gz (3.5 kB)
  Preparing metadata (setup.py) ... done
Building wheels for collected packages: common
  Building wheel for common (setup.py) ... done
  Created wheel for common: filename=common-0.1.2-py3-none-any.whl size=3708 sha256=51d2f21c15829e49375762f5ca246d7f0e4d0bc82c425b25b5e77fcc83e97eae
  Stored in directory: /home/fasih/.cache/pip/wheels/12/f4/3f/8982873f5bfad3134251f605011de0c35f93d64b78cb07e3b8
Successfully built common
Installing collected packages: common
Successfully installed common-0.1.2
{% endcode %}

Notice how we added the location an extra index using `--extra-index-url`. The URL points to the `devtools` directory in the bucket which is the root directory for the packages created by the "devtools" team. The subdirectories follow the layout mentioned previously.

Next we'll try the same with Poetry after uninstalling the package using pip. First, we'll check how Poetry allows us to do it. 

{% code lang:bash %}
poetry add --help
...
A url (https://example.com/packages/my-package-0.1.0.tar.gz)
{% endcode %}

Let's go ahead and add the package in that format.  

{% code lang:bash %}
poetry add --dry-run https://selfhostedpackages.s3.us-west-2.amazonaws.com/packages/devtools/common/common-0.1.0.tar.gz

Updating dependencies
Resolving dependencies... (0.1s)

Package operations: 3 installs, 0 updates, 0 removals, 29 skipped
...
{% endcode %}

We can verify that the package was installed successfully by importing it in a Python shell. 

{% code lang:python %}
Python 3.12.0 | packaged by Anaconda, Inc. | (main, Oct  2 2023, 17:29:18) [GCC 11.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from common import create_histogram
>>>
{% endcode %}

## Conclusion

That's it. That's how we can host our own Python packages using S3. Note that this is a setup to get started quickly. If you're looking for a more matured setup, please take a look at the devpi project.<sup>[2]</sup> [The code is available on Github.](https://github.com/thescalaguy/library)

## Footnotes and References  
[1] https://packaging.python.org/en/latest/guides/hosting-your-own-index/#manual-repository
[2] https://devpi.net/docs/devpi/devpi/latest/+doc/index.html