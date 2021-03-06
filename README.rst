ESP SDK Python
==============

A Python interface for calling the Evident.io_ API.

.. _Evident.io: https://evident.io

This is still being built and subject to changes.

Installation
------------

Install the latest stable using pip::

    pip install esp

If you prefer to install from the latest git HEAD, you can use the setup.py script::

    git clone https://github.com/EvidentSecurity/esp-sdk-python
    cd esp-sdk-python
    python setup.py install

Configuration
-------------

The recommended way to set your keys is with environment variables. Export one
for your access key and another one for your secret access key::

    export ENV['ESP_ACCESS_KEY_ID']='access key from ESP'
    export ENV['ESP_SECRET_ACCESS_KEY']='secret access key from ESP'

You can also set them in the ESP module directly::

    from esp.settings import settings
    
    settings.access_key_id = 'access key from ESP'
    settings.secret_access_key = 'secret access key from ESP'

If you need to use an http proxy to hit the ESP API, you can set that using the
settings as well::

    settings.http_proxy = 'your-proxy-uri:8080'

For appliance users, you can change the host the SDK points at in the settings::

    settings.host = 'esp.my-host'

Usage
-----

All resources provided are class objects. You they export common methods to help
you interact with the ESP API. Below are a few examples that describe how the
SDK can be used.

Import the SDK using the "esp" namespace::

    In [1]: import esp

Fetching reports is simple::

    In [2]: reports = esp.Report.find()

    In [3]: reports
    Out[3]: <esp.resource.PaginatedCollection at 0x10b291dd8>

This returns a paginated collection object that will let you navigate the pages returned::

    In [4]: reports.current_page_number
    Out[4]: '1'

    In [5]: reports = reports.next_page()

    In [6]: reports.current_page_number
    Out[6]: '2'

This object acts like a list and supports indexing and the len() function::

    In [7]: len(reports)
    Out[7]: 20

    In [8]: reports[0]
    Out[8]: <esp.report.Report at 0x10b2ce278>

Lets checkout that report::

    In [10]: report.id_
    Out[10]: '592'

    In [11]: report.status
    Out[11]: 'complete'

    In [12]: report.alerts
    Out[12]: <esp.resource.PaginatedCollection at 0x10b2d68d0>

Looks like it's complete and we have alerts attached to it. calling .alerts
returns a PaginatedCollection of Alert objects, lets look at one::

    In [14]: alert = report.alerts[0]

    In [15]: alert.id_
    Out[15]: '97'

    In [16]: alert.resource
    Out[16]: 'fisheye-rel-build'

    In [17]: alert.status
    Out[17]: 'pass'

    In [19]: alert.signature.name
    Out[19]: 'VPC ELB Security Groups'

In that last line we accessed the signature object by calling .signature, then
called .name on that object to get the name of the signature. Method chaining like
this makes using the ESP API data very useful and simple.

Okay, so we know a report ID we want to look up, lets try that::

    In [20]: report = esp.Report.find(1)

    In [21]: report
    Out[21]: <esp.report.Report at 0x10b2e2978>

Here we used find() again, but we passed in an ID as a argument. This did not
return a paginated collection, but instead returned an instance of the report by
itself.

So maybe we want to get a collection of signatures who check for DNS related stuff,
we can do that::

    In [22]: signatures = esp.Signature.where(name_cont='dns')

    In [23]: len(signatures)
    Out[23]: 3

    In [24]: signatures[0].name
    Out[24]: 'Global DNS TCP'

    In [25]: signatures[1].name
    Out[25]: 'Global DNS UDP'

    In [26]: signatures[2].name
    Out[26]: 'Route53 DNS'

Looks like the API gaves us 3 and all of them have DNS in the name. Good job!
where() takes parameters and converts them into search filters for ESP. There
is a list of predicates available can be found here http://api-docs.evident.io/?json#available-predicates

Predicates are used within Evident.io API search queries to determine what information to match. For instance, the cont predicate, when added to the name attribute, will check to see if name` contains a value using a wildcard query.

You can add more to where() to form complex queries::

    In [2]: esp.Suppression.where(regions_code_start='us', resource_not_null='1')
    Out[2]: <esp.resource.PaginatedCollection at 0x104a18dd8>

You can also change the combinator for complex queries from the default AND to OR by adding the m='or' parameter::

    In [5]: esp.Suppression.where(regions_code_start='us', resource_not_null='1', m='or')
