# BPA Workshops

Workshops run by BPA.

## To see information on the initial Mkdocs setup see: https://bpa-csiro-workshops.github.io/btp-module-template-md/modules/setup_editing/mkdocs_setup/

## Deployment

The tutorials have been deployed here: https://bpa-csiro-workshops.github.io/btp-manuals-md  
The template has been depolyed here: https://bpa-csiro-workshops.github.io/btp-module-template-md/

## Github Repository

The tutorials Github repo is here: https://github.com/BPA-CSIRO-Workshops/btp-manuals-md  
The template GitHub repo is here: https://github.com/BPA-CSIRO-Workshops/btp-module-template-md

## How to work on them locally

We are using the Python package [MkDocs](http://www.mkdocs.org/) and the
[Material for MkDocs](https://squidfunk.github.io/mkdocs-material/) theme.
On OS X, if you haven't got commandline tools installed you can follow this
[guide](http://softwaretester.info/install-and-upgrade-pip-on-mac-os-x/).

You can install MkDocs and extensions into a virtualenv:

```
virtualenv mkdocs_dev
source mkdocs_dev/bin/activate
pip install package_name
```

Alternatively, if you have not installed in a virtualenv and do not have sudo
permissions, you can install as a user only, using --user

```
pip install --user package_name
```

### Install the mkdocs tools needed

```
pip install mkdocs markdown-include mkdocs-alabaster mkdocs-bootstrap python-markdown-math mkdocs-material pymdown-extensions pygments
```

To update a package

```
pip install -U package_name
```

### Fork and cloning the repo

We will be using a fork and pull collaborative model: https://github.com/BPA-CSIRO-Workshops/btp-workshop-template#fork-and-pull-collaborative-model

First fork https://github.com/BPA-CSIRO-Workshops/btp-manuals-md to your own GitHub account.  

You can then clone the repo from your GitHub account

```
git clone https://github.com/*Your_GitHub*/btp-manuals-md
cd btp-manuals-md
```

[GitHub Desktop](https://desktop.github.com/) is an alternative for command line git.

### The master document is a YAML file

This file contains the instructions of what to display. It has already been configured for BPA so shouldn't require
any editing unless a new module is being added or new styling is required.

```
less mkdocs.yml
```

### The actual Markdown pages are in the `docs` folder:

#### If making a new module from scratch

```
cd docs;
mkdir module_name;
mkdir module_name/images
```

To add a new page, say a page on the 'Canu' assembler, find the right location and create a page.
In this case it would be `docs/modules/module_name/canu.md`. Write the tutorial in that file, and then add the file to the
master document `mkdocs.yml` in the correct section under pages:

#### If editing an existing module

Existing modules are located in https://github.com/BPA-CSIRO-Workshops/btp-manuals-md/tree/master/docs/modules.
Navigate to the appropriate module and directly edit the .md file. Images relating to
the .md file should be located in module_name/images

You do not have to add the module to `mkdocs.yml` as it would have previously been added.


### Editing markdown files

You can edit the .md file with any text editor. [Atom](https://atom.io/) works really
well as an editor and it highlights files that have been updated.

### BTP template

You should apply the BTP template in your .md file.

The built version can be seen here: https://bpa-csiro-workshops.github.io/btp-module-template-md/modules/example/example/  
The corresponding raw Markdown and underlying syntax for this can be viewed here:
https://raw.githubusercontent.com/BPA-CSIRO-Workshops/btp-module-template-md/master/docs/modules/example/example.md  

Please feel free free to add to, or edit, the template here:
https://github.com/BPA-CSIRO-Workshops/btp-module-template-md

### Browse the site locally without deploying to public internet.

**In same directory as mkdocs.yml file**

```
mkdocs serve
```
or
```
python -m mkdocs serve
```

Open your web browser to http://127.0.0.1:8000/ and leave it open.
This will update automatically as you make changes to the documentation and allow you to view changes you have made.


### When you are happy, add it the repo

```
git add docs/modules/module_name/canu.md
git commit -m "Added canu" docs/modules/module_name/canu.md
git push origin master
```
Your private local web version  http://127.0.0.1:8000/ will also update.

### Pushing changes and pull requests

Once you have added it to the repo, you can push changes to the repo in your own GitHub account and
can then put in a pull request and BTP admin will review your changes.

### (**BTP Admin Step**): To deploy the whole lot to the *public* website

***Important: Building the public website will be done by BTP Admin.  
Your changes will not be publicly seen until Admin has deployed the changes.***

```
mkdocs gh-deploy --clean --message "Added canu"
```
This first builds a web HTML version of our Markdown hierarchy into the `site/` folder, then pushes it to a special
branch of the github repo called `gh-pages` which GitHub makes available at the public URL
https://bpa-csiro-workshops.github.io/btp-manuals-md
