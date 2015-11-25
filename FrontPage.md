Project Name:ISAPI WSGI

License: MIT

Description: An implementation of WSGI (PEP 333) for running as a ISAPI extension under IIS. WSGI is considered as important standard for the future of web deployed Python code. There are implementations for CGI, mod python, twisted, jython etc but to my knowledge not one using ISAPI for IIS. The goal of this project is to provide one. It is dependant on Mark Hammond's Python win32 isapi extension.

Current status: The isapi\_wsgi handler has been modified to overcome the limitations of the single threaded version of the handler. You can checkout the new version from the trunk (svn checkout http://isapi-wsgi.googlecode.com/svn/trunk/). The documentation needs to be updated for the changes but the examples in the trunk show how to use it.

[ISAPISimpleHandlerDocs](ISAPISimpleHandlerDocs.md)