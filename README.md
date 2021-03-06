# Idea-Box

[![Build Status](https://travis-ci.org/cfpb/idea-box.svg?branch=master)](https://travis-ci.org/cfpb/idea-box)

Idea-Box is a django app for collecting and surfacing ideas from users, in the vein of
IdeaScale, IdeaX, and Django Voice. Idea-Box differs from these projects in its minimal,
easily integrate-able interface. Idea-Box also takes a strong stance on transparency,
such that ideas, votes, etc. are tied to specific users.

## Features
* Searching 
* Idea Submission
* Tagging (via taggit)
* Voting
* Comments
* Listing by recent, comment count, vote count
* Separate state for archived ideas
* Customizable banner for specific campaigns

## Screen shot

![Idea Profile](https://raw.github.com/cfpb/idea-box/master/doc/images/profile.png)

## Requirements
* django (1.5.4) - This is a django app, so you need django.
* django-haystack (2.0.0) - A mapper between django models and search
backends.
* pyelasticsearch - Library for communicating with elasticsearch.
* django-taggit - A library for Tags within django
* mock - A library for creating mock objects for testing. 
* south - A library for schema and data migrations. 

* elasticsearch - A Search backend. Unfortunately, we currently require
elasticsearch (rather than another backend) because we need specific functionality
that haystack doesn't give us direct access to. Eventually, we'll get a
pull request to haystack which will reduce our elasticsearch requirement.

## Installation

### Settings File
Modify your settings file to add the following apps:
* django.contrib.comments
* haystack
* idea

You will also need to configure haystack. See the haystack
[documentation](http://django-haystack.readthedocs.org/en/v1.2.7/tutorial.html#configuration)

If you'd prefer to take the quick route, add the following to your
settings.py:
```python
HAYSTACK_CONNECTIONS = {
    'default': {
        'ENGINE': 'haystack.backends.elasticsearch_backend.ElasticsearchSearchEngine',
        'URL': 'http://127.0.0.1:9200/',
        'INDEX_NAME': 'haystack',
    },
}
```

If you are going that route, make sure that you have a search_sites.py
module in the root of your project with something like the following:

```python
import haystack
haystack.autodiscover()
```

### Folder Structure

You will need to get the contents of the ```src/idea``` directory into
your django project. The simplest way to do this might be a symbolic
link.

```
mydjango_project/
  |- idea/      (includes models.py, views.py, etc.)
  |- mydjango_project/ (settings.py, url.py, etc.)
  |- manage.py
  |- etc etc etc
```

### URLs

Add the idea.urls, haystack.urls, and comments.urls to you url.py. For 
example:

```python
if 'idea' in settings.INSTALLED_APPS and \
        'django.contrib.comments' in settings.INSTALLED_APPS and\
        'haystack' in settings.INSTALLED_APPS:
    urlpatterns.append(url(r'^haystack/', include('haystack.urls')))
    urlpatterns.append(url(r'^comments/',
        include('django.contrib.comments.urls')))
    urlpatterns.append(url(r'^idea/', include('idea.urls')))
```

### Elasticsearch

You will need to have elasticsearch installed and running. You can use
[this guide](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/setup.html#setup-installation) to install it.


### Migrations

From your project root, synchronize and migrate the new apps.

```bash
$ python ./manage.py syncdb --noinput --migrate
```

### Templates

A basic set of templates has been provided. The 'base.html' template should
probably be overwritten into something less basic for a better user experience. 

If users in the system have a profile module specified our templates will use
this to link users to a profile page.  This is specified through the
AUTH_PROFILE_MODULE setting. Your profile module will also have to specify a
get_absolute_url() method.

### CSS

The default CSS style for Idea-Box is not ready for production.  This was done
intentionally so that Idea-Box's style can match the style of the platform in
which it resides (i.e. not everyone wants a green header).  The simplest way to
improve the styling is to source `src/idea/static/idea/css/sample_style.css`
in the `css_files` block in the `/src/idea/templates/idea/idea-base.html` template:

```
{% block "css_files" %}
    # ... existing code ...
    <link rel="stylesheet" href="{{ STATIC_URL }}idea/css/sample_style.css">
{% endblock %}
```

Ideally, your Django platform will provide styles that can be sourced in the
`base.html` template described in the section above so Idea-Box can match the
look and feel of your system.


### Buildout
To use buildout, run the following:
```bash
$ pip install zc.buildout distribute
$ buildout
```
Then, run the django binary in the ```bin``` directory.

### Campaign Banner

To create a campaign banner, use django's administrative page to add a Banner model. The
text field will be displayed at the top of the Idea-Box idea listing page. The banner
will only be displayed between Start Date and End Date (or indefinitely after the Start
Date if the End Date is empty.)
