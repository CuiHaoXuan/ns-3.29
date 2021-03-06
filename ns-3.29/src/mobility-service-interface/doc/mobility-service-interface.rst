Mobility Service Interface
--------------------------

.. include:: replace.txt
.. highlight:: cpp

.. heading hierarchy:
   ------------- Chapter
   ************* Section (#.#)
   ============= Subsection (#.#.#)
   ############# Paragraph (no number)

Mobility trace generation in |ns3| is done either by using the
mobility module to generate synthetic mobility traces, or by importing
mobility traces generated by third party programs in a compatible format.
Although mobility traces can be generated by third party programs, such
as SUMO, they can involve too much work for their generation,
especially when the researcher's interest - and expertise - is more on
the data communication than on the traffic simulation.
Using the Google Maps API, it is possible to generate complex mobility
traces simply by providing a start and end point.

This module currently creates an |ns3| mobility trace based on the
XML output of the Google Maps Directions API. This API provides its
user with an encoded polyline string, which can be easily transformed
into geographical coordinates. These, in turn, can be transformed into
Cartesian coordinates for use with the |ns3| mobility module. 

Model Description
*****************

The source code for the module lives in the director
``src/mobility-service-interface``.  The module has dependencies on
the following shared libraries:

* libcurlpp

* Xerces-C++

* GeographicLib

The ``ns3::WaypointMobilityModel`` works by adding ``ns3::Waypoints`` to a
dequeue sequentially. The Google Maps Directions service, however, provides
an XML output that contains the following concepts:

* Leg - A Leg is a list of Steps. Legs only occur if the user specifies a Waypoint (A to B, passing through C)

* Step - A Step contains Points encoded in a polyline

* Point - A latitude/longitude set

To create a mobility trace, this module uses a list of Legs,
each one containing a list of Steps and these, in turn, with a list of
Points.

The ``ns3::WaypointMobilityModel`` defines a journey as an ordered (by time)
sequence of ``ns3::Waypoints``. This module defines a journey as a list of
Legs, with each Leg having a list of Steps and each Step with a list of
Points.

Design
======

This module interacts directly with the ``ns3::WaypointMobilityModel``.
The module downloads the information from the
Google Maps APIs (be it the Places or the Directions API), and converts
that information to ``ns::Waypoints``, which can then be used with 
``ns3::WaypointMobilityModel`` to generate mobility based on real-world 
streets, highways, etc.

This module will also calculate at which time the ``ns3::Waypoints`` are
placed, therefore providing variable speed for the nodes.

In order to support different APIs, such as OpenStreetMaps's APIs, the
Strategy design pattern was used. This design will make it easier to add
other sources of information, be it for directions or for real-world
locations. At the moment, only the Google Maps Directions and Places APIs
are implemented.

This module includes a helper class that enables the user to perform every
use case mentioned. It also enables the user to set a number of environment
variables to control the mobility traces generated.

UML Documentation
*****************

The UML diagrams below where created using different software and the
editable files for these diagrams can be found under ``src/mobility-service-interface/doc/uml``.

Use case diagram
================
.. _UCDiagram:

.. figure:: figures/MobilityServiceUC.*
   :align: center
   :scale: 80%
Class diagram
=============

.. _ClassDiagram:

.. figure:: figures/MobilityServiceClassDiagram.*
   :align: center
   :scale: 80%

Sequence diagrams
=================

ChooseRoute(std::string startPoint, std::string endPoint, Ptr<Node> node)
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

.. _ChooseRouteBaseSD:

.. figure:: figures/ChooseRouteBaseSD.*
   :align: center
   :scale: 80%

ChooseRoute(NodeContainer& nodeContainer, double lat, double lng, double radius)
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

.. _ChooseRouteBaseSD:

.. figure:: figures/ChooseRouteAutomaticSD.*
   :align: center
   :scale: 90%

Scope and Limitations
=====================

The purpose of this module is to quickly generate mobility traces with some
degree of realism. This means that it's as easy to use as the |ns3| random
mobility models, while providing the users with some degree of control
over the mobility trace generated. With this module, a user can quickly
define mobility for large node containers without wasting too much time
configuring it.

The mobility traces generated are based on real world streets, highways,
public transit pathways (train lines, subway lines, etc), which can give
some users, a quick and easy way to generate mobility for their research.

This module is not a replacement (and it doesn't aim to be one) for third
party software such as SUMO/TraNS, which take into account more parameters,
producing traces with a higher degree of realism.

At the moment, the module is capable of generating mobility traces between
two real world locations and generating a variable speed for the node.
This means that the node will try to maintain the (real-world) speed limit
for the type of road it's traveling, while decelerating for a curve,
a roundabout, an intersection, etc, and accelerating after those obstacles
are overcome.

In conclusion, this module aims to be a middle ground between the easy to
use random mobility models, which aren't very realistic, and the very
complex methods such as SUMO/TraNS, which require some time to configure,
but are as realistic as possible.

One of the main goals of this module was to allow for an easy way to create
mobility traces for hundreds/thousands of nodes. To achieve this, the
Google Maps Places API was chosen, to provide start and endpoints, which
would then be used with the previously mentioned functionalities.

Unfortunately, due to a limitation of the API, only 60 addresses can be
returned in a single search. These locations are, then, randomly selected
as start and endpoints. The problem with this approach is that, with
numerous node containers, many nodes will have the same start or endpoints.

This approach, while not being ideal in some scenarios, will provide the user
with real world locations that correspond with places of interest in the real
world. This will mean that the nodes will be travelling between places of
interest, therefore providing a more realistic selection of routes.

The team implemented an alternative approach to the automatic generation of 
routes, by generating random start and endpoints in a user specified area.

This approach allows the user to generate mobility in a contained area,
thus providing mostly unique start and endpoints.
The algorithm scales well with large node containers, as can be seen in the 
figure below

.. _MobilityTrace:

.. figure:: figures/MobilityTraceExample.*
   :align: center
   :scale: 80%
The area specified and contain obstacles such as bodies of water or unmapped
roads. If a random coordinate is chosen as start or endpoint that points to
a body of water, for example, the Google Maps API will start (or end) the journey from
the nearest shore.

Usage
*****

The mobility model relies on querying the Google Places API service
via https, so web resources must be available on public URL paths
(either via direct Internet access or via proxy configuration on
the host machine); in other words, ``https://maps.googleapis.com`` must
be reachable.

Creating and using a Google Maps API key
========================================

In order for this module to work, a user must create a valid Google Maps
API key and activate the relevant services (Directions and Places).

This is a straightforward operation, which I'll explain in detail in the
following section.

Creating and activating the API key
++++++++++++++++++++

Currently, to create an API key, a user needs to go to
https://console.developers.google.com and login with his or hers Google
Account. A Google account is needed, and can be created for free if the
user doesn't have one.

Next, the user will need to create a new project and access it
(by clicking on it). The user should be on the Project Panel for the
recently created project.

A section entitled Activate an API should be accessed and the Directions
and Places API should be activated, by clicking the buttons to the right
handside of the respective APIs.

The user should then proceed to the Credentials section, under APIs and
Authentication, and click on Create a new key. Several options should
appear, and the user should select "server key".

Using the API key with the module
+++++++++++++++++++++++++++++++++

The API key can be accessed on the Developer's console
(https://console.developers.google.com). To achieve this, the user must
first select his or her project on the Developer's console and, then, on
the navigation bar choose Credentials, under APIs and Authentication.

The API key listed should be copied to a file under
``src/mobility-service-interface/conf/api-key.txt``, which should contain a
single line with the API key.

Some legal notes on the Google Maps APIs
++++++++++++++++++++++++++++++++++++++++

After an analysis to the Google Maps ToS, a couple of legal questions were
posed. The Google Maps API ToS does not specify if a user can have his own
API key. It also does not allow for caching of Google Maps responses for
more than 30 days.

A Google representative was contacted in order to sort these issues. The
reply was positive on the first issue (the users can each have their own
API key), however, the users cannot cache Google Maps responses for more
than 30 days.

Unfortunately, it's impossible to enforce this restriction on the module.
Because of this, the features allowing for response caching for offline use
were removed from the User API. However, some users might still want to
cache the data and use it within the 30 day time-frame, which is explained
on the Advanced Usage section. The user is responsible for his own actions
and must not keep the cached responses for more than 30 days.

The user must abide by the terms of service of the Google Maps APIs.

Building the module
===================

*To be completed. This module will use the bake tool in the future.*
Currently, the user can patch the ns-3 installation and do a 
``./waf configure``
``./waf build``

This assumes that the libraries are correctly installed and are
identified by waf.

If waf fails to find the libraries, the user should edit the wscript
of the module to the correct paths. This problem will, hopefully, be
solved with bake in the near future.

Examples
========

Three examples located in ``src/mobility-service-interface/examples``
highlight typical usage.

* ``routes-mobility-example.cc``:  This example program queries the Google
  Places API for the route corresponding to several waypoints in Porto,
  Portugal, and creates a NetAnim animation file to illustrate the mobility.

* ``routes-mobility-offline-example.cc``:  This example checks for the
  presence of specified cached XML data corresponding to a particular route.  If
  available, it uses the cached XML. The user must download (using wget or other tools) and specify the path to the XML response. LEGAL NOTICE: The user must not keep the cached response for more than 30 days

* ``routes-mobility-automatic-example.cc``: The module queries the
  Google Maps Places service to find start and endpoints it can use.
  This program takes the node container,
  the latitude and longitude of the center of the search and the search's
  radius to generate mobility.

To illustrate the operation of the models, users may pass the ``--verbose``
flag to the examples.

Helpers
=======

At the moment, only one helper (``routes-mobility-helper.h``) has been
developed for this module, which lives under ``src/mobility-service-interface/helper``.
This helper provides every use case detailed on the use case diagram above.

A very basic use of this helper is to create an object of the helper type::

 RoutesMobilityHelper routes (41.1621429,-8.6218531, 13);

The latitude (41.1621429) and longitude (-8.6218531) parameters passed to
the constructor are used to center the Cartesian plane. The last parameter
refers to the altitude of the location.
GeographicLib (the library responsible for converting the WSG84
coordinates to Cartesian coordinates), requires the user to pass it the
coordinates the library should consider as the center of the Cartesian
plane.

In practice, this serves as a reference for the conversion to Cartesian
coordinates. Because of this, it's possible to specify the center of the
Cartesian plane on a given country and request direction on another
country with valid results.

After the object instantiation, the user should call the
``ChooseRoute(std::string startPoint, std::string longitude, Ptr<Node> node)`` which will create a mobility trace for the specified node. An example
of such a call would be::

 routes.ChooseRoute("Rua S.Tomé, Porto, Portugal", "Praça de Gomes Teixeira, Porto, Portugal", node);

Note: The start and endpoint strings can be anything that Google Maps
(http://www.google.com/maps/) can understand. This means that it can be a
street name, a Place (a restaurant, a coffee shop, etc), a set of
latitude/longitude pairs, etc. For example, the following call to the
ChooseRoute function would work just as well::

 routes.ChooseRoute("Instituto Superior de Engenharia do Porto, Porto, Portugal", "Praça de Gomes Teixeira, Porto, Portugal", node);

Note that the first parameter was altered to include a place
("Instituto Superior de Engenharia do Porto", the institute of Engineering
of the Polythecnic Institute of Oporto).

The idea behind the module, as said previously, was to be able to quickly
generate mobility for several nodes without much effort. In order to
accomplish this, a ChooseRoute method was implemented to take a std::list
of start and endpoint tokens.
For example::

  std::vector<std::string> startEndTokenList;
  startEndTokenList.push_back("Rua S. Tomé, Porto, Portugal");
  startEndTokenList.push_back("Rua Dr. Roberto Frias, Porto, Portugal");
  startEndTokenList.push_back("Travessa Dr. Barros, Porto, Portugal");
  startEndTokenList.push_back("Rua S. Tomé, Porto, Portugal");
  routes.ChooseRoute(startEndTokenList, nodeContainer);

This code creates mobility traces for two nodes, with the following routes:

Node 0 - Travels from Rua S. Tomé, Porto, Portugal to Rua Dr. Roberto
Frias, Porto, Portugal.

Node 1 - Travels from Travessa Dr. Barros, Porto, Portugal to Rua S. Tomé,
Porto, Portugal.

This can be, however, a lot of trouble to configure for a large node
container. To automate this process, the user can call the ChooseRoute
function like this::

 routes.ChooseRoute(nodeContainer, 41.1028429,-8.6729531, 5000);

The module will generate mobility traces for a node container within
a 5000 meter radius of the coordinates.
The 60 locations returned by the Places API will be combined, at random,
as start and endpoints for the nodes.

Another similar function is the
``:ChooseRoute(NodeContainer& nodes, double upperLat,double upperLng,double lowerLat,double lowerLng)``.
This function will generate mobility for a node container, by randomly
generating start and endpoints for each node, inside the user specified
area.

The user must supply the maximum value of latitude and longitude for the area,
as well as the minimum values of latitude and longitude.

Attributes
==========

The RoutesMobilityHelper class contains two variables, ``m_departureTime``
and ``m_travelMode``.

The ``m_travelMode`` variable sets the method of transportation, and can
take the following values: driving, walking,bicycling,transit.
This variable is, by default, set to driving. To change the default setting,
the user should access it by calling the
``SetTransportationMethod (std::string travelMode)``.

The ``m_departureTime`` variable sets the real-world departure time for the
nodes. In Google Maps, the user is able to set the time at which he will
actually depart, and Google Maps will show the relevant information for
that particular time. The module is able to reproduce this feature,
producing different mobility traces based on the time of departure and
method of transportation.

The Google Maps API requests that the ``m_departureTime``  be in
EPOCH (time, in seconds, since January 1st, 1970) format. By default,
this parameter is set to the current system time, however, to set it
manually, the user needs to convert the time and date he wants to use for
departure time to EPOCH before setting the value. In the future, a more
user friendly way to specify the time should be implemented.

Two quick ways to obtain the desired time in EPOCH are:

 * Online converters such as http://www.epochconverter.com/

 * The linux date command.

A couple of examples that might interest users is invoking the date command
as::
 date +%s

This will render the current system time in EPOCH. To obtain the EPOCH time
for a specific date, users can run the date command with the following
parameters::
 date --date='2014-12-31 12:30:30 +0000' +%s


Output
======

This module uses the ``ns3::WaypointMobilityModel`` to generate its mobility
traces, which means that every trace source that is available for the
``ns3::WaypointMobilityModel`` will work.

NetAnim provides a good interface to visualize the mobility trace generated,
but the ASCII mobility traces can also be used.

The ASCII mobility traces provide useful mobility information, such as the
Cartesian coordinates and the vectorial velocity of a given node,
at a given time.

Every class introduced for this module has its own log component, with the
most relevant ones being the RoutesMobilityModel, the GoogleMapsApiConnect
and the GoogleMapsPlacesApiConnect.

Advanced Usage
==============

An advanced user might want to make some changes to the model classes, in
order to accomplish a particular aspect of his/her research. Some features
that might be accessed are the addition of real-world waypoints
(A to B, passing through C and D), chaging the speed of a node and offline
caching of the Google Maps API responses.

Waypoints
+++++++++

Real world waypoints can be placed by changing the relevant parts of the
connection URL on the ``ns3::GoogleMapsApiConnect`` class. The user should
refer to the Google Maps Directions API documentation
(https://developers.google.com/maps/documentation/directions/) in order to
correctly build the URL.

Other features might be available by changing some parameters of the URL.
The user should refer to the Google Maps Directions API to see which
parameters are available and how to request them.

Speed of a node
+++++++++++++++

The speed of a node is given by the time that the Google Maps API
calculates the whole journey will take, given the different conditions
(traffic, road configuration, public transit waiting times, etc).
The module then computes how long a node has travelled between
``ns3::Waypoints`` to calculate the time between two waypoints.

The result is evenly distributed ``ns3::Waypoints`` in time, producing
realistic mobility traces. Nodes will decelerate for a curve, an obstacle,
etc. If the public transit method of transportation is chosen, a node will
walk to the subway station, for example, and wait for the subway to arrive
(it won't immediately embark).

An advanced user might want to tweak the node speed values. To achieve
this, the user will only be able to decrease of increase the speed of
a node in percentage, by changing the
``ns3::GoogleMapsDecoder::SetWaypointTime`` function.

Offline caching of the Google Maps API responses
++++++++++++++++++++++++++++++++++++++++++++++++

Because of legal restrictions, this function is not available from the user
API. The user must be aware that offline caching of data for more than 30
days is in direct violation of the Google Maps API terms of service.

A direct call to the ``ns3::GoogleMapsApiConnect::PerformRequest (std::string startingPoint, std::string endPoint, std::string travelMethod, std::string departureTime, bool doDownload, std::string pathToFile)``
function with the boolean parameter (doDownload) set to true and provided a
valid local path will download the Google Maps API response to the local
filesystem. This feature is only available for the Google Maps
Directions API.

The user is also free to download the file manually, querying the API
directly using wget, for example.

To load the XML response, a user can call the helper function
``ns3::RoutesMobilityHelper::ChooseRoute(Ptr<Node> node, std::string pathToFile)``. A user can also load mobility traces for an entire node container,
by downloading all the XML responses for each node in the container and
place them on given directory. It is then possible to invoke the
``ns3::RoutesMobilityHelper::ChooseRoute (NodeContainer &nodeContainer, std::string dirPath)``
with the path to the node container.

Troubleshooting
===============

The user should ensure that the API key he/she is using is activated for
the APIs required (Directions and Places API). He or she must also make
sure that the quota has not been exhausted.

``NS_FATAL_ERROR`` statements are in place to prevent most problems and
will output the corresponding error. The ``NS_LOG`` statements are also
a very useful way to debug this module.

The ``NS_FATAL_ERROR`` statements also validate the responses from the
Google Maps APIs, terminating the simulation in case the user specified
an invalid API key, or is over his quota for a given API.

Unfortunately, when a problem arises in the APIs response, the error messages
outputted by the module do not offer a detailed explanation of what exactly
went wrong with the query.
To debug API responses, the user should query the API directly, using 
wget, for example. In order to obtain the exact query the simulator used,
``NS_LOG``statements were provided, which print the request URL.
The user can, then, use wget to query the API and analyse the response.

The user should also consult the Google Maps APIs documentation, specifically the 
error codes returned, to obtain more details on what could cause that specific error.

Validation
**********

Only one test was developed for this module, which tests against values
obtained from the Google Maps Polyline Enconder Utility
(https://developers.google.com/maps/documentation/utilities/polylineutility)
, for the latitude and longitude decoder function
(``ns3::GoogleMapsDecoder::ConvertToGeoCoordinates``). The Cartesian
coordinate are obtained from GeographicLib and are also tested against
known working values inputted manually.

The ``ns3::Time`` values of the ``ns3::Waypoints`` are also tested against
known working values.

This, in essence, tests everything involved in the inner workings of the
model. However, more tests should be written in the future, based on
requests and suggestions from the community.

Future development
******************

The team's current goals are the implementation of serialization,
which will allow the user to generate the exact same mobility when
repeating simulations.
Another one of the team's goals is the implementation of a bidirectional
coupling between the data layer of the simulator and the mobility trace.

