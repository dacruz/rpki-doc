.. _doc_routinator_configuration:

Configuration 
=============

Routinator has a number of default settings, such as the location where files
are stored, the refresh interval, and the log level. You can view these
settings by running:

.. code-block:: text

   routinator config 
   
It will return the list of defaults in the same notation that is used by the optional configuration file, which will be largely similar to this:

.. code-block:: text

   exceptions = []
   expire = 7200
   history-size = 10
   http-listen = []
   log = "default"
   log-level = "WARN"
   refresh = 3600
   repository-dir = "/Users/me/.rpki-cache/repository"
   retry = 600
   rsync-command = "rsync"
   rsync-count = 4
   rsync-timeout = 600
   rtr-listen = []
   strict = false
   syslog-facility = "daemon"
   systemd-listen = false
   tal-dir = "/Users/me/.rpki-cache/tals"
   validation-threads = 4

You can override these defaults, as well as configure a great number of
additional options using either command line arguments or via the 
configuration file. 

To get an overview of all available options, please refer to the manual 
page, which can be viewed by running ``routinator man``. Alternatively,
the manual page is also available on the `NLnet Labs website <https://www.nlnetlabs.nl/documentation/rpki/routinator/>`_.

Using a Configuration File
--------------------------

Routinator can take its configuration from a file. You can specify such a
config file via the ``-c`` option. If you don’t, Routinator will check
if there is a file ``$HOME/.routinator.conf`` and if it exists, use it. If it
doesn’t exist and there is no ``-c`` option, the default values are used.

For specifying configuration options, Routinator uses a `TOML file
<https://github.com/toml-lang/toml>`_. Its entries are named similarly to the
command line options. A complete sample configuration file showing all the 
default values can be found in the repository at `etc/routinator.conf
<https://github.com/NLnetLabs/routinator/blob/master/etc/routinator.conf>`_.

For example, if you want Routinator to refresh every 15 minutes and run as
an RTR server on 192.0.2.13 and 2001:0DB8::13 on port 3323, in addition to
providing an HTTP server on port 9556, simply take the output from 
``routinator config`` and change the ``refresh``, ``rtr-listen`` and
``http-listen`` values in your favourite text editor:

.. code-block:: text

   exceptions = []
   expire = 7200
   history-size = 10
   http-listen = ["192.0.2.13:9556", "[2001:0DB8::13]:9556"]
   log = "default"
   log-level = "WARN"
   refresh = 900
   repository-dir = "/Users/me/.rpki-cache/repository"
   retry = 600
   rsync-command = "rsync"
   rsync-count = 4
   rsync-timeout = 600
   rtr-listen = ["192.0.2.13:3323", "[2001:0DB8::13]:3323"]
   strict = false
   syslog-facility = "daemon"
   systemd-listen = false
   tal-dir = "/Users/me/.rpki-cache/tals"
   validation-threads = 4

After saving this file as ``.routinator.conf`` in your home directory, you can 
start Routinator with:

.. code-block:: bash

   routinator server

Applying Local Exceptions
-------------------------

In some cases, you may want to override the global RPKI data set with your own
local exceptions. For example, when a legitimate route announcement is
inadvertently flagged as *invalid* due to a misconfigured ROA, you may want to
temporarily accept it to give the operators an opportunity to resolve the
issue.

You can do this by specifying route origins that should be filtered out of the
output, as well as origins that should be added, in a file using JSON notation
according to the SLURM standard specified in `RFC 8416
<https://tools.ietf.org/html/rfc8416>`_.

A full example file is provided below. This, along with an empty one is
available in the repository at `/test/slurm
<https://github.com/NLnetLabs/routinator/tree/master/test/slurm>`_.

.. code-block:: json

   {
     "slurmVersion": 1,
     "validationOutputFilters": {
      "prefixFilters": [
        {
         "prefix": "192.0.2.0/24",
         "comment": "All VRPs encompassed by prefix"
        },
        {
         "asn": 64496,
         "comment": "All VRPs matching ASN"
        },
        {
         "prefix": "198.51.100.0/24",
         "asn": 64497,
         "comment": "All VRPs encompassed by prefix, matching ASN"
        }
      ],
      "bgpsecFilters": [
        {
         "asn": 64496,
         "comment": "All keys for ASN"
        },
        {
         "SKI": "Zm9v",
         "comment": "Key matching Router SKI"
        },
        {
         "asn": 64497,
         "SKI": "YmFy",
         "comment": "Key for ASN 64497 matching Router SKI"
        }
      ]
     },
     "locallyAddedAssertions": {
      "prefixAssertions": [
        {
         "asn": 64496,
         "prefix": "198.51.100.0/24",
         "comment": "My other important route"
        },
        {
         "asn": 64496,
         "prefix": "2001:DB8::/32",
         "maxPrefixLength": 48,
         "comment": "My other important de-aggregated routes"
        }
      ],
      "bgpsecAssertions": [
        {
         "asn": 64496,
         "comment" : "My known key for my important ASN",
         "SKI": "<some base64 SKI>",
         "routerPublicKey": "<some base64 public key>"
        }
      ]
     }
   }
   
Use the ``-x`` option to refer to your file with local exceptions. Routinator 
will re-read that file on every validation run, so you can simply update the 
file whenever your exceptions change.