# Introduction #

ISAPI-WSGI is a Python module which allows you to deploy a Python WSGI application natively in Microsoft IIS. This means that, for example, you can use IIS thread pooling and load balancing to manage the resources of your application: there is no need to start a separate process or service.

# Installation and pre-requisites #

To use ISAPI-WSGI, you first need to install:

  * Python version 2.3 or later
  * The [Python Win32 extensions](http://python.net/crew/skippy/win32/Downloads.html)
  * If you are in Python 2.4 or earlier, the [wsgiref library](http://pypi.python.org/pypi/wsgiref)
  * The [isapi\_wsgi module](http://pypi.python.org/pypi/isapi_wsgi) itself

`wsgiref` and `isapi_wsgi` are available on PyPI and can be installed using `easy_install` or `zc.buildout`. Thus, if you want to install these into your global Python environment and you have easy\_install available, you can get the latest version with:

```
  > easy_install -U wsgiref
  > easy_install -U isapi_wsgi
```

# Theory #

Under the hood, ISAPI-WSGI uses the `isapi` module from the Python Win32 extensions. The normal usage pattern is like this:

  1. Create a script which loads (or contains) your WSGI application and initialies it
  1. In this script, place a method `__ExtensionFactory__()` which returns an ISAPI-WSGI object (either the 'simple' or 'threadpool' handler)
  1. Also in this script, add some initialisation boilerplate which is run when the script is executed through the interpreter.
  1. The initialisation boilerplate creates a DLL file and installs it as a "wildcard" handler in IIS, either for an entire site, or for a virual directory in that site.
  1. When a request is received by IIS, it will pass it onto the wildcard handler, which will translate the request into a WSGI request for your application.

Note that the automatic IIS configuration in step 4 is convenience only. It is possible to configure IIS manually by installing a wildcard handler using the generated DLL file. If you prefer to do this, you can ignore the `VirtualDirs` list being created below. Do remember to turn off "Check for existence of file" in the IIS configuration.

# Example #

This is a simple example that uses [Paste Deploy](http://pythonpaste.org/deploy/). Paste Deploy loads a WSGI application (configured in a Paste Deploy INI file not shown here), runs in a thread-pool, and installs to a virtual directory called 'wsgi-test':

```
# Enable tracing. Run 'python -m win32traceutil'
if hasattr(sys, "isapidllhandle"): 
    import win32traceutil

# The entry point for the ISAPI extension.
def __ExtensionFactory__():
    from paste.deploy import loadapp
    from paste.script.util.logging_config import fileConfig

    configfile="C:\\MyApp\\MyApp.ini"
    fileConfig(configfile)
    application = loadapp("config:" + configfile)

    import isapi_wsgi
    return isapi_wsgi.ISAPIThreadPoolHandler(application)

# ISAPI installation
if __name__=='__main__':
    from isapi.install import ISAPIParameters, ScriptMapParams, VirtualDirParameters, HandleCommandLine
    
    params = ISAPIParameters()
    sm = [
        ScriptMapParams(Extension="*", Flags=0)
    ]
    
    vd = VirtualDirParameters(Name="wsgi-test",
                              Description = "A test application",
                              ScriptMaps = sm,
                              ScriptMapUpdate = "replace"
                              )
    
    params.VirtualDirs = [vd]
    HandleCommandLine(params)

```

At the top of the file, we enable trace logging. This allows you to watch for log messages by running:

```
python -m win32traceutil
```

This is very useful for debugging, though you probably want to remove the import if `win32traceutil` for production use.

Next, we define the `__ExtensionFactory__()` function, which is the entry point for ISAPI. IIS will call this function and expect to get a handler object back. Here, we use the `ISAPIThreadPoolHandler`, which enables thread pooling. There is also `ISAPISimpleHandler`, which runs single-theraded. The handler is passed the WSGI application (a callable).

The handler may optionally take other applications as keyword arguments, which will be available on various sub-domains:

```
    return isapi_wsgi.ISAPIThreadPoolHandler(default_app, foo=foo_app, bar=bar_app)
```

Here, `/foo` (under the virtual directory where the handler is installed - more on that in moment) will execute the WSGI application represented by the `foo_app` callable, `/bar` will execute `bar_app` and everything else will go to `default_app`.

With the entry point configured, the rest of the script is concerned with creating a DLL file and configuring IIS. Let's look at the code again:

```
if __name__=='__main__':
    from isapi.install import ISAPIParameters, ScriptMapParams, VirtualDirParameters, HandleCommandLine
    
    params = ISAPIParameters()
    sm = [
        ScriptMapParams(Extension="*", Flags=0)
    ]
    
    vd = VirtualDirParameters(Name="wsgi-test",
                              Description = "A test application",
                              ScriptMaps = sm,
                              ScriptMapUpdate = "replace"
                              )
    
    params.VirtualDirs = [vd]
    HandleCommandLine(params)
```

The `HandleCommandLine` function parses command line options and is capable of configuring IIS when the script is run like so (if the script is called `myapp.py`):

```
python myapp.py install
```

This will configure the default IIS site. If you want to configure another site, you can use the --server argument:

```
python myapp.py install --server=MySite
```

`MySite` is the name of the site object in the IIS console.

When this script is run, a DLL file will be created in the current directory, called e.g. `_myapp.dll`. This is then intalled into IIS as wildcard handler.

In the example above, the configuration takes place in a virtual directory called "wsgi-test" that is created if it does not exist. To configure the root of the IIS site, pass `Name="/"`:

```
    vd = VirtualDirParameters(Name="wsgi-test",
                              Description = "A test application",
                              ScriptMaps = sm,
                              ScriptMapUpdate = "replace"
                              )
```

# IIS configuration #

Presuming you are using the thread pool handler, you can configure the application pool to use from the properties of the site or virtual directory in IIS, under the "Home directory" tab. This application pool can be configured to monitor memory usage, limit resource consumption, automatically restart processes and so on. If you are using IIS 7.x, Isapi\_wsgi requires that IIS 6.0 Management Compatibility is installed.

## Security ##

IIS may need write access to be able to write log files or other temporary files. The exact requirements will be application-dependent, but bear in mind that when your application runs in an IIS application pool, the user to run as can be set in the application pool configuration. The win32traceutil log should be able to tell you of any "permission denied" errors stopping your application from executing correctly.

# Managing your environment #

Most non-trivial applications will depend on third party libraries, and most people will not want to install those packages in their global Python environment. It is probably a good idea to install the Python Win32 extensions globally, but everything else (including `wsgiref` and `isapi_wsgi`) can be placed in a local environment.

The two most common approaches for managing a local Pythone environment are [virtualenv](http://pypi.python.org/pypi/virtualenv) and [zc.buildout](http://www.buildout.org/)

## Using virtualenv ##

Presuming your global Python environment is relatively clean, you should be able to load the packages in a specific virtualenv by placing the following at the top if your ISAPI-WSGI script:

```
import site
site.addsitedir('/path/to/virtualenv/lib/python2.5/site-packages')
```

This assumes the virtualenv uses Python 2.5 and is installed in /path/to/virtualenv.

## Using zc.buildout ##

If you are using zc.buildout, you probably want to make sure that WSGI application contains the packages in the working set managed for your application by the buildout process. As long as you are using Paste Deploy to configure your WSGI application, there is a buildout recipe which can generate the ISAPI-WSGI script and even automatically install it for you. It can be used like this:

```
[buildout]
parts = wsgi
develop = src/myapp

[wsgi]
recipe = collective.recipe.isapiwsgi
eggs =
    wsgiref
    isapi_wsgi
    myapp
config-file = ${buildout:directory}/production-deployment.ini
directory = /
server = Default
debug = True
```

This example configures the root of the IIS site called 'Default' with a script similar to the one shown above.

See the [collective.recipe.isapiwsgi](http://pypi.python.org/pypi/collective.recipe.isapiwsgi) documentation for more details.