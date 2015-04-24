Requirements
============

Tested on Django 1.7 through 1.8, It requires ``django.contrib.auth`` to be in
the ``INSTALLED_APPS`` list.

Note that from 1.0, Django 1.7 or later will be required.

Installation
============

django-lazysignup can be installed with your favourite package management tool
from PyPI::

  pip install django-lazysignup

Once that's done, you need to add ``lazysignup`` to your ``INSTALLED_APPS``.
You will also need to add ``lazysignup``'s authentication backend to your
site's ``AUTHENTICATION_BACKENDS`` setting::

  AUTHENTICATION_BACKENDS = (
    'django.contrib.auth.backends.ModelBackend',
    'lazysignup.backends.LazySignupBackend',
  )

Finally, you need to add lazysignup to your URLConf, using something like
this::

  urlpatterns += (''
      (r'^convert/', include('lazysignup.urls')),
  )

