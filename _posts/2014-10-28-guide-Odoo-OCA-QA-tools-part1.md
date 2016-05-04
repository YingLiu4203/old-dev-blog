---
layout: post
title: Guide to Odoo Community Association Quality Tools Part 1
---

## Choices of Odoo 8.0 Testing Services 
Odoo 8.0 has a new test framework/platform called 
[runbot](http://runbot.odoo.com/runbot). It looks good but is evolving and 
lack of documents. On the other hand, 
[Odoo Community Association (OCA)](http://odoo-community.org/) 
has [a Maintainer Quality Tools project](https://github.com/OCA/maintainer-quality-tools) that 
helps to ensure the quality of Odoo addons. It can run module unit tests 
and report test coverage. It uses [Travis CI](https://travis-ci.com/) 
continuous integration service and [Coveralls](https://coveralls.io/) 
code coverage reporting service. Both are free services for open source
projects. Both services support GitHub integration and automatically run 
test and report results to GitHub projects. This article describe the 
the setup of these two service for an Odoo addon project. 

## Travis CI Configuration 

The sample Travis CI configuration files in the 
[sample folder of the OCA QA tools project]
(https://github.com/OCA/maintainer-quality-tools) needs some changes
to include new Python packages and Odoo addon version requirement. 
We cloned the the whole project to [a new repository]
(https://github.com/amdeb/maintainer-quality-tools) for two reasons: 

1. to decouple our project from teh OCA maintainer quality tools
2. to customize the testing configuration such as installation of a new 
Python package. 

### QA tools customization
In our addon, we need to use an Amazon Marketplace Web Service 
Python package called [boto](https://github.com/boto/boto). In 
the [cloned maintainer quality tools repository]
(https://github.com/amdeb/maintainer-quality-tools), we add  
a new line with text 'boto' to the end of the 'requirements.txt' 
file in 'travis' folder in the QA tools repository. 
The 'travis_install_nightly' script file in the same folder
installs all packages specified in the 'requirements.txt' file 
before running tests.

### travis.yml
There is a sample `travis.yml` in the 'sample_files' directory in the QA
project repository. we need to change the Odoo version and 
only run test in the official Odoo repository, i.e., no need 
to run in the OCA repository. We changed the `.travis.yml` to 
have a new '8.0' version, running only in the official Odoo repository 
and using customized GitHut QA repository. 
The content is as the following: 

```
language: python

python:
  - "2.7"

env:
  - VERSION="8.0" ODOO_REPO="odoo/odoo"

virtualenv:
  system_site_packages: true

install:
  - git clone https://github.com/amdeb/maintainer-quality-tools.git ${HOME}/maintainer-quality-tools
  - export PATH=${HOME}/maintainer-quality-tools/travis:${PATH}
  - travis_install_nightly

script:
  - travis_run_tests

after_success:
  coveralls
```

Copy the `.travis.yml`  to the top level of 
an Odoo addon GitHub repository that needs to run tests. 


### Travis CI GitHub Webhook
To automatically run tests when there is any new commits, we need
to link Travis CI service with out GitHub repository. 

After signing in with a GitHub account in [Travis CI]
(https://travis-ci.org/), one can enable a GitHub project in 
[Travis CI profile page](https://travis-ci.org/profile).   

A new push to the GitHub repository will trigger a build and 
test process in Travis CI. 

## Coveralls Configuration

There are two steps to configure Coverall. 

First, create a `.coveragerc` file in the top level of a 
GitHub repository. A sample configuration file is provided in
the 'cfg' directory in the above QA tools repository. Change 
the include line as the following:

```
include =
    */github-account/repository-name/*
```

Second, create an account in [Coveralls website](https://coveralls.io/)
using a GitHub account and authorize Coveralls to access 
the account's repositories. Then click Repository button to 
turn on the Coveralls report for the repository.

## Add Travis CI and Coveralls Status icon
The above configuration enables automatic runs of Travis CI and Coveralls
for a new GitHub commit. Adding the status icon links in the 
'README.md' enables displaying test results in GitHub repository. 
 
In the Repository page in Travis CI website, there is a build status icon 
on the top right. Clicking the icon will popup a dialog with an image
URL string. Copy the string and add it as an image link at the top of 
'README.md'. Below is an example: 

`[![Build Status](https://travis-ci.org/amdeb/amdeb-amazon.svg?branch=master)](https://travis-ci.org/amdeb/amdeb-amazon)`

Similarly, in the repository page of Coveralls site, there is a
'BADGE URLS' icon on the top right of the page. Clicking it will give
the URL as an SVG Markdown. Here is an example:

`[![Coverage Status](https://img.shields.io/coveralls/amdeb/amdeb-amazon.svg)](https://coveralls.io/r/amdeb/amdeb-amazon)`

Adding the code next to the Travis CI line. After pushing a new commit 
to GitHub, both icons should be display at the top of the 'README.md' file. 