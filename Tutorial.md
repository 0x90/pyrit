# Introduction #

This document will guide you through your first steps with Pyrit. **Before continuing, you should have Pyrit installed and working.** See the [Installation](Installation.md)-Wiki for details. You will also need to have [Scapy](http://www.secdev.org/projects/scapy/) installed, which should come with your distribution or may be installed from source. Pyrit can use [SQLAlchemy](http://www.sqlalchemy.org/) to access various kinds of SQL-databases and you'll need to have it installed if you want to try that feature as explained below.

You should also take a look at the [manual](ReferenceManual.md) when new commands get introduced below; more information and details about the features a command provides are given there.

Throughout this tutorial we will refer to files and examples that are distributed together with Pyrit's source-code. Therefore the first step is to get yourself a copy of [the source-code tarball](http://pyrit.googlecode.com/files/pyrit-0.3.0.tar.gz), unpack it and switch to the /test-directory:
```
wget http://pyrit.googlecode.com/files/pyrit-0.3.0.tar.gz
tar xvzf pyrit-0.3.0.tar.gz
cd pyrit-0.3.0/test
```

You should find three files within this directory that will be of interest for us:
  * _dict.gz_ is a gzip-compressed wordlist
  * _wpa2psk-linksys.dump.gz_ is a gzip-compressed dump of a WPA2-PSK handshake
  * _wpapsk-linksys.dump.gz_ is a gzip-compressed dump a WPA-PSK handshake


# First steps with capture-files and wordlists #

Pyrit can understand packet capture files in pcap-format. These files basically contain what was captured from the air. Our first meaningful step in this tutorial is to let Pyrit analyze one of the capture files and give us some information about the content.

## Analyzing a capture file ##

Issue to following command to analyze the file _wpapsk-linksys.dump.gz_:
```
pyrit -r wpapsk-linksys.dump.gz analyze
```

Pyrit should answer with output very similar like the following:
```
Pyrit 0.3.0 (C) 2008-2010 Lukas Lueg http://pyrit.googlecode.com
This code is distributed under the GNU General Public License v3+

Parsing file 'wpapsk-linksys.dump.gz' (1/1)...
587 packets (587 802.11-packets), 1 APs

#1: AccessPoint 00:0b:86:c2:a4:85 ('linksys')
  #0: Station 00:13:ce:55:98:ef, handshake found
  #1: Station 01:00:5e:7f:ff:fa
  #2: Station 01:00:5e:00:00:16
```

Pyrit has successfuly parsed the capture file and found one AccessPoint with BSSID _00:0b:86:c2:a4:85_ and ESSID _'linksys'_ and three Stations communicating with that AccessPoint. The key-negotiation (known as the fourway-handshake) between the Station with MAC _00:13:ce:55:98:ef_ and the AccessPoint has also been recorded in the capture file. We can use the data from this handshake to guess that password that is used to protect the network.

**Please note** that Pyrit can transparently read/write gzip-compressed files; this becomes very handy when dealing with large wordlists or cowpatty-files that may take hundrets of megabytes.


## Attacking a handshake and revealing the password ##

We now use the example wordlist _dict.gz_ and let Pyrit guess the password that was used in the key-negotiation between AccessPoint _00:0b:86:c2:a4:85_ and Station _00:13:ce:55:98:ef_. The correct password should get detected, if it is part of the list. In our terms, this is known as a "passthrough-attack". Issue the following command:

```
pyrit -r wpapsk-linksys.dump.gz -i dict.gz -b 00:0b:86:c2:a4:85 attack_passthrough
```

This tells Pyrit to take the capture-file _wpapsk-linksys.dump.gz_ and attack the key-negotiation with AccessPoint _00:0b:86:c2:a4:85_ using the dictionary-file _dict.gz_.

**Please note** that you do not always have to tell Pyrit which AccessPoint to choose from the capture-file - Pyrit will usually be able to figure that out by itself.

You should get a response very similar to the following:
```
Pyrit 0.3.0 (C) 2008-2010 Lukas Lueg http://pyrit.googlecode.com
This code is distributed under the GNU General Public License v3+

Parsing file 'wpapsk-linksys.dump.gz' (1/1)...
587 packets (587 802.11-packets), 1 APs

Tried 4091 PMKs so far; 935 PMKs per second.

The password is 'dictionary'.
```

We've successfully revealed that the password used to protect the network _00:0b:86:c2:a4:85_ is _"dictionary"_...


## Interlude: Stripping a capture-file from unnecessary cruft ##

Capture-files are usually simple dumps of the traffic captured directly from the air. For our purpose, we are only interested in a very tiny fraction of the traffic between AccessPoint and Station. Pyrit can help reducing the size of a packet-capture file by analyzing the traffic and throwing away all packets that are of no use for us. We end up with a new, very small capture file that still holds all valuable information and is useable with other tools like _Wireshark_.

**Please note** that stripping a capture file is not necessary. It's sole purpose is to make life a little easier when it comes to large capture files.

Our original example has 587 packets and a size of roughly 13kb. Issue the following command:

```
pyrit -r wpapsk-linksys.dump.gz -o wpapsk-linksys_stripped.dump.gz strip
```

You should get a response like the following:

```
Pyrit 0.3.0 (C) 2008-2010 Lukas Lueg http://pyrit.googlecode.com
This code is distributed under the GNU General Public License v3+

Parsing file 'wpapsk-linksys.dump.gz' (1/1)...
587 packets (587 802.11-packets), 1 APs

#1: AccessPoint 00:0b:86:c2:a4:85 ('linksys')
  #0: Station 00:13:ce:55:98:ef (1 authentications)

New pcap-file 'wpapsk-linksys_stripped.dump.gz' written (4 out of 587 packets)
```

The new capture file _wpapsk-linksys\_stripped.dump.gz_ has a size of only a few hundred bytes and contains only three from the key-negotiation (used to attack the password) and one beacon-frame (used to detect the network's ESSID).


# Working with Pyrit's database #

As you may already know, guessing the password used in a WPA(2)-PSK key-negotiation is a computational-intensive task. During this process, more than 99.9% of the CPU-cycles have to be spent to compute what is known as the _Pairwise Master Key_, a 256-bit key derived from the ESSID and a password using the PBKDF2-HMAC-SHA1-algorithm. One of the major weaknesses of WPA(2)-PSK is that the _Pairwise Master Key_ has no elements that are unique to the moment of the key-negotiation between AccessPoint and Station. It is therefor possible to pre-compute the _Pairwise Master Key_ and store it for later use. In the moment of attacking a key-negotiation, we are left with the remaining 0.1% of what depends on session-unique data. It is therefore extremely valueable for an attacker to pre-compute large tables of _Pairwise Master Keys_ for common ESSIDs.

This is where Pyrit's database kicks in. It can store ESSIDs, passwords and their corresponding _Pairwise Master Keys_, possibly growing to the size of hundrets of millions of entries. Starting with a fresh installation of Pyrit, your database will most probably be empty. Issue the following command to get an overview:
```
pyrit eval
```

Pyrit should respond like this:
```
Pyrit 0.3.0 (C) 2008-2010 Lukas Lueg http://pyrit.googlecode.com
This code is distributed under the GNU General Public License v3+

Connecting to storage at 'file://'...  connected.
Passwords available: 0
```

Nothing fancy to see here, yet.

**Please note** the default filesystem-based storage _'file://'_. We'll come to SQL-databases later on.

## Populating and batch-processing the database ##

In order to make the database usefull, we'll populate it with passwords from a wordlist. Issue the following command:
```
pyrit -i dict.gz import_passwords
```

Pyrit will read the file _'dict.gz'_ and store the wordlist in it's internal database format. You should get a response like the following:
```
Pyrit 0.3.0 (C) 2008-2010 Lukas Lueg http://pyrit.googlecode.com
This code is distributed under the GNU General Public License v3+

Connecting to storage at 'file://'...  connected.
10202 lines read. Flushing buffers... 
All done.
```

**Please note** that you can add more passwords to the database later on; the command _'import\_passwords'_ ensures that duplicates within the wordlist or between the wordlist and the database are tossed out and not stored again. For now, run the _'eval'_-command again to see how the database has been populated with passwords from _'dict.gz'_. You should get output similar to this:
```
Pyrit 0.3.0 (C) 2008-2010 Lukas Lueg http://pyrit.googlecode.com
This code is distributed under the GNU General Public License v3+

Connecting to storage at 'file://'...  connected.
Passwords available: 4078
```

You'll notice that Pyrit has only stored 4,078 out of the 10,202 passwords from the file. Pyrit has automatically filtered passwords that are not suitable for WPA(2)-PSK and also sorted out duplicates. Now that we have some passwords in the database, we have to create an ESSID. Issue the following command:
```
pyrit -e linksys create_essid
```

... and you'll get an output like this:
```
Pyrit 0.3.0 (C) 2008-2010 Lukas Lueg http://pyrit.googlecode.com
This code is distributed under the GNU General Public License v3+

Connecting to storage at 'file://'...  connected.
Created ESSID 'linksys'
```

Run the _'eval'_-command again and you'll see that ESSID _'linksys'_ has been created in the database:
```
Pyrit 0.3.0 (C) 2008-2010 Lukas Lueg http://pyrit.googlecode.com
This code is distributed under the GNU General Public License v3+

Connecting to storage at 'file://'...  connected.
Passwords available: 4078

ESSID 'linksys' : 0 (0.00%)
```

The database now contains enough information to start batch-processing it. Pyrit will take all (ESSID:password)-combinations, compute the corresponding _Pairwise master Keys_ and store those for later use.

**Please note** that you can stop Pyrit's batch-processing at any time (with ctrl+c or sending SIGTERM). Pyrit will start at the point where it stopped the next time you start batch-processing. Issue to following command:
```
pyrit batch
```

... and watch how Pyrit crunches through the database until it runs out of work:
```
Pyrit 0.3.0 (C) 2008-2010 Lukas Lueg http://pyrit.googlecode.com
This code is distributed under the GNU General Public License v3+

Connecting to storage at 'file://'...  connected.
Working on ESSID 'linksys'
Processed all workunits for ESSID 'linksys'; 1035 PMKs per second.

Batchprocessing done.
```

You can use the _'eval'_-command once more to see that all workunits for ESSID _'linksys'_ have been computed.


## Using the database to attack a handshake ##

We can now use the _Pairwise Master Keys_ stored in the database to attack the same handshake as in the example above. Instead of running a "passthrough-attack", where the database is not touched at all, we issue a "database-attack" like the following:

```
pyrit -r wpapsk-linksys.dump.gz attack_db
```

**Please note** that we did neither specify the network's ESSID nor it's BSSID.

You should get a response very similar to the following:
```
Pyrit 0.3.0 (C) 2008-2010 Lukas Lueg http://pyrit.googlecode.com
This code is distributed under the GNU General Public License v3+

Connecting to storage at 'file://'...  connected.
Parsing file 'wpapsk-linksys.dump.gz' (1/1)...
587 packets (587 802.11-packets), 1 APs

Picked AccessPoint 00:0b:86:c2:a4:85 ('linksys') automatically.
Attacking handshake with Station 00:13:ce:55:98:ef...
Tried 1639 PMKs so far (39.8%); 1577435 PMKs per second.

The password is 'dictionary'.
```

Again, the password protecting the network has been revealed.

While our example uses an extremely small wordlist and the performance-numbers are thereby not very reliable, attacking a handshake from a database of pre-computed _Pairwise Master Keys_ will usually crunch through more than one million passwords per second. You can also run a database-attack against the second capture file _'wpa2psk-linksys.dump.gz'_, which will also take use of the pre-computed _Pairwise Master Keys_.


## Scaling up: Using a SQL-database as storage ##

Using a SQL-database instead of the filesystem will give you some benefits:

  * Real ACID-compliance, backup- and load-balancing-features.
  * Multiple Pyrit-clients can operate on the same database at the same time over the network.
  * Meta- and binary-data are (possibly) stored independent of each other, making the database easier to query and operate on.

Pyrit uses [SQLAlchemy](http://www.sqlalchemy.org/) and can therefor use all kinds of SQL-databases for it's internal storage mechanism: [SQLite](http://www.sqlite.org/) has all the benefits described above (except the network-functionality), [MySQL](http://dev.mysql.com/) and [PostgreSQL](http://www.postgresql.org/) require some setup but provide more features and better scaling. Please refer to [SQLAlchemy](http://www.sqlalchemy.org/)'s documentation for more details about supported databases.

Using a database as storage is extremely easy - all you got to do is to provide an alternative _connection-string_ instead of _'file://'_ that Pyrit uses by default (please refer to the [manual](ReferenceManual.md) for details about the connection-string). In the following example, we use a SQLite-database stored in the single file _'mydb.db'_:

```
pyrit -u sqlite:///mydb.db -i dict.gz import_passwords
```

**Please note** that we do not have to care about creating the database (in the case of SQLite) or any tables within it. Pyrit will take care of this. You should get an output very similar to this:
```
Pyrit 0.3.0 (C) 2008-2010 Lukas Lueg http://pyrit.googlecode.com
This code is distributed under the GNU General Public License v3+

Connecting to storage at 'sqlite:///mydb.db'...  connected.
10202 lines read. Flushing buffers... 
All done.
```

Setting up a MySQL- or PostreSQL-server is beyond the scope of this tutorial. However, after setting up the database-server, creating a (empty) database and providing the necessary credentials, the required steps in Pyrit are the same as above. For example, to use the (already created) database _'pyrit'_ on a PostgreSQL-server at 192.168.0.7 with user _'pyrit'_ and no password, your commandline would look something like this:

```
pyrit -u postgres://pyrit:@192.168.0.7/pyrit -e linksys create_essid
```

To make life a little easier, you can save the default connection-string in Pyrit's configuration-file at _'~/.pyrit/config'_. Change the value of the key _default\_storage_ to a new connection-string and you won't have to supply it every single time.