As Graham Dumpleton, the author of [mod\_wsgi](http://www.modwsgi.org) [points out](http://code.google.com/p/modwsgi/wiki/PerformanceEstimates) "Providing benchmarks comparing one system to another is always dangerous, especially when they use different technologies. This is because the performance can be dependent on how the system is setup or tested and what the hardware configuration and operating system used is."

Drawing inspiration from Grahams benchmarking of mod\_wsgi, a normalised figure of 1000 requests/sec for serving up static files through IIS is used as a baseline reference, with the different means of supporting a Python applications through IIS expressed relative to that value.

The actual figures given here were derived from testing with Microsoft IIS 6.0 running on a Windows Server 2003 Standard Edition [R2](https://code.google.com/p/isapi-wsgi/source/detail?r=2) SP1 and the Microsoft Web Application Stress Tool [(WAST)](http://support.microsoft.com/kb/231282). IIS is configured as a standard installation. WAST was configured to run with 50 concurrent requests for the duration of 1 minute. The benchmarks were run multiple times, and the published figures have been averaged and rounded to give as described what is more an estimate of the relative performance to the other mechanisms for running python code under IIS, than a definitive indicator of performance for the specific mechanism.

In all cases, the particular WSGI application which was used for testing was as follows:

```

def app(environ, start_response):
    '''Simple app as per PEP 333'''
    status = '200 OK'
    output = '''<html><header>
            <title>Benchmark Page</title>
            </header><body>
            <p>Hello world</p>
            </body></html>'''
    start_response(status, [('Content-type', 'text/html')])
    return [output]

```

### Static File Testing ###

The static file test used as a baseline consisted of placing a file in a IIS virtual directory setup to only serve static files from the local filesystem. This contained the basic HTML tags and text for a 'Hello World' page. The throughput for this test was normalised to being 1000 requests/sec. Subsequent results have been adjusted in the same proportion to make it easier to compare the different values.

### Test Of wsgi\_cgi Adapter ###

For the CGI test, the WSGI adapter listed in the WSGI PEP was used. This was included in the same file as the actual WSGI application.

The result of the test gave an adjusted figure of 10 requests/sec. So using the static file benchmark as our baseline, static files could be served up 100 times quicker than running the CGI script to generate the same value.

Note that the CGI test involves spawning a separate python interpreter process to run the CGI script.


### Test Of ASP.NET ###

An ASP.NET file using ASP classic style and VB as it's code language was used to generate the HMTL Hello World.

```

<%@ Page Language="VB" %>
<html><header>
<title>Benchmark Page</title>
</header><body>
<p><%="Hello world" %></p>
</body></html>

```

The result of the test gave an adjusted figure of 670 requests/sec.

### Summary of Results ###

| **Mechanism** | **Requests/sec** |
|:--------------|:-----------------|
| wsgi\_cgi     | 10               |
| isapi\_wsgi (ISAPISimpleHandler)| 167              |
| isapi\_wsgi (ISAPIThreadPoolHandler) | 170              |
| ASP.NET (Classic ASP) | 637              |
| static        | 1000             |