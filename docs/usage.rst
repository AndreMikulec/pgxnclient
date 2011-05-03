Program usage
=============

The program entry point is the script called :program:`pgxn`. The script
offers several commands, whose list can be obtained using ``pgxn --help``. The
options available for each subcommand can be obtained using :samp:`pgxn
{command} --help`.

The main commands you may be interested in are `install`_ (to download, build
and install a distribution extensions into the system) and `load`_ (to load an
installed extension into a database). Commands to perform reverse operations
are `uninstall`_ and `drop`_. Use `download`_ to get a package from a mirror
without installing it.

There are also informative commands: `search`_ is used to search in the
repository, `info`_ and `list`_ to get informations about a distribution.
`mirror`_ can be used to get a list of mirrors.

The program can accept a few basic options, to be listed before the command:

:samp:`--mirror {URL}`
    Select a mirror to interact with. If not specified default to
    http://api.pgxn.org/.

``--yes``
    Assume affirmative answer to all questions. Useful for unattended scripts.

``--verbose``
    Print more informations during the process.


Package specification
---------------------

Many commands such as install_ require a *package specification* to operate.
In its simple form the specification is just the name of a distribution:
``pgxn install foo`` means "install the most recent stable release of the
package ``foo``".

The specification allows specifying an operator and a version number, so that
``pgxn install 'foo<2.0'`` will install the most recent stable release of the
package before the release 2.0. The version numbers are ordered according to
the `Semantic Versioning specification <http://semver.org/>`__. Supported
operators are ``=``, ``==`` (alias for ``=``), ``<``, ``<=``, ``>``, ``>=``.
Note that you probably need to quote the string as in the example to avoid
invoking some shell redirection commands.

Whenever a command takes a specification in input, it also accepts options
``--stable``, ``--testing`` and ``--unstable`` to specify the minimum release
level accepted. The default is "stable".

A few commands also allow specifying a local ``.zip`` package or a local
directory containing a distribution: in this case the specification should
contain at least a path separator to disambiguate it from a distribution name,
for instance ``pgxn install ./foo.zip``.


.. _install:

``pgxn install``
----------------

Download, build and install a distribution on the local system.

The program takes a `package specification`_ identifying the distribution to
work with.  The download phase is skipped if the distribution specification
refers to a local directory or package.

Note that the built extension is not loaded in any database: use the command
`load`_ for this purpose.

The command will run the ``./configure`` script if available in the package,
then will perform ``make all`` and ``make install``. It is assumed that the
``Makefile`` provided by the distribution uses PGXS_ to build the extension,
but this is not enforced: you may provide any Makefile as long as the expected
commands are implemented.

.. _PGXS: http://www.postgresql.org/docs/9.1/static/extend-pgxs.html

The install phase usually requires root privileges in order to install a build
library and other files in the PostgreSQL directories: by default
:program:`sudo` will be invoked for the purpose. An alternative program can be
specified with the option :samp:`--sudo {PROG}`; ``--nosudo`` can be used to
avoid running any program.

If there are many PostgreSQL installations on the system, the extension will
be built and installed against the instance whose :program:`pg_config` is
first found on the :envvar:`PATH`. A different instance can be specified using
the option :samp:`--pg_config {PATH}`.


.. _load:

``pgxn load``
-------------

Load the extensions included in a distribution into a database. The
distribution must be already installed in the system, e.g. via the `install`_
command.

The distribution is specified according to the `package specification`_ and
can refer to a local directory or file. No consistency check is performed
between the packages specified in the ``install`` and ``load`` command: the
specifications should refer to compatible packages. The specified distribution
is only used to read the metadata: only installed files are actually used to
issue database commands.

The database to install into can be specified using options
``-d``/``--dbname``, ``-h``/``--host``, ``-p``/``--port``,
``-U``/``--username``. The default values for these parameters are the regular
system ones and can be also set using environment variables
:envvar:`PGDATABASE`, :envvar:`PGHOST`, :envvar:`PGPORT`, :envvar:`PGUSER`.

If the specified database version is at least PostgreSQL 9.1, and if the
extension specifies a ``.control`` file, it will be loaded using the `CREATE
EXTENSION`_ command, otherwise it will be loaded as a loose set of objects.
For more informations see the `extensions documentation`__.

.. _CREATE EXTENSION: http://www.postgresql.org/docs/9.1/static/sql-createextension.html
.. __: http://www.postgresql.org/docs/9.1/static/extend-extensions.html

.. todo:: Specify a schema
.. todo:: Is pg_config really required here?
