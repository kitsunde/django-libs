Middlewares
===========

AjaxRedirectMiddleware
----------------------

When calling a view from an AJAX call and when that view returns a redirect,
jQuery changes the status code to 200. This means, in your success callback
you will not be able to determine, if the view returned to 200 or a redirect.

Interestingly, there is a workaround: If we return some made up status code,
jQuery will not change it.

This middleware makes sure that, if there was a redirect and if it was an
AJAX call, the return code will be set to ``278``.

In order to use this middleware, add it to your ``MIDDLEWARE_CLASSES``
setting::

    MIDDLEWARE_CLASSES = [
        'django.middleware.common.CommonMiddleware',
        'django.contrib.sessions.middleware.SessionMiddleware',
        ...
        'django_libs.middleware.AjaxRedirectMiddleware',
    ]


In your jQuery script you can now react to redirects::

    $.post(url, data, function(data, textStatus, jqXHR) {
        if (jqXHR.status == 278) {
            window.location.href = jqXHR.getResponseHeader("Location");
        } else {
            $("#" + container).replaceWith(data);
        }
    });

When you are using this middleware, it means that Redirects will no longer be
executed on the server and your AJAX function has to call the redirect URL
manually. If you really want to get the HTML that the last view in the redirect
chain would return, you can disable this middleware for some requests by
adding `ajax_redirect_passthrough` parameter to your data payload. When this
parameter is given, the middleware will be skipped::

    <form method="post" action=".">
        <input type="hidden" name="ajax_redirect_passthrough" value="1" />
        ...
    </form>


CustomBrokenLinkEmailsMiddleware
--------------------------------

Use this instead of the default `BrokenLinkEmailsMiddleware` in order to see
the current user in the email body. Use this with Django 1.6. Earlier versions
should use the `CustomCommonMiddleware` instead.


CustomCommonMiddleware
----------------------

Use this instead of the default `CommonMiddleware` in order to see the current
user in the email body. Use this with Django 1.5. Later versions should use the
`CustomBrokenLinkEmailsMiddleware` instead.


CustomSentry404CatchMiddleware
------------------------------

Make sure to add `user-agents` to your ``requirements.txt``.

Use this instead of the middleware provided by raven. It allows you to define
a list of regex strings in the setting ``RAVEN_IGNORABLE_USER_AGENTS``. This
way you can stop reporting for nasty spiders crawling your site and testing
all kinds of weird non-existant URLs.

This middleware also uses user-agents to parse the user agent string and
determine the device. If the device family equals ``Spider``, the request
will be ignored. If you don't want to block all spiders like this, set
``RAVEN_IGNORE_SPIDERS = False``.


ErrorMiddleware
---------------

Add this middleware if you would like to see the user's email address in the
traceback that is sent to you when a 500 error happens.

In order to use this middleware, add it to your ``MIDDLEWARE_CLASSES``
setting::

    MIDDLEWARE_CLASSES = [
        'django.middleware.common.CommonMiddleware',
        'django.contrib.sessions.middleware.SessionMiddleware',
        ...
        'django_libs.middleware.ErrorMiddleware',
    ]


SSLMiddleware
-------------

Add this middleware as the first middleware in your stack to forward all
requests to `http://yoursite.com` to `https://yoursite.com`. Define exceptions
via the setting `NO_SSL_URLS` - these requests will be served without HTTPS.
