# rblmon


Rblmon monitors your IP address or list of IP addresses on a specific DNS Blacklist server or list of servers. It allows you to specify your list of IP and DNSBL servers. By default, it prints all results in a more user-friendly way, but for further processing, you can use a 'quiet' option.



![rblmon](https://raw.githubusercontent.com/bkzk/screens/master/rblmon/rblmon.png)



### Installation / Requirements

Clone the repository and copy it to one of globally 
accessible paths and set executable bit.

    git clone https://github.com/bkzk/rblmon.git

    cd rblmon
    install -m0755 rblmon /usr/local/bin

It requires a DNS tool called 'dig' which can be found in a most modern Linux distributions.  

    #debian/ubuntu
    apt-get install dnsutils
    
    #centos/red-hat
    yum install bind-utils

Install two extra Perl modules, they are not mandatory but the script works better with them.

*  **`Data::Validate::IP`**
*  **`Data::Validate::Domain`**

These modules should be found in your distro repositories, if not you can install them using a `cpan` tool.

    #debian/ubuntu
    apt-get install libdata-validate-ip-perl libdata-validate-domain-perl
    
    #centos/redhat
    yum -y install perl-Data-Validate-IP perl-Data-Validate-Domain
    
    #cpan - compile
    cpan -i Data::Validate::IP Data::Validate::Domain


### Usage

    rblmon -h [ip|hostname|file] [-d hostname|file] [-r ip|hostname] [-q]
              [-log|-logall] [-l filename] [-ld dirname] [-man|-help]

Use `-help` or `-man` argument to get more information. You may need to install a `perl-doc` package to read perl-based manuals.

### Description

If there is no DNSBL server specified at the command line, a default built-in list of popular DNSBL servers is used. 

__Note__: I do NOT maintain that built-in list anymore. There is a high chance number of those servers do not exist anymore. This may result in a positive FOUND match. 

    $ rblmon -h srv01.example.net

    Processing list of DNSBL servers (built-in) ... 
    Checkin [ srv01.example.net ] in:
    
                 cbl.abuseat.org |    CLEAR  
                 dnsbl.sorbs.net |    CLEAR  
            spam.dnsbl.sorbs.net |    CLEAR  
                  bl.spamcop.net |    CLEAR  
                pbl.spamhaus.org |    CLEAR  
                sbl.spamhaus.org |    CLEAR  
                xbl.spamhaus.org |    CLEAR  
                zen.spamhaus.org |    CLEAR  
            sbl-xbl.spamhaus.org |    CLEAR  
              ubl.unsubscore.com |    CLEAR  
          dnsbl-1.uceprotect.net |    CLEAR  
          dnsbl-2.uceprotect.net |    CLEAR  
          dnsbl-3.uceprotect.net |    CLEAR  
               dyna.spamrats.com |    FOUND  
              noptr.spamrats.com |    CLEAR  
               spam.spamrats.com |    CLEAR  
          b.barracudacentral.org |    CLEAR  
           ips.backscatterer.org |    FOUND  
              truncate.gbudb.net |    CLEAR  
             bl.spamcannibal.org |    CLEAR  
                 rbl.megarbl.net |    CLEAR  

    Summary: srv01.example.net is FOUND on [ 2/20 ]


To verify single host status on a specific DNSBL server.

    $ rblmon -h 192.0.2.2 -d dnsbl.sorbs.net

To verify the single host status on your list of DNSBL servers.

    $ rblmon -h srv01.example.net -d dnsblserver.txt -r 8.8.8.8

When you have a pool of IP addresses or you need to verify multiple hosts you may use a file with the list of IPs o hostnames (one per line) eg. myservers.txt. You can verify this list against a specific DNSBL server or a list of DNSBL servers stored in a file eg. dnsblserver.txt .

    $ rblmon -h myservers.txt -d dnsblserver.txt -r 8.8.8.8
   
    Resolver: @8.8.8.8
    Processing list of DNSBL servers (myservers.txt) ... 
    Processing list of hosts (dnsblserver.txt) ...
    
    Checkin [              srv01.example.net ]   47/47   |     CLEAR  
    Checkin [              srv02.example.net ]   47/47   |     FOUND      [2] 
    Checkin [              srv03.example.net ]   47/47   |     CLEAR  
    Checkin [              srv04.example.net ]   47/47   |     FOUND      [2] 
    Checkin [              srv05.example.net ]   47/47   |     FOUND      [1] 
    Checkin [              srv06.example.net ]   47/47   |     FOUND      [3] 
    Checkin [              srv07.example.net ]   47/47   |     CLEAR  
    Checkin [              srv08.example.net ]   47/47   |     CLEAR  
    Checkin [              srv09.example.net ]   47/47   |     CLEAR  
    Checkin [              srv10.example.net ]   47/47   |     FOUND      [1] 
    Checkin [              srv11.example.net ]   47/47   |     CLEAR  
    Checkin [              srv12.example.net ]   47/47   |     CLEAR  
    Checkin [              srv13.example.net ]   47/47   |     CLEAR  
    Checkin [              srv14.example.net ]   47/47   |     CLEAR  
    Checkin [              srv15.example.net ]   47/47   |     CLEAR  
    Checkin [              srv16.example.net ]   47/47   |     FOUND      [1] 
    Checkin [              srv17.example.net ]   47/47   |     FOUND      [1] 
   
    Summary: FOUND 7 of 17 hosts on DNSBL

Setting resolver from the command line allows you to use a different DNS server than defined by your system  (eg. /etc/resolv.conf).

If there is a need for further processing, use a **`--quiet`** parameter to only retrieve statuses.

    $ rblmon -h srv01.example.net -d dnsblserver.txt -r 8.8.8.8 -q
    srv01.example.net: 0
    srv02.example.net: 2
    srv03.example.net: 0
    srv04.example.net: 2
    srv05.example.net: 1
    srv06.example.net: 3
    srv07.example.net: 0
    srv08.example.net: 0
    srv09.example.net: 0
    srv10.example.net: 1
    srv11.example.net: 0
    srv12.example.net: 0
    srv13.example.net: 0
    srv14.example.net: 0
    srv15.example.net: 0
    srv16.example.net: 1
    srv17.example.net: 1 

#### Logging option

To log some information use **`-log`** or **`-log-all`** . The difference between this two is that the first of these contains only hosts present on at least one blacklist and the other contains all the hosts and their statuses.

    #-log
    srv02.example.net: sbl.spamhaus.org, zen.spamhaus.org
    srv04.example.net: cbl.abuseat.org, ips.backscatterer.org
    srv05.example.net: ips.backscatterer.org
    srv06.example.net: xbl.spamhaus.org, zen.spamhaus.org, sbl-xbl.spamhaus.org
    srv10.example.net: b.barracudacentral.org
    srv16.example.net: b.barracudacentral.org
    srv17.example.net: dnsbl-1.uceprotect.net
    
    #-log-all
    srv01.example.net: CLEAR
    srv02.example.net: sbl.spamhaus.org, zen.spamhaus.org
    srv03.example.net: CLEAR
    srv04.example.net: cbl.abuseat.org, ips.backscatterer.org
    srv05.example.net: ips.backscatterer.org
    srv06.example.net: xbl.spamhaus.org, zen.spamhaus.org, sbl-xbl.spamhaus.org
    srv07.example.net: CLEAR
    srv08.example.net: CLEAR
    srv09.example.net: CLEAR
    srv10.example.net: b.barracudacentral.org
    srv11.example.net: CLEAR
    srv12.example.net: CLEAR
    srv13.example.net: CLEAR
    srv14.example.net: CLEAR
    srv15.example.net: CLEAR
    srv16.example.net: b.barracudacentral.org
    srv17.example.net: dnsbl-1.uceprotect.net

The default name of the log file is `dnsbl-YYYY-MM-DD.log` and is stored in the current working directory. You can change the name of the file using  **`-l`** switch and, or the destination of the log directory using the  **`-ld`** switch.

### Cron

To keep logs, you can create a cron job to do it. In the example below all logs will be stored in /var/log/rbl/, so make sure this directory exists and that the user running the cron has the write permissions. Set a proper path to your server and to the DNSBL list (one IP or hostname per line).

    crontab -e 
    30 0 * * * rblmon -h /path/to/server.list -d /path/to/dnsbl.list -q -log-all -ld /var/log/rbl/ &>/dev/null
