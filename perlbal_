#!/usr/bin/env python
"""
Munin script to monitor perlbal.

Following the munin style, this reads argv[0] to get the name of the pool
to monitor.

Currently reports raw requests/second, but would be cool to break it down
by response code/pool...
"""

import telnetlib
import socket
import munin

class Perlbal(munin.Plugin):
    
    env_vars = {
        "host": "localhost",
        "port": "60000"
    }
    
    def _connect(self):
        return telnetlib.Telnet(self.env["host"], int(self.env["port"]))
    
    def fetch(self, pool=None):
        if not pool: return []
        
        c = self._connect()
        c.write("show pool %s\r\n"% pool)
        data = c.read_until(".\r\n")
        nodelines = data.strip().split("\r\n")[:-1]
        nodes = (line.strip().split() for line in nodelines)
        return [("requests.value", sum(int(reqs) for node, reqs in nodes))]
    
    def config(self, pool=None):
        if not pool: return []
        return [
            ('graph_title', 'Perlbal requests in pool %s' % pool),
            ('graph_args', '-l 0 --base 1000'),
            ('graph_vlabel', 'Requests'),
            ('graph_category', 'Perlbal'),
            ('graph_info', 'Tracks the number of requests the %s pool handles' % pool),
            ('graph_period', 'second'),
            ('requests.label', 'Requests per second'),
            ('requests.type', 'DERIVE'),
            ('requests.min', '0'),
        ]

    def autoconf(self):
        try:
            self._connect()
        except socket.error:
            return False
        return True
    
    def suggest(self):
        c = self._connect()
        c.write("show pool\r\n")
        data = c.read_until(".\r\n").strip()
        return set(line.strip().split()[0] for line in data.split("\r\n") if line != '.')
    
if __name__ == '__main__':
    munin.run(Perlbal)