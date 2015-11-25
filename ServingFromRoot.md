## Serving a WSGI app from IIS root ##

The mount point for an isapi wsgi app is normally the virtual directory name. But since isap\_wsgi 0.4 it is possible to serve the application from IIS root. It is as simple as setting virtual directory parameter name to /

```
    vd = VirtualDirParameters(Name="/",
                              Description = "ISAPI-WSGI serve from root demo",
                              ScriptMaps = sm,
                              ScriptMapUpdate = "replace"
                              )
```

An example app can be found here:

http://code.google.com/p/isapi-wsgi/source/browse/trunk/examples/demo_serve_from_root.py