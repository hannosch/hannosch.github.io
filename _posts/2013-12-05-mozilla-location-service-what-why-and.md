---
layout: post
title: Mozilla Location Service - The What, Why and Privacy
date: '2013-12-05T22:13:00.001+01:00'
author: Hanno Schlichting
tags:
- mozilla
modified_time: '2015-11-07T13:23:00.505+01:00'
blogger_orig_url: http://blog.hannosch.eu/2013/12/mozilla-location-service-what-why-and.html
---

I'm currently working on the
[Mozilla Location Service](https://wiki.mozilla.org/CloudServices/Location)
and have been doing so for the last quarters. The project was finally
[announced more broadly](https://blog.mozilla.org/services/2013/10/28/introducing-the-mozilla-location-service/)
over at the Mozilla Cloud Services blog in late October. Since then we've had
a steady amount of interest and a number of repeating questions coming in.
I'd like to take this opportunity to give more background information on why
we are interested in location services and address some questions about the way
we do things. This area is extremely complex, so bear with me while I provide
a lot of background information.

Note that this is my personal opinion and not an official position of Mozilla.

# What is it?

There's a lot of interest in "the location space" and lots of news, start-ups
and competition around location, mobile location, location context, maps and
all things related. We aren't investing broadly in "all things location", but
are focused on one very concrete aspect: The ability of a device to determine
its own position as represented by a latitude and longitude. A device will use
a variety of sensors to determine information about its environment and do
some calculation to derive a position from those. In most cases a remote
service will be queried to provide extra data to aid in the calculation or
large parts of the calculation will be done remotely.

As a concrete example a mobile phone might be connected to a cell tower. Using
this information it will query a remote service. The remote service has a
database of all cell towers mapped to positions and can return the position of
the cell tower as an approximation of the device position.

The sensors typically used for this purpose include a satellite based
positioning sensor (GPS, Glonass, ...) and data for visible cell towers, WiFi
access points and Bluetooth LE beacons. Other sensors like the compass or
accelerometer can be used to determine relative position changes. There are
even scientific experiments to use light sensors and variations in the
magnetic field as sources of location information. If multiple sensors deliver
data, these can be combined to provide more precise positions or positions
with a smaller error margin or better accuracy. For some sensors additional
data like signal strengths, signal noise or time of flight data can be used to
narrow down the position.

# What it is not!

The Mozilla Location Service only deals with determining a device position. It
doesn't and won't deal with services like maps, points of interest data,
transportation or traffic information. For some of these
[OpenStreetMap](https://www.openstreetmap.org/) is an established open
community and it doesn't make sense for us to compete with them. In addition
there are also a good number of commercial offerings for these higher level
services and both users and developers have a good number of choices here.
Different Mozilla products might partner with both communities and companies
to provide user value and bundle their services - those questions are outside
of our scope here.

Our location service helps with what you might call device initiated
positioning. It is up to the device and ultimately the user to decide if a
positioning request should be done. Separate from this are network or operator
based positioning methods used for mobile phones. As part of being connected
to a cell network, the cell network operator maintains a database of which
phone is currently connected to which cell tower. This is required to route
incoming phone calls or SMS to the right tower and ultimately the phone
itself. Depending on the cell standard a phone might maintain connections to
multiple cell towers. You can find out more about this on the
[radiolocation wikipedia](https://en.wikipedia.org/wiki/Radiolocation)
page.

These operators based methods don't require additional user consent and work
as a side-effect of having a cell connection. The only way to opt-out of these
is by not using phones. In many countries operators are required to provide
this data to government agencies. There are two common examples of this. One
is emergency services (for example
[E-911](https://en.wikipedia.org/wiki/Enhanced_9-1-1)
) in which the emergency call is directed to the geographically closest
response center and the response center gets the exact location of the caller
to quickly dispatch emergency help to the right location. Even if the caller
is unsure of his or her location or confused or distressed.

The other much more controversial case is
[data retention](https://en.wikipedia.org/wiki/Telecommunications_data_retention)
in which operators have to store and share metadata about calls and phone
positions for a lengthy period of months to years. There are also examples
of intelligence services capturing this information on sometimes questionable
legal grounds. These operator based methods are completely outside the
control and scope of our location service.

# Why do it at all?

There are plenty of examples where users are willing to have their devices
determine their location. There always has to be and will be an explicit user
choice in this, but most people find some of the use-cases compelling enough
to allow this.

Some of the use-cases are obvious and others not so much. One of the earliest
examples is being able to pin- point the current location on a map, for
example to show points of interest like coffee shops around the current
location, or show the transit data for the nearest bus stop. Another popular
example is continuous location updates for driving directions or capturing
walking or bike trails as part of fitness apps. In the future this will be
extended to indoor location and for example being able to get walking
directions to a specific shop inside a large shopping center, including hints
to take escalators down or up various levels.

Another not quite so obvious use-case is taking photos and automatically
recording the position in the photos metadata. Or a service to "find my
device" - either when a user lost it or fears it has been stolen. Maybe a user
is also willing to share his or her location with a trusted group of people,
for example as a parent I might want to be able to tell the location of my
kids phones. Or whenever I'm near a friend of mine, I'd like to be alerted so
we don't miss each other. These cases certainly aren't for everyone and
require strong and explicit user consent or consensus in the family.

Even more recent examples are what's technically called
["geo-fencing"](https://en.wikipedia.org/wiki/Geo-fence)
and the use of Bluetooth low energy beacons. These allow the user or apps
on the users behalf to take some action when the user leaves or arrives at a
certain area. A common example is being reminded to call someone or a reminder
to go shopping once you leave work. If the user opts-in this might also be
used to show interesting offers or ads when the user is near a favorite coffee
shop or next to a specific section inside a store, or interact with a museum
exhibit once the user gets close to it. A largely unexplored field is various
sorts of augmented reality games, where players have to move around in the
real world, but the real world is overlaid with game mechanics.

In addition to those user level use-cases there's also at least one technical
use-case most people are unaware of. Using a satellite based sensor like the
GPS completely on its own provides a rather poor experience, as the chip needs
up to 10 minutes to provide the first location. If you owned an older
standalone GPS navigation system, you might remember the long waiting times on
first use. In order to improve this, all modern sensors use some form of
[assisted GPS](https://en.wikipedia.org/wiki/Assisted_GPS),
in which other sensors are used to tell the GPS in which general area it is
and based on that what satellites it should see. It also uses a faster data
link to download information about the exact orbit of satellites. Currently
this support is outside the scope of our location service and provided by
partners via the
[mobile "secure user plane location" standard](https://openmobilealliance.org/about-oma/work-program/location/).

Currently we also don't deal with IP address based location or Geo-IP, which
is commonly used to either direct website visitors to a regional part of the
website or restrict content if distribution rights aren't available for all
countries.

# Why Mozilla?

As the use-cases in the previous section should have shown there is tremendous
user interest in this. As a result any platform needs to offer the capability
to position users. For us this means we need to support these use-cases for
the web platform, independent of whether you are running Firefox on top of
another operating system (Windows, Linux, Mac OS, Android, etc.) or directly
(Firefox OS). In terms of official developer API's the
[W3C Geolocation](https://www.w3.org/2008/geolocation/)
working group has defined a
[geolocation API](https://developer.mozilla.org/en-US/docs/WebAPI/Using_geolocation)
and is being re-chartered to add new API's for geo-fencing and indoor location.
These are developer visible API's, but they don't define how the underlying
browser gets the required data or interacts with device sensors or an
operating system.

Furthermore it should have become clear that positioning needs an external
service component, even for seemingly standalone sensors like GPS. We
currently have to either use the underlying operating system services or use
commercial partners to provide these service components. There is little
competition in the market and we aren't in a good position to influence terms
of service on our users behalf, especially in the area of privacy. In addition
cost of entry into this specific market is extremely high, as collecting a
global data set and keeping it up-to-date requires a lot of resources. As a
result there are few commercial companies trying to gain market entry, service
costs are high and no open community project emerged that covers the whole
world and combines all of the required data types into one consistent
experience. We maintain a
[list of some of the projects](https://wiki.mozilla.org/CloudServices/Location/Bootstrap)
we know about and most are practically constrained to certain regions or
data types like only cell or only WiFi data.

When we started out to investigate the problem space, we were aiming to create
a public data set and not just a public service. Unfortunately over time we
discovered more and more legal and privacy problems and we haven't found a
solution for those yet.

# Privacy

Privacy is a serious concern while dealing with location data. As part of our
location service we have to deal with the privacy concerns of three different
actors. The first is the normal end-user of the service, who uses it to
determine his or her position. The second is any user who chooses to
contribute back data to the service and improve the underlying database for
the benefit of all users. The third is the owner of the equipment used as an
information source, like cell towers, WiFi access points or Bluetooth beacons.

For another much shorter take on privacy, you can read
[Gerv's much shorter blog post](http://blog.gerv.net/2013/10/location-services-and-privacy/)
from late October.

## Privacy - WiFi interlude

Especially for owners of WiFi access points we have some interesting
challenges. The service uses a unique identifier for each WiFi access point in
the form of the
[BSSID](https://en.wikipedia.org/wiki/Service_set_%28802.11_network%29).
This is a technical identifier which is most often the
[MAC address](https://en.wikipedia.org/wiki/MAC_address)
of the underlying network interface, but can also be a randomized number for
ad-hoc networks. In addition to this technical id WiFi networks also have a
[human readable name (SSID)](https://en.wikipedia.org/wiki/SSID).
You see the name whenever choosing what WiFi to connect to.

The BSSID is a unique device id and never changes. What's worse is that many
modern smart phones not only connect to WiFi networks, but also act as WiFi
access points of their own, to share their internet connection with your other
devices like a laptop or tablet. Using WiFi access points as part of a
location service isn't ideal, both for these privacy concerns and due to
technical limitations. The existing protocols where never meant for this use-
case and thus have many shortcomings. For instance there is no way to
distinguish a phone from a stationary access point, while observing it only
once. But WiFi access points are the only wildly deployed type of device that
can be used and provides a precise enough position for many of the current
use-cases. Changes to wireless networking standards take years to agree upon
and many more years to be available in a majority of deployed devices. So
workarounds based on current standards are the only viable mid-term option.

# Privacy continued

If you haven't followed tech news closely over the years, you might have
missed a couple of the early problems and revelations about location services
by other companies. Starting in 2010 a variety of news stories, governmental
and data privacy agency actions took place. It's valuable to look a bit more
closely at the history here and learn from these lessons.

Two good overviews of all these come from CNET in a roundup
[article from July 14, 2011](http://news.cnet.com/8301-13579_3-20057175-37/geotracking-controversy-homes-in-on-iphone-roundup/)
and another
[article from September 1, 2011](http://news.cnet.com/8301-1009_3-20086623-83/how-companies-use-wi-fi-to-track-you-roundup/).
There where four interesting cases.

## Apple and the large cache

Researchers found that iOS devices stored a year long detailed list of all the
places the device had been in a local cache file. The concerns where addressed
in the end and
[Apple provided a detailed Q&A on its location data practices](http://www.apple.com/pr/library/2011/04/27Apple-Q-A-on-Location-Data.html).
At the end of the article three concrete actions are detailed:

1. Only maintain a location cache for a short period of time. The cache is
useful to improve the experience and allow device local look-ups for frequently
visited locations like your home or workplace. Android was found to have a
similar cache with a much shorter time period.
2. Don't backup this cache to any external device or cloud service.
3. Delete the cache if the user opts-out of using location services
(and never maintain it without user consent in the first place).

## Google, Microsoft and the single WiFi look-up

In
[June 2011 a CNET article](http://news.cnet.com/8301-31921_3-20070742-281/exclusive-googles-web- mapping-can-track-your-phone/)
appeared describing how researchers probed Google's location service and where
able to lookup smart phones or personal WiFi routers over time and track them.
Later in
[August 2011 a similar CNET article covered Microsoft](http://news.cnet.com/8301-31921_3-20086489-281/microsoft-curbs-wi-fi-location-database/)
and an
[article in Ars Technica specifically covered Microsoft's](http://arstechnica.com/information-technology/2011/08/microsoft-locks-down-wi-fi-location-service-after-privacy-concerns/)
response. The immediate result were two new restrictions to the services. On
one hand greater care was taken to filter out constantly moving WiFi access
points, as these were likely smart phones and useless for determining a users
position. On the other hand a "you need to know two" restriction was added.
The idea here is to only answer service requests, if you can provide
information about two nearby WiFi networks. The assumption is that you can
only reasonably get two matching data points if you are indeed near those WiFi
access points. This should make it considerably harder to do remote tracking
of anyone. If you already know where someone is, potentially confirming this
result via the service is much less of a problem than tracking the user in the
first place.

## Google street view cars, wiretapping and opt-out

As part of building up a database of street view imagery, Google also
collected additional data while driving around. There's an
[original Google statement from April 2010](http://googlepolicyeurope.blogspot.de/2010/04/data-collected-by-google-cars.html)
and an
[update from May 2010](http://googleblog.blogspot.de/2010/05/wifi-data-collection-update.html)
in which Google admitted to collecting not only metadata about WiFi access
points like everyone else, but also samples of the network traffic of
unencrypted WiFi networks.

There's so far been no regulation or statements which would disallow the
metadata collection, but the collection of network traffic has seen various
legal actions. In the US the court cases over wiretapping charges are still
going on, with some details in a recent
[Wired article from September 2013](http://www.wired.com/threatlevel/2013/09/googles-wifi-wiretapping/).
In the European Union cases by various data protection agencies have already
been resolved, as for example confirmed by a
[report by the Dutch DPA from 2012](http://www.dutchdpa.nl/Pages/en_pb_20120405_google-complies-with-Dutch-DPA-requirements.aspx).

While the wiretapping charges are unrelated to our location service, the
initial concerns have led to the creation of an opt-out approach for owners of
WiFi access points. A
[Google blog post from November 2011](http://googlepolicyeurope.blogspot.de/2011/11/greater-choice-for- wireless-access.html)
explains their "_nomap" suffix approach. This approach lets any WiFi access
point owner signal his intent to avoid tracking. This is done by changing the
user visible WiFi name (SSID) and append a "_nomap" string. Changing a user
visible name isn't ideal, but unfortunately it's the only user changeable
setting in WiFi standards. This approach is similar to efforts like
"do not track", which rely on a combination of a user signal and the industry
respecting this signal.

Unfortunately there's so far no industry standard for WiFi tracking. A manual
[opt-out approach was chosen by Microsoft](https://www.windowsphone.com/en-us/support/location-block-list)
instead. Other competitors have so far not offered any opt-out approach at all.

For our own location service we have chosen to respect the "_nomap" signal. It
has the advantage of only requiring a single user action, instead of each user
having to track down all possible location services and dealing with lots of
companies. Not to mention having to keep track of any new company entering the
field. In addition we are still
[discussing if and how to implement the manual opt-out approach](https://bugzilla.mozilla.org/show_bug.cgi?id=874916).

## Apple and the randomized BSSID

There is one case that isn't documented very well and only mentioned in
various support forums. With the release of iOS 5 in late 2011 users reported
a variety of problems with WiFi connectivity. Some users suggest that Apple
actually made at least one intended change and switched its mobile hotspot
from using normal infrastructure mode to ad-hoc mode. In infrastructure mode
the WiFi is using the underlying MAC address as the BSSID. In ad-hoc mode it
sets two bits of the BSSID to mark it as ad-hoc and than generates a random
bit sequence for the rest. The device can than regenerate a randomized id once
in a while.

If this were true, at first glance it would give a neat solution to the live-
long unique id problem. By using ever changing randomized ids, the tracking
potential for phones would be limited a lot and reduced to a much shorter time
span. Since the BSSID contains two magic bits to mark it as ad-hoc, it would
be easy for any location service to exclude these WiFi networks from its
databases. Since they are only set on phones or otherwise ad-hoc and moving
WiFi networks, it's in the best interest of any location service to filter
them out, since they don't provide any value. Independent of whether this
story is true or not, both the code deployed on Google's street view cars and
our own clients filter out ad- hoc networks.

According to the forum reports, Apple however had to revert this change to
satisfy users. The problem that occurred was that most operating systems
remember WiFi networks not only by the clear text name, but also by their
BSSID. Laptop users who wanted to use their trusted mobile phone hotspot were
suddenly constantly presented with a dialog to re-enter the WiFi password, as
the operating system thought these were new and never seen before devices.
This user frustration apparently trumped a privacy win at the time.

It's unclear how much of this story is true. But it highlights one possible
avenue for improving privacy of mobile phone owners. Either operating systems
could be adjusted to keep remembering WiFi networks independent of their
BSSID. Or there could be an acceptable trade-off, where the BSSID would only
change every couple of days or weeks. Still narrowing down the tracking window
from forever to a much shorter time span.

# Privacy - summary

What most of the historical cases have in common is adding protections which
only work in a service model, where access to the underlying data can be
restricted or data can be removed from the database. At the start of this
project we wanted to make the underlying data source publicly available in as
much detail as possible. But the protections around Wifi access points that
other companies have found and data privacy agencies have agreed upon, don't
work for a public database.

So while we currently don't know of any model to share the WiFi data and
protecting privacy, you might argue that the same problem doesn't apply to
cell data or other types of data. Unfortunately there's a different group of
users whose privacy is a concern here. The users deciding to contribute to the
service should be protected as well. As long as there is just one or very few
users in any geography, those location samples can rather easily be tied back
to individual users, even without Mozilla tagging the data with unique user
ids. And we made sure to never store the location data samples with a user id
in our current system. We keep our optional leader board system completely
separate from the location data.

There's research that convincingly shows that once you add user ids, you only
need a very small number of data samples to uniquely identify people.
Basically most people always travel the same routes and stay at work and home
for long periods of time. Once I know both a home and work address, it's easy
enough to match this data with other data sources. Now while we don't add user
ids, we aren't clear on whether or not any of the data that is sent, is unique
enough to still identify users. This starts with having data about the country
and mobile carrier and the user cell standard. Combine that with potential
device specific patterns in the reported signal strength or other measures,
and there might well be a way to find identifying marks. For instance the
first LTE user in any area would be easy to identify, or a laptop or tablet
user would likely show different signal strength readings. At this stage we
don't know enough to decide this. Once we publish the data there is no way of
going back and the data is available for all future people to analyze.

# Final words

I hope this sheds some more light on why we are so extremely cautious and
don't have any good answers on questions like: "are you going to release the
data" or "what license is the data available under". We set out to improve
user privacy as one of our main goals. It might be that this goal can only be
achieved by keeping the data private and identify a trusted organization or a
trusted group of organizations and companies to watch over this data -
assuming we can find an acceptable set of terms that indeed do protect users.
Other sources of concerns are changing legal requirements, which might get us
into trouble for hosting this data. And there is a real problem of brand and
trust damage that would hinder our mission, if Mozilla is suddenly a company
that allowed someone to track other users and the potential harm that can
result from it.

I could go on about the technical difficulties on extracting on processing
this data or the challenge of creating a global data set. But I'll leave those
for another time.

If you want to correct me on any of the statements in this blog post, please
leave a comment. If you want to engage with us, we'd love to hear from you
[on IRC or our mailing list](https://wiki.mozilla.org/CloudServices/Location#Communication).

Hanno
