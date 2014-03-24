gstatsd - A statsd service implementation in Python + gevent.

If you are unfamiliar with statsd, you can read
`why statsd exists <http://codeascraft.etsy.com/2011/02/15/measure-anything-measure-everything/>`_,
or look at the
`NodeJS statsd implementation <https://github.com/etsy/statsd>`_.

License:
`Apache 2.0 <http://www.apache.org/licenses/LICENSE-2.0>`_

Requirements
------------


-  `Python <http://www.python.org/>`_ - I'm testing on 2.6/2.7 at
   the moment.
-  `gevent <http://www.gevent.org/>`_ - A libevent wrapper.
-  `distribute <http://pypi.python.org/pypi/distribute>`_ - (or
   setuptools) for builds.

Using gstatsd
-------------

Show gstatsd help:

::

    % gstatsd -h

Options:

::

    Usage: gstatsd [options] [config files]
    
     A statsd service in Python + gevent.
    
    Options:
      --version             show program's version number and exit
      -b BIND_ADDR, --bind=BIND_ADDR
                            bind [host]:port (host defaults to '')
      -s SINK, --sink=SINK  a service to which stats are sent
                            ([[host:]port:]type[,backend options]). Supported
                            types are "graphite", "influxdb" and "redis"
      -v                    increase verbosity (currently used for debugging)
      -f INTERVAL, --flush=INTERVAL
                            flush interval, in seconds (default 10)
      -x KEY_PREFIX, --prefix=KEY_PREFIX
                            key prefix added to all keys (default None)
      -p PERCENT, --percent=PERCENT
                            percent threshold (default 90)
      -D, --daemonize       daemonize the service
      -h, --help 

Start gstatsd listening on the default port 8125, and send stats to graphite
server on its default port (2003) every 5 seconds:

::

    % gstatsd -s graphite -f 5

Bind listener to host 'foo' port 8126, and send stats to the Graphite server on
host 'bar' port 2003 every 20 seconds:

::

    % gstatsd -b foo:8126 -s bar:2003:graphite -f 20

To send the stats to multiple graphite servers, specify '-s'
multiple times:

::

    % gstatsd -b :8125 -s stats1:2003:graphite -s stats2:2004:graphite

To send the stats influxdb server on its default port 8086 every 5 seconds:

::

    % gstatsd -s influxdb,mydb,myuser,mypass -f 5

To send the stats to both graphite and influxdb servers, specify '-s' multiple
times:

::

    % gstatsd -b :8125 -s stats1:2003:graphite -s stats2:8086:influxdb,mydb,myuser,mypass

You can also pass a yaml configuration file to gstatsd:

::

    % gstatsd ./config_example.yml

where 'config_example.yml' could be:

::

    host: localhost
    port: 8125
    daemonize: false
    verbose: false
    flush: 5
    prefix: 
    percent: 90
    sinks:
      - host: localhost
        port: 8086
        type: influxdb
        database: gstatsd
        user: gstatsd
        password: gstatsd
      - host: localhost
        port: 2003
        type: graphite


Using the client
----------------

The code example below demonstrates using the low-level client
interface:

::

    from gstatsd import client
    
    # location of the statsd server
    hostport = ('localhost', 8125)
    
    raw = client.StatsClient(hostport)
    
    # add 1 to the 'foo' bucket
    raw.increment('foo')
    
    # timer 'bar' took 25ms to complete
    raw.timer('bar', 25)

You may prefer to use the stateful client:

::

    # wraps the raw client
    cli = client.Stats(raw)
    
    timer = cli.get_timer('foo')
    timer.start()
    
    ... do some work ..
    
    # when .stop() is called, the stat is sent to the server
    timer.stop()


