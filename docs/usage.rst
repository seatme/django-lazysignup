Usage
=====

The package works by creating temporary user accounts based on a user's
session key whenever a flagged view is requested. You can specify which
views trigger this behaviour using the ``lazysignup.decorators.allow_lazy_user``
decorator.

When an anonymous user requests such a view, a temporary user account will be
created for them, and they will be logged in. The user account will have
an unusable password set, so that it can't be used to log in as a regular
user. The way to tell a regular use from a temporary user is to call
the ``is_lazy_user()`` function from ``lazysignup.templatetags.lazysignup_tags``.
If this returns ``True``, then the user is temporary. Note that
``user.is_anonymous()`` will return ``False``  and ``user.is_authenticated()``
will return ``True``. See below for more information on ``is_lazy_user``.

A view is provided to allow such users to convert their temporary account into
a real user account by providing a username and a password.

A Django management command is provided to clear out stale, unconverted user
accounts - although this depends on your use of database-backed sessions, and
assumes that all user accounts with an expired session are safe to delete. This
may not be the case for all apps, so you may wish to provide your own cleaning
script.

The ``allow_lazy_user`` decorator
---------------------------------

Use this decorator to indicate that accessing the view should cause anonymous
users to have temporary accounts created for them.

For example::

  from django.http import HttpResponse
  from lazysignup.decorators import allow_lazy_user

  @allow_lazy_user
  def my_view(request):
    return HttpResponse(request.user.username)

When accessing the above view, a very simple response containing the generated
username will be displayed.

``require_lazy_user`` and ``require_nonlazy_user`` decorators
-------------------------------------------------------------

It is also possible to mark views as requiring only a lazily-created user,
or requiring only a non-lazy user, with the ``require_lazy_user`` and
``require_nonlazy_user`` decorators respectively. These decorators take
arguments and keyword arguments which are passed verbatim to Django's own
``redirect`` view.


The ``is_lazy_user`` template filter
------------------------------------

This template filter (which can also be imported from ``lazysignup.utils``
and used in your own code) will return True if the user is a generated user.
You need to pass it the user to test. For example, a site navigation
template might look like this::

    {% load i18n lazysignup_tags %}

    <nav id="account-bar">
      <ul>
        <li><a href="{% url home %}">{% trans "Home" %}</a></li>
        {% if not user|is_lazy_user %}
          <li><a href="#">{% trans "Account" %}</a></li>
          <li><a href="{% url auth_logout %}">{% trans "Log out" %}</a></li>
        {% else %}
          <li><a href="{% url lazysignup_convert %}">{% trans "Save your data" %}</a> {% trans "by setting a username and password" %}</li>
        {% endif %}
      </ul>
    </nav>

This filter is very simple, and can be used directly in view code, or tests. For example::

    from lazysignup.utils import is_lazy_user

    def testIsLazyUserAnonymous(self):
        user = AnonymousUser()
        self.assertEqual(False, is_lazy_user(user))

Note that as of version 0.6.0, the user tested no longer needs to have been
authenticated by the ``LazySignupBackend`` for lazy user detection to work.


User agent blacklisting
-----------------------

The middleware will not created users for certain requests from blacklisted
user agents. This is simply a fairly crude method for preventing many spurious
users being created by passing search engines.

The blacklist is specified with the ``USER_AGENT_BLACKLIST`` setting. This
should be an iterable of regular expression strings. If the user agent string
of a request matches a regex (``search()`` is used, so the match can be anywhere
in the string) then a user will not be created.

If the list is not specified, then the default is as follows

  - slurp
  - googlebot
  - yandex
  - msnbot
  - baiduspider

Specifying your own ``USER_AGENT_BLACKLIST`` will replace this list.

Using the convert view
----------------------

Users will be able to visit the ``/convert/`` view. This provides a form with
a username, password and password confirmation. As long as they fill in valid
details, their temporary user account will be converted into a real user
account that they can log in with as usual.

You may specify your own form class into the `convert` view in order to customise
user creation. The code requires expects the following:

  - It expects to be able to create the form passing in the generated ``User``
    object with an ``instance`` kwarg (in general, this is fine when using a
    ModelForm based on the User model)
  - It expects to be able to call ``save()`` on the form to convert the user
    to a real user
  - It expects to be able to call a ``get_credentials()`` method on the form
    to obtain a set of credentials to authenticate the new user with. The
    result of this call should be a dictionary suitable for passing to
    ``django.contrib.auth.authenticate()``. Typically, this would be a dict
    with ``username`` and ``password`` keys - but this may vary if you're using
    a different authentication backend.

The default configuration, using the provided ``UserCreationForm``, should
be enough for most users, but the customisation point is there if you need
it.

To specify your own form, set the ``LAZYSIGNUP_CUSTOM_USER_CREATION_FORM``
setting to your settings file like so::

  LAZYSIGNUP_CUSTOM_USER_CREATION_FORM = 'myproject.apps.myapp.forms.MyForm'

The view also supports ``template_name`` and ``ajax_template_name`` arguments,
to specify templates to render in web and ajax contexts respectively.

The ``converted`` signal
------------------------

Whenever a temporary user account is converted into a real user account, the
``lazysignup.signals.converted`` signal will be sent.  If you need to do any
processing when an account is converted, you should listen for the signal, eg::

    from lazysignup.signals import converted
    from django.dispatch import receiver

    @receiver(converted)
    def my_callback(sender, **kwargs):
        print "New user account: %s!" % kwargs['user'].username

The signal provides a single argument, ``user``, which contains the
newly-converted User object.
