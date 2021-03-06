
# Table of Contents

1.  [Installation Information](#orgddfd4d0)
    1.  [Silk Toolset](#org031a6ac)
    2.  [the type of Silk output files](#orge0e62fc)
    3.  [Filtering Flow Data and other sets](#org26672e6)


<a id="orgddfd4d0"></a>

# Installation Information


<a id="org031a6ac"></a>

## Silk Toolset

**Introduction**

SiLK (System for Internet-Level Knowledge) is a toolset that allows for efficient manageable security analysis across networks. it will help refresh your traffic knowledge and investigation. Silk may collect many gigabytes of network flow data from across the enterprise network. :

-   The Silk Flow Repository
    -   Yaf

An enterprise network comprises a variety of organizations and systems. The flow data to be handled by
SiLK are first processed by the collection system, which receives flow records from the sensors and organizes
them for later analysis SiLK is a collection of C, Python, and Perl, and as such, works in almost any UNIX- based environment. 

-   PySilk

Silk has complementary handbook of a full PySilk starter guide for those wanting to implement Silk as extension to python. Nowadays, Had some poeple produced visualization of Silk with R and Rayon([link](https://www.rsreese.com/silk-network-traffic-analysis-visualization-with-r-and-rayon/)). That's a quick and dirty way to identify traffic and help us to provide some interesting perspective on how you can  visually Research into your investigation process describe what types of evidence and questions. 

there are have some reasons important for you build own Silk platform, and why are we chose Silk set as characteristic data.

-   `libfixbuf`

    wget http://tools.netsa.cert.org/releases/libfixbuf-2.0.0.tar.gz
    tar -zxvf libfixbuf-2.0.0.tar.gz
    cd libfixbuf-2.0.0
    ./configure && make
    sudo make install

-   `Silk`

    cd ~/src
    mkdir silk
    cd silk
    
    wget https://tools.netsa.cert.org/releases/silk-3.17.2.tar.gz
    
    tar -xvf silk-3.17.2.tar.gz
    cd silk-3.17.2
    ./configure \
     --with-libfixbuf=/usr/local/lib/pkgconfig/ \
     --with-python \
     --enable-ipv6
    make && sudo make install

-   `YAF`

YAF (Yet Another Flowmeter) is a flow generation tool that offers IPFIX output  YAF was created by the CERT Network Situation Awareness (NetSA) team, who designed it for generating IPFIX records for use with SiLK.

    wget http://tools.netsa.cert.org/releases/yaf-2.10.0.tar.gz
    tar -zxvf yaf-2.10.0.tar.gz
    cd yaf-2.10.0
    export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig
    ./configure --enable-applabel
    make
    sudo make install

Use the default silk.conf file

    cd silk-3.17.0/
    sudo mkdir /data
    sudo cp site/twoway/silk.conf /data

-   Next create the sensors.conf file. Add the following lines.

`IMPORTANT`: Make sure the ipblocks below match your "internal" network blocks.

    cat <<EOF >sensors.conf
    probe S0 ipfix
     listen-on-port 18001
     protocol tcp
     listen-as-host 10.220.170.110
    end probe
    group my-network
     ipblocks 192.168.1.0/24 # address of eth0. CHANGE THIS.
     ipblocks 10.0.0.0/8 # other blocks you consider internal
    end group
    sensor S0
     ipfix-probes S0
     internal-ipblocks @my-network
     external-ipblocks remainder
    end sensor
    EOF
    
    sudo mv sensors.conf /opt/silk/

-   `Configure rwflowpack`

    cat /usr/local/share/silk/etc/rwflowpack.conf | \
    sed 's/ENABLED=/ENABLED=yes/;' | \
    sed 's/SENSOR_CONFIG=/SENSOR_CONFIG=\/data\/sensors.conf/;' | \
    sed 's/SITE_CONFIG=/SITE_CONFIG=\/data\/silk.conf/' | \
    sed 's/LOG_TYPE=syslog/LOG_TYPE=legacy/' | \
    sed 's/LOG_DIR=.*/LOG_DIR=\/var\/log/' | \
    sed 's/CREATE_DIRECTORIES=.*/CREATE_DIRECTORIES=yes/' \
    >> rwflowpack.conf
    sudo mv rwflowpack.conf /usr/local/etc/

Next copy the start up script into /etc/init.d and set it to start on boot. 

    sudo cp /usr/local/share/silk/etc/init.d/rwflowpack /etc/init.d
    sudo sudo update-rc.d rwflowpack start 20 3 4 5 .
    sudo service rwflowpack start

First, Configure the firewall 

Lets allow YAF to talk to rwflowpack by allowing port 18001 in.
the rules must be saved in the file /etc/iptables/rules.v4 for IPv4 and /etc/iptables/rules.v6 for IPv6.

    sudo iptables -I INPUT -s 127.0.0.1 -p tcp -m tcp --dport 18001 -j ACCEPT
    sudo service iptables save

    sudo nohup /usr/local/bin/yaf --silk --ipfix=tcp --live=pcap  --out=127.0.0.1 --ipfix-port=18001 --in=ens192 --applabel --max-payload=384 &

Run a test query 

    /usr/local/bin/rwfilter --sensor=S0 --proto=0-255 --pass=stdout --type=all | rwcut | tail


<a id="orge0e62fc"></a>

## the type of Silk output files

    /data % tree -l
    .
    ├── ext2ext
    │   └── 2018
    │       └── 02
    │           └── 27
    │               ├── ext2ext-S0_20180227.16
    │               └── ext2ext-S0_20180227.17
    ├── in
    │   └── 2018
    │       └── 02
    │           └── 27
    │               └── in-S0_20180227.16
    ├── int2int
    │   └── 2018
    │       └── 02
    │           └── 27
    │               ├── int2int-S0_20180227.16
    │               └── int2int-S0_20180227.17
    ├── out
    │   └── 2018
    │       └── 02
    │           └── 27
    │               ├── out-S0_20180227.16
    │               └── out-S0_20180227.17
    ├── sensors.conf
    └── silk.conf

-   Ext2ext: From an external network to the same, or another external network
-   Int2int: From an internal network to the same, or another internal network
-   In: Inbound to a device on an internal network using either port 80, 443,

or 8080.

-   Out: Outbound to a device on an external network using either port 80, 443, or 8080.


<a id="org26672e6"></a>

## Filtering Flow Data and other sets

-   rwcut
-   rwstats

-rwfilter
The rwfilter usually employ in really Nsm environment. So it is imortant to understand and  manipulates it.
In addition, rwfilter can be created as C or pySilk plugin loaded into.

-   rwgeoip2ccmap
    -   Create a country code prefix map from a GeoIP

-   The GeoIP2 or GeoLite2 comma-separated value (CSV) files
-   The GeoIP2 or GeoLite2 binary database file when SiLK is built the libmaxminddb support
-   The GeoIP Legacy or GeoLite Legacy CSV file
-   The GeoIP Legacy or GeoLite Legacy binary file

    ;; First downloaded geoipdata database
    
    wget http://geolite.maxmind.com/download/geoip/database/GeoLite2-Country-CSV.zip
    unzip GeoLite2-Country-CSV.zip
    rwgeoip2ccmap --input-path=GeoLite2-Country-CSV_20181127 --output-path=country_codes.pmap
    udo cp country_codes.pmap /usr/local/share/silk/country_codes.pmap
    
    ;;Secondly, commanding Unzip the database and convert it into the rwgeoip
    gzip -d -c Geoip.dat.gz | rwgeoip2ccmap --encoded -input>country_code.pmap
    cp country_code.pmap /usr/local/share/silk/
    
    ;;IPv6 Comma Separated Values File
    gzip -d -c GeoIPv6.csv.gz | \
           rwgeoip2ccmap --mode=ipv6 > country_codes.pmap

