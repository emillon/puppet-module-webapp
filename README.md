Puppet Webapp Module tying together Nginx, Gunicorn, and Monit
==============================================================

Helper module for easy configuration of Python WSGI applications
with Virtualenv, Gunicorn, Monit, and Nginx.

Tested on Debian GNU/Linux 6.0 Squeeze. Patches for other
operating systems welcome.


Installation
------------

Clone this repo and all its dependencies to respective directories under
your Puppet modules directory:

    git clone git://github.com/uggedal/puppet-module-webapp.git webapp
    git clone git://github.com/uggedal/puppet-module-python.git python
    git clone git://github.com/uggedal/puppet-module-monit.git monit
    git clone git://github.com/uggedal/puppet-module-nginx.git nginx

If you don't have a Puppet Master you can create a manifest file
based on the notes below and run Puppet in stand-alone mode
providing the module directory you cloned this repo to:

    puppet apply --modulepath=modules test_webapp.pp


Usage
-----

To install Python with development dependencies, Virtualenv, Gunicorn support
directories, Monit, and Nginx simply include the module:

    include webapp::python

You should provide an unprivileged user which will own the Virtualenv files
and Gunicorn processes by including the module with this special syntax:

    class { "webapp::python": owner => "www-mgr", group => "www-mgr" }

By default this module will look for source code under `/usr/local/src/$name`
and create virtualenvs under `/usr/local/venv`. To override this, provide
the following arguments on class instantiation:

    class { "webapp::python": owner => "www-mgr",
                              group => "www-mgr",
                              src_root => "/home/www-mgr/src",
                              venv_root => "/home/www-mgr/venv",
    }

Note that you'll need to define a global search path for the `exec`
resource to make the `python::webapp` resource function
properly. This should ideally be placed in `manifests/site.pp`:

    Exec {
      path => "/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
    }

The most basic setup of a Nginx virtualhost, virtualenv, Gunicorn installation
inside the virtualenv, and Monit watching the Gunicorn processes:
    
    python::webapp { "blog":
      domain => "blog.uggedal.com",
      wsgi_module => "blog:app",
    }

You can provide domain aliases which Nginx redirects to your main domain:

    python::webapp { "blog":
      domain => "blog.uggedal.com",
      aliases => ["journal.uggedal.com"],
      wsgi_module => "blog:app",
    }

If your application is busy you can increase the amount of Gunicorn workers:

    python::webapp { "blog":
      domain => "blog.uggedal.com",
      wsgi_module => "blog:app",
      workers => 4,
    }

Django applications does not use the `wsgi_module`, but enabled by using the
`django` flag:

    python::webapp { "cms":
      domain => "cms.uggedal.com",
      django => true,
    }

Puppet can manage installation of requirements from a `requirements.txt`
inside your source directory:

    python::webapp { "cms":
      domain => "cms.uggedal.com",
      django => true,
      requirements => true,
    }
