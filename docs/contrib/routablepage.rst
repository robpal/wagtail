.. _routable_page_mixin:

====================================
Embedding URL configuration in Pages
====================================

The ``RoutablePageMixin`` mixin provides a convenient way for a page to respond on multiple sub-URLs with different views. For example, a blog section on a site might provide several different types of index page at URLs like ``/blog/2013/06/``, ``/blog/authors/bob/``, ``/blog/tagged/python/``, all served by the same ``BlogIndex`` page.

A ``Page`` using ``RoutablePageMixin`` exists within the page tree like any other page, but URL paths underneath it are checked against a list of patterns, using Django's urlconf scheme. If none of the patterns match, control is passed to subpages as usual (or failing that, a 404 error is thrown).


The basics
==========

To use ``RoutablePageMixin``, you need to make your class inherit from both :class:`wagtail.contrib.wagtailroutablepage.models.RoutablePageMixin` and :class:`wagtail.wagtailcore.models.Page`, and configure the ``subpage_urls`` attribute with your URL configuration.

Here's an example of an ``EventPage`` with three views:

.. code-block:: python

    from django.conf.urls import url

    from wagtail.contrib.wagtailroutablepage.models import RoutablePageMixin
    from wagtail.wagtailcore.models import Page


    class EventPage(RoutablePageMixin, Page):
        subpage_urls = (
            url(r'^$', 'current_events', name='current_events'),
            url(r'^past/$', 'past_events', name='past_events'),
            url(r'^year/(\d+)/$', 'events_for_year', name='events_for_year'),
        )

        def current_events(self, request):
            """
            View function for the current events page
            """
            ...

        def past_events(self, request):
            """
            View function for the past events page
            """
            ...

        def events_for_year(self, request):
            """
            View function for the events for year page
            """
            ...


The ``RoutablePageMixin`` class
===============================

.. automodule:: wagtail.contrib.wagtailroutablepage.models
.. autoclass:: RoutablePageMixin

    .. autoattribute:: subpage_urls

        Example:

        .. code-block:: python

            from django.conf.urls import url

            from wagtail.wagtailcore.models import Page


            class MyPage(RoutablePageMixin, Page):
                subpage_urls = (
                    url(r'^$', 'main', name='main'),
                    url(r'^archive/$', 'archive', name='archive'),
                    url(r'^archive/(?P<year>[0-9]{4})/$', 'archive', name='archive'),
                )

                def main(self, request):
                    ...

                def archive(self, request, year=None):
                    ...

    .. automethod:: resolve_subpage

        Example:

        .. code-block:: python

            view, args, kwargs = page.resolve_subpage('/archive/')
            response = view(request, *args, **kwargs)

    .. automethod:: reverse_subpage

        Example:

        .. code-block:: python

            url = page.url + page.reverse_subpage('archive', kwargs={'year': '2014'})


 .. _routablepageurl_template_tag:

The ``routablepageurl`` template tag
====================================

.. currentmodule:: wagtail.contrib.wagtailroutablepage.templatetags.wagtailroutablepage_tags
.. autofunction:: routablepageurl

    Example:

    .. code-block:: html+django

        {% load wagtailroutablepage_tags %}

        {% routablepageurl self "feed" %}
        {% routablepageurl self "archive" 2014 08 14 %}
        {% routablepageurl self "food" foo="bar" baz="quux" %}
