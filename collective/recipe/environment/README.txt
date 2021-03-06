Overview
========

This recipe allows environment variables to be accessed during the execution of
a buildout by making them buildout variables.
It also allows you to change environment variables.

The recipe mirrors the current environment variables into its section, so that e.g.
``${environment:USER}`` will give the current user.

To set an environment variable you just set it in the section.

The environment variables are set and get during the initialization of the ``Recipe`` instance,
i.e. after ``buildout.cfg`` is read but before any recipe is installed or updated.

If an environment variable contains an illegal char such as ":" it will be
replaced by a "-".

If you want to read from the environment only you can use the recipe
``collective.recipe.environment:read-only``. This allows you to set default
values for buildout variables in case the environment variables don't exist.


Example usage: Use an environment variable
==========================================

We'll start by creating a buildout that uses the recipe::

    >>> write('buildout.cfg',
    ... """
    ... [buildout]
    ... parts = environment print
    ...
    ... [some-section]
    ... some-option = ${environment:SOME_VARIABLE}
    ...
    ... [environment]
    ... recipe = collective.recipe.environment
    ...
    ... [print]
    ... recipe = mr.scripty
    ... install =
    ...     ... print self.buildout['some-section']['some-option']
    ...     ... return []
    ... """)

The `mr.scripty`_ recipe is used to print out the value of the ${some-section:some-option}
option.

Now we set the environment variable::

    >>> import os
    >>> os.environ['SOME_VARIABLE'] = 'some_value'

Running the buildout gives us::

    >>> print 'start', system(buildout)
    start...
    some_value
    <BLANKLINE>


Example usage: Set an environment variable
==========================================

We'll start by creating a buildout that uses the recipe::

    >>> write('buildout.cfg',
    ... """
    ... [buildout]
    ... parts = environment print
    ...
    ... [some-section]
    ... some-option = value2
    ...
    ... [environment]
    ... recipe = collective.recipe.environment
    ... var1 = value1
    ... var2 = ${some-section:some-option}
    ...
    ... [print]
    ... recipe = mr.scripty
    ... install =
    ...     ... import os
    ...     ... for var in ('var1', 'var2'):
    ...     ...     print '%s = %s' % (var, os.environ[var])
    ...     ... return []
    ... """)

The `mr.scripty`_ recipe is used to print out the values of the environment variables.

Running the buildout gives us::

    >>> print 'start', system(buildout)
    start...
    var1 = value1
    var2 = value2
    <BLANKLINE>


Example usage: Default values
==========================================

We'll start by creating a buildout that uses the recipe::

    >>> write('buildout.cfg',
    ... """
    ... [buildout]
    ... parts = environment print
    ...
    ... [some-section]
    ... some-option = value2
    ...
    ... [environment]
    ... recipe = collective.recipe.environment:read-only
    ... var1 = value1
    ... var2 = default1
    ...
    ... [print]
    ... recipe = mr.scripty
    ... install =
    ...     ... import os
    ...     ... for var in ('var1', 'var2'):
    ...     ...     print '%s = %s' % (var, os.environ[var])
    ...     ... return []
    ... """)

The `mr.scripty`_ recipe is used to print out the values of the environment variables.

    >>> import os
    >>> os.environ['var1'] = 'changed'


Running the buildout gives us::

    >>> print 'start', system(buildout)
    start...
    var1 = changed
    var2 = default1
    <BLANKLINE>


Similar recipes
===============

The functionality to mirror the environment variables into the recipe's section is largely copied
from `gocept.recipe.env`_.

.. References
.. _`mr.scripty`: http://pypi.python.org/pypi/mr.scripty
.. _`gocept.recipe.env`: http://pypi.python.org/pypi/gocept.recipe.env
