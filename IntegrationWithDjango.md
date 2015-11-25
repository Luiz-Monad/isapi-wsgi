Note: This has only been tested with Django 1.0.2 and isapi\_wsgi.py Release 0.4

## Django ##

The Django framework provides the django.core.handlers.wsgi.WSGIHandler() function for constructing a WSGI application corresonding to a Django application. Using this function, a script file for a Django application which will work with isapi\_wsgi would be constructed as follows:

```
import os, sys
sys.path.append('C:\\sw\\django')
sys.path.append('C:\\sw\\django\\mysite')
os.environ['DJANGO_SETTINGS_MODULE'] = 'mysite.settings'
import django.core.handlers.wsgi
application = django.core.handlers.wsgi.WSGIHandler()

import isapi_wsgi
# The entry points for the ISAPI extension.
def __ExtensionFactory__():
    return isapi_wsgi.ISAPISimpleHandler(application)

if __name__=='__main__':
    # If run from the command-line, install ourselves.
    from isapi.install import *
    params = ISAPIParameters()
    # Setup the virtual directories - this is a list of directories our
    # extension uses - in this case only 1.
    # Each extension has a "script map" - this is the mapping of ISAPI
    # extensions.
    sm = [
        ScriptMapParams(Extension="*", Flags=0)
    ]
    vd = VirtualDirParameters(Name="mysite",
                              Description = "ISAPI-WSGI ISAPISimpleHandler Django mysite",
                              ScriptMaps = sm,
                              ScriptMapUpdate = "replace"
                              )
    params.VirtualDirs = [vd]
    HandleCommandLine(params)
```

The directories added to sys.path will be both the directories containing the package and the Django site created by running:

```
django-admin.py startproject mysite
```

The script (in this example called django\_wsgi.py) is installed as an ISAPI application by running:

```
python django_wsgi.py install
```

## Pinax ##

Pinax is an integrated collection of Django applications that provides the most commonly needed social networking features. Since the collection of applications and their supporting python libraries are in a Pinax specific directory structure, a more complex script file is required. For example to run the Pinax basic project under isapi\_wsgi, the following script is required:

```
import os, sys
from os.path import abspath, dirname, join
from site import addsitedir

PINAX_ROOT = abspath(join(dirname(__file__), "../../"))
PROJECT_ROOT = abspath(join(dirname(__file__), "./"))

path = addsitedir(join(PINAX_ROOT, "libs/external_libs"), set())
if path:
    sys.path = list(path) + sys.path

sys.path.insert(0, join(PINAX_ROOT, "apps/external_apps"))
sys.path.insert(0, join(PINAX_ROOT, "apps/local_apps"))
sys.path.insert(0, join(PINAX_ROOT, "projects"))
sys.path.insert(0, join(PROJECT_ROOT, "apps"))
sys.path.insert(0, PROJECT_ROOT)
sys.path.insert(0, abspath(join(dirname(__file__), "../../")))

#debug
print "PINAX_ROOT:%s" % PINAX_ROOT
print "PROJECT_ROOT:%s" % PROJECT_ROOT
print "sys.path:%s" % sys.path


os.environ['DJANGO_SETTINGS_MODULE'] = 'basic_project.settings'
import django.core.handlers.wsgi
application = django.core.handlers.wsgi.WSGIHandler()
import isapi_wsgi
# The entry points for the ISAPI extension.
def __ExtensionFactory__():
    return isapi_wsgi.ISAPISimpleHandler(application)
if __name__=='__main__':
    # If run from the command-line, install ourselves.
    from isapi.install import *
    params = ISAPIParameters()
    # Setup the virtual directories - this is a list of directories our
    # extension uses - in this case only 1.
    # Each extension has a "script map" - this is the mapping of ISAPI
    # extensions.
    sm = [
        ScriptMapParams(Extension="*", Flags=0)
        ]
    vd = VirtualDirParameters(Name="pinax_basic",
                              Description = "ISAPI-WSGI ISAPISimpleHandler Django Pinax basic project site",
                              ScriptMaps = sm,
                              ScriptMapUpdate = "replace"
                              )
    params.VirtualDirs = [vd]
    HandleCommandLine(params)

```

## Common Problems ##

Something to watch out if you are using SQLite as the backend database for your Django application. In your Django settings.py, the DATABASE\_NAME value must be the full path to the SQLite database file. Also by default the Django app is running as the standard IIS user which has minimal privileges. This means that the Django app does not have permission to write to the SQLite database file. So your app will appear to work until a database write occurs and you will be presented with an internal server error. The solution is to configure the virtual directory anonymous access account to a user with greater permissions. This can be done in the virtual directory Properties->Directory Security->Anonymous access and authentication control panel or you can do it as a post-install action in the script:

```
# Set virtual directory authentication post-install
def set_vd_authentication(params, options, webdir=None):
    if webdir is not None:
        webdir.AuthAnonymous = True
        # where Domain and userid are replaced with valid entries for your network
        webdir.AnonymousUserName = "Domain\\userid"
        webdir.AnonymousPasswordSync = True
        webdir.SetInfo()
        
if __name__=='__main__':
    # If run from the command-line, install ourselves.
    from isapi.install import *
    params = ISAPIParameters()
    # Setup the virtual directories - this is a list of directories our
    # extension uses - in this case only 1.
    # Each extension has a "script map" - this is the mapping of ISAPI
    # extensions.
    sm = [
        ScriptMapParams(Extension="*", Flags=0)
        ]
    vd = VirtualDirParameters(Name="pinax_basic",
                              Description = "ISAPI-WSGI ISAPISimpleHandler Django Pinax basic project site",
                              ScriptMaps = sm,
                              ScriptMapUpdate = "replace",
                              PostInstall = set_vd_authentication
                              )
    params.VirtualDirs = [vd]
    HandleCommandLine(params)
```