# Puppet DNS (BIND9) Module

[![Build Status](https://travis-ci.org/SourceDoctor/puppet-dns.png?branch=master)](https://travis-ci.org/SourceDoctor/puppet-dns)

Module for provisioning DNS (bind9)

Supports:

* Ubuntu: 17.04, 16.04, 14.04, 12.04
* Debian: 9, 8
* CentOS: 7.x, 6.x

## About
This is a partial rewrite of [ajjahn puppet-dns](https://github.com/ajjahn/puppet-dns)
for supporting hiera


## Usage

```puppet
node 'server.example.com' {

  # DNS Settings and Zone Configuration
  class { 'dns':
    forwarders => [ '8.8.8.8',
                    '8.8.4.4' ],
    zone       => { 'example.com' => {
                        soa         => 'ns1.example.com',
                        soa_email   => 'admin.example.com',
                        nameservers => ['ns1']
                      },
                    'example2.com' => {
                        soa         => 'ns2.example2.com',
                        soa_email   => 'admin.example2.com',
                        nameservers => ['ns2']
                      },
      }
  }

  # A Records:
  dns::record::a {
    'huey':
      zone => 'example.com',
      data => ['98.76.54.32'];
    'duey':
      zone => 'example.com',
      data => ['12.34.56.78', '12.23.34.45'];
    'luey':
      zone => 'example.com',
      data => ['192.168.1.25'],
      ptr  => true; # Creates a matching reverse zone record.  Make sure you've added the proper reverse zone in the manifest.
  }

  # MX Records:
  dns::record::mx {
    'mx,0':
      zone       => 'example.com',
      preference => 0,
      data       => 'ASPMX.L.GOOGLE.com';
    'mx,10':
      zone       => 'example.com',
      preference => 10,
      data       => 'ALT1.ASPMX.L.GOOGLE.com';
  }

  # NS Records:
  dns::record::ns {
    'example.com':
      zone => 'example.com',
      data => 'ns3';
    'delegation-to-ns4-jp-example-net':
      zone => 'example.com',
      host => 'delegated-zone',
      data => 'ns4.jp.example.net.';
  }

  # CNAME Record:
  dns::record::cname { 'www':
    zone => 'example.com',
    data => 'huey.example.com',
  }

  # TXT Record:
  dns::record::txt { 'www':
    zone => 'example.com',
    data => 'Hello World',
  }

  # TSIG
  class { 'dns':
    tsig => { 'ns3' :
                ensure    => present,
                algorithm => "hmac-md5",
                secret    => "La/E5CjG9O+os1jq0a2jdA==",
                server    => "192.168.1.3"
            }
  }

}
```

You can also declare forwarders for a specific zone, if you don't have one in the dns::option.

```puppet
  class { 'dns':
    zone => { 'example.com' => {
                  soa         => 'ns1.example.com',
                  soa_email   => 'admin.example.com',
                  allow_forwarder => ['8.8.8.8'],
                  forward_policy  => 'first',
                  nameservers => ['ns1']
                },
      }
  }
```

You can change the checking of the domain name. The policy can be either warn fail or ignore.

```puppet
  class { 'dns':
    check_names_master => 'fail',
    check_names_slave  => 'warn',
    forwarders => [ '8.8.8.8',
                    '8.8.4.4' ],
  }
```

You can enable the report of bind stats trough the `statistics-channels` using:

```puppet
  class { 'dns':
      check_names_master     => 'fail',
      check_names_slave      => 'warn',
      forwarders             => [ '8.8.8.8', '4.4.4.4' ],
      statistic_channel_ip   => '127.0.0.1',
      statistic_channel_port => 8053
  }
```

You can also create dynamic zones. Mind they are only created once by puppet and never replaced unless allow_update is empty.

```puppet
dns::zone {
}

  class { 'dns':
    zone => { 'example.com' => {
              soa             => 'ns1.example.com',
              soa_email       => 'admin.example.com',
              allow_forwarder => ['8.8.8.8'],
              allow_update    => ['192.168.1.2', '192.168.1.3'],
              forward_policy  => 'first',
              nameservers     => ['ns1'],
            },
        }
  }
```

## License

This module is released under the MIT license:

* [http://www.opensource.org/licenses/MIT](http://www.opensource.org/licenses/MIT)
