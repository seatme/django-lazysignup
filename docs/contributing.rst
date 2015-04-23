Helping Out
===========

If you want to add a feature or fix a bug, please go ahead! Fork the project
on `GitHub`_ and when you're done with your changes, let me know. Fixes and
features with tests have a greater chance of being merged. To run the tests,
do::

  git clone https://github.com/danfairs/django-lazysignup
  cd django-lazysignup

  # Install dependencies and requirements
  pip install -e .[all]

  # Setup Database
  psql -C "CREATE USER lazysignup with login createdb password 'lazysignup';"
  psql -C "CREATE DATABASE lazysignup with OWNER lazysignup;"

  # Run the tests and report coverage
  coverage run manage.py test
  coverage report --fail-under=100

  coverage run manage.py test --settings=custom_user_tests.settings
  coverage report --fail-under=98

.. _GitHub: https://github.com/danfairs/django-lazysignup
