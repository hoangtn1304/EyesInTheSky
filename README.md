# Get eyes in the sky with your Raspberry Pi

Did you know that you can use your Raspberry Pi to _get eyes in the sky_? By tuning into radio signals emitted from planes up to 250 miles away from your location you can track flights and it only takes a few minutes and a cheap USB TV stick to get started.

![Lead image of RPi and dongle](https://blog.alexellis.io/content/images/2017/07/IMG_1424_1080.jpg) _Pictured: dump1090 - testing FlightAware Antenna vs a quarter-wave whip and Cantenna_

This guide will give you a brief introduction into _flight tracking_ - looking at the software, hardware and most importantly the terminology and jargon you need to know. I'll also show you how **[Docker](http://blog.alexellis.io/tag/docker/) and containers** make a great combination for managing your software in IoT projects.

Sites like [FlightAware.com](http://flightaware.com/about/) have been able to track 10s of thousands of aircraft by crowd-sourcing the task out to people all over the world equipped with their $35 Raspberry Pis and cheap USB TV tuners.

> You can keep the signals you pick up to yourself or contribute them to a real-time tracking site like [FlightAware.com](http://flightaware.com/about/), [FlightRadar24](https://www.flightradar24.com) and [PlaneFinder.net](Planefinder.net) - in return you get detailed metrics - RADAR readouts and other rewards.

![FlightRadar](https://blog.alexellis.io/content/images/2017/07/fr-sample.jpg)

_Pictured: FlightRadar showing planes over UK airspace_

Why track flights?

*   Super low-cost and practical application for your unused or dusty Raspberry Pis
*   Contribute to online flight trackers and get onto your local leaderboards
*   Accumulate detailed statistics and find the best antenna and position for your setup
*   Use the data for your own programming projects and learn about aviation at the same time

> Most of all - it's a fun project to try with your Pi and it offers instant gratification. The best part is that it doesn't have to cost you much at all.

### Bill of materials

*   Raspberry Pi, SD card
    *   I would recommend using a Raspberry Pi 2 or 3 because it has more memory available and is better suited to multi-tasking.
    *   (Note: if you're trying to install multiple receivers and save money the Pi Zero will also work well.)
*   USB TV Tuner - _starting at around 8USD up to 30USD_
    1.  [FlightAware Pro Stick from ModMyPi](https://www.modmypi.com/raspberry-pi/breakout-boards/flightaware/flightaware-pro-stick-plus-usb-sdr-ads-b-receiver/?search=flight%20stick) - a premium option with built-in noise filter, SMA connector - purchase your 1090 MHz antenna separately.
    2.  [Generic DVB-T USB tuner from Pimoroni](https://shop.pimoroni.com/products/dvb-t-dongle-ideal-for-ads-b-real-time-plane-tracking) - _I was sent a sample for testing - it works well and has the correct chipset, but push the telescopic antenna into the smallest position._
    3.  [NESDR Smart Premium SDR](http://www.ebay.co.uk/itm/NESDR-SMArt-Premium-RTL-SDR-Set-w-0-5PPM-TCXO-Aluminum-SMA-R820T2-RTL2832U/152142033580?ssPageName=STRK%3AMEBIDX%3AIT) - comes with three antennas, thick SMA cable and a built-in noise filter for urban environments.
    4.  If you're looking to save money there are much cheaper tuners on eBay but they must have the _R820T chipset_ to work.

> Note: For the best results please buy one of the branded DVB-T sticks (these are not affiliate links).

*   Antennas

Most DVB-T tuners come with antennas that are designed to pick up terrestrial TV rather than ADS-B signals which resonate at 1090 MHz. You should still be able to see some flights though because the primary factors determining range are height and line of sight to the sky.

*   If you need a 1090 MHz antenna you can purchase [an indoor 3 dBi version here](https://www.modmypi.com/electronics/antenna/antennas/3dbi-ads-b-1090mhz-sma-antenna-w-magnetic-base/?search=antenna) with an SMA fitting. I tested one out and was able to pick up flights 150 miles away when placed on a windowledge.
*   Even in a built-up area I was able to pick up flights over 250 miles away with the [FlightAware antenna](https://www.modmypi.com/electronics/antenna/antennas/flightaware-1090mhz-ads-b-antenna-66cm--26in/?search=antenna) hanging out my window. It is rated for even better range when mounted to a roof.

Power

*   You'll also need a 2.5A-3A power supply _to make sure we have enough power available for the Pi and the TV tuner_. You can pick up one of the [official Pi power supplies](https://shop.pimoroni.com/products/5v-3a-barrel-jack-power-supply?utm_medium=cpc&utm_source=googlepla&variant=32491020362&gclid=Cj0KEQjwkZfLBRCzg-69tJy84N8BEiQAffAwqsLpydpxI6_vwflBxeoyjJEdJAw_ooGl-56ZN6Vv5QEaAntt8P8HAQ) from Pimoroni.

> ModMyPi also provide [a Raspberry Pi 3 kit](https://www.modmypi.com/raspberry-pi/set-up-kits/rpi3-model-b-kits/piaware-aircraft-tracking-kit-inc.-raspberry-pi-3) with everything you need to get started.

### Glossary

We will repurpose a USB TV tuner to pick up ADS-B broadcasts from aircraft within range. Let's start with some definitions and explanations of flight tracking jargon.

*   ADS-B

Modern aircraft have automatic transponders on board which gather info from navigational instruments and broadcast it to the surrounding area using ADS-B. It's not encrypted so anyone can pick it up whether you're a flight controller, another plane or even a Raspberry Pi owner.

> Automatic dependent surveillance – broadcast (ADS–B) is a surveillance technology in which an aircraft determines its position via satellite navigation and periodically broadcasts it, enabling it to be tracked -- [Wikipedia](https://en.wikipedia.org/wiki/Automatic_dependent_surveillance_–_broadcast)

*   DVB-T

The TV tuner we need is called a _DVB-T_ and stands for - _Digital Video Broadcasting — Terrestrial_. You can also use these devices for tuning into your favourite TV shows. Not all DVB-T devices are able to be repurposed - so make sure you pick one of the recommended devices or do some research before purchase.

*   SMA antenna connector

SMA ([SubMiniature version A](https://en.wikipedia.org/wiki/SMA_connector)) connectors are smaller than coaxial ones and normally come on premium or purpose-built DVB-Ts. If you bought a cheap DVB-T it will probably have a smaller connector - a "Pigtail" can be bought on eBay or from an electronics store to convert between any of the major antenna connectors - coaxial, SMA or RF.

![SMA connector from ModMyPi](https://www.modmypi.com/image/cache/data/rpi-products/breakout-boards/flightaware/DSC_0955-800x609.jpg)

*   dump1090

![](https://blog.alexellis.io/content/images/2017/07/1090.jpg)

A core component of decoding ADS-B signals is the 1090 software component - _1090_ refers to the frequency we're dealing with and _dump_ is the job it performs - decoding and dumping out raw data.

The dump1090 application is an open-source project which has been forked several times by different people who have each added their own tweaks and enhancements. It can be confusing trying to pick the right fork to build and test.

This is the history I could work out from GitHub:

*   [antirez](https://github.com/antirez/dump1090) started the project in 2012 over a Christmas vacation
*   [MalcolmRobb](https://github.com/MalcolmRobb/dump1090) took over the project - forking the code and making many enhancements
*   [mutability](https://github.com/mutability/dump1090) then forked the work of MalcomRobb
*   [FlightAware](https://github.com/flightaware/dump1090) also maintain a fork of the mutability repository.

We will use Docker to build the code, but you can just as easily run the commands separately in a terminal. Some pros of using Docker containers:

*   Repeatable build-script
*   Gives a reliable build mechanism vs. struggling with README pages
*   Lets us switch between different versions of the code
*   Nothing is installed on our Pi directly, so it's well-contained
*   Can share the built image with friends, or with other Pis

Most versions of dump1090 also include a web interface that lets you view the aircraft within range in real-time.

*   FlightAware

FlightAware is one of several websites that will aggregate data picked up from your dump1090 program. You can pick up detailed statistics on which flights you have helped track and what your range is like via a virtual RADAR visualization.

Here are the results from my account on the FlightAware page using their dedicated antenna and noise reducing DVB-T.

![](https://blog.alexellis.io/content/images/2017/07/Screen-Shot-2017-07-16-at-19.12.08.png)

*   MLAT

MLAT stands for Multilateration - a technique where multiple ground-stations (such as our Raspberry Pi) can be used to track aircraft which are not transmitting ADS-B data. You can [read more here](http://flightaware.com/adsb/mlat/).

It relies on very tight timing tolerances - it should work out of the box and then allow you to track many more flights than you would see with ADS-B alone.

### Software installation

*   Install Docker

We'll use a Docker image so that we can get a repeatable build of the various components we need and keep any additional binaries outside of the host's filesystem.

    $ curl -sSL https://get.docker.com | sh

*   Clone the GitHub repository

    $ git clone https://github.com/alexellis/eyes-in-the-sky

*   Blacklist the USB TV stick

In order for the dump1090 software to access the USB TV stick we have to create a _blacklist_ entry for its kernel module.

Add this line to `/etc/modprobe.d/blacklist.conf`:

    blacklist dvb_usb_rtl28xxu  

Now reboot

#### The decoder - dump1090

![](https://blog.alexellis.io/content/images/2017/07/pim_zero.jpg)

_Pictured: testing Pimoroni's DVB-T - connected to a Pi Zero and left at a relative's house._

*   Build the dump1090 image

If you want to enter your own position (latitude and longitude) then edit the last line of Dockerfile.malcolmrobb which starts with `CMD`. You can find your position using Google Maps.

    $ cd eyes-in-the-sky/dump1090
    $ docker build -t alexellis2/dump1090:malcomrobb . -f Dockerfile.malcolmrobb

*   `-t` is how we specify the image name for use later
*   `-f` allows us to pick a Dockerfile with a custom name, I've provided one for mutability's fork too.

With Docker you can share your pre-built images with anyone by using the `docker push` command. This uploads them to the Docker Hub. To pull my dump1090 without building the code from scratch type in:

    $ docker pull alexellis2/dump1090:malcolmrobb

*   Test the dump1090 image

You're now ready to test out the image.

    $ docker rm -f 1090 # remove any old container

    $ docker run --privileged -p 8080:8080 -p 30005:30005 -p 30003:30003 --privileged --name 1090 -d alexellis2/dump1090:malcomrobb

The `docker run` command is responsible for starting our code. You can use `docker rm -f 1090` to stop the code later on or if you reboot the Pi just type in `docker restart 1090`.

*   the `-p` flag is used to tell Docker which ports to expose from the container. You can run two copies of the dump1090 code by changing the port number and name on your Docker container.

*   the `-d` flag puts the container into the background as a daemon, so if you want to see the console output just type in `docker logs --tail 20 -f 1090`

_Log sample:_

![](https://blog.alexellis.io/content/images/2017/07/sample-log.png)

If you know the IP address of your Raspberry Pi - you can now open up the built-in webpage i.e. [http://192.168.0.10:8080/](http://192.168.0.10:8080/)

> To find out your IP address type in `ifconfig`.

Now you can log into your Pi at any point and find out what flights are in your area and how good your range is in the chosen location.

**Advanced tip: running without privileges**

If you don't want to run the container with privileges then you can identify the USB device ID and then swap `--privileged` for `--device=/dev/bus/usb/001/004` for instance.

You should swap the last number i.e. `004` - you can find the correct number by typing in `lsusb`.

    $ lsusb
    Bus 001 Device 004: ID 0bda:2838 Realtek Semiconductor Corp. RTL2838 DVB-T  

#### Installing FlightAware

There are several flight-tracking websites available, but I started out with trying FlightAware. Their software connects to your dump1090 code and streams your data to their online servers where you can collate statistics and compare your range and statistics with others through leaderboards.

You can install FlightAware's .deb file directly on your Pi, but I've created a Dockerfile. This has two advantages - we can run two or more copies of the software and switch between versions without re-flashing our Pi.

Either build the image with the following, or pull the image with `docker pull alexellis2/flightaware:3.5.0`:

    $ cd eyes-in-the-sky/flightaware
    $ docker build -t alexellis2/flightaware:3.5.0 .

> Be careful not to miss the dot at the end of the line

Now sign up on the FlightAware.com website for a username and password.

Edit the piaware.conf file replacing the following fields:

*   receiver-host

(Use the Pi's IP address)

*   flightaware-user
*   flightaware-password

(Use the details you entered online)

FlightAware has a smart feature to track your Raspberry Pi by MAC address - fortunately Docker allows us to spoof the MAC address so we can run multiple copies of the software. If you do this just change the MAC so it's unique for each copy of the software.

Let's now run the image and watch the logs:

    $ cd eyes-in-the-sky/flightaware

    $ docker rm -f piaware_1
    $ docker run --mac-address 02:42:ac:11:00:01 -v `pwd`/piaware.conf:/etc/piaware.conf --name piaware_1 -d alexellis2/piaware:3.5.0

View the logs and hit Control + C at any time:

    $ docker logs --tail 20 -f piaware_1

Your Pi will appear on the website in a few minutes.

### Q&A

*   How much power will this draw?

Your Pi Zero or 2/3 will draw 2-3 watts while running idle. The dump1090 app will use the Pi's processor - up to 50% of it for a Zero, so factor in additional power for the load and the USB DVB-T.

*   Can I run this from a USB battery back?

Yes - for a limited amount of time. The same battery pack that I can use to run a timelapse for 3 days was flat in less than 3 hours while tracking flights.

*   Can I run this from solar power?

Solar power is probably not the right answer - you cannot reliably run a Raspberry Pi directly from a solar panel. You would need sophisticated equipment including a charging controller, adequately sized solar panels and batteries which can store enough charge for several days.

You may be better off running the Pi over Power over Ethernet and a waterproof housing. Here's a [parts-list found online](https://puck.nether.net/~jared/blog/?p=148).

*   Is there an out-the-box solution like an image or ISO?

You can find a complete SD Card image from FlightAware, but if you build a system from modular components you can easily take advantage of any other software designed to work with dump1090.

*   Are you planning on reviewing any equipment in detail?

Keep an eye out for a follow-up blog post looking at the results I was able to achieve with various antenna and tuners.

![](https://blog.alexellis.io/content/images/2017/07/antenna_type.jpg)

_Pictured: antennas on test - cantenna, FlightAware, 2x 1090 MHz_
