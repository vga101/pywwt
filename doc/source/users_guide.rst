User's Guide
------------

.. _WWTClient: api/pywwt.client.html
.. _load: api/pywwt.client.html#pywwt.client.WWTClient.load
.. _new_layer: api/pywwt.client.html#pywwt.client.WWTClient.new_layer
.. _new_layer_group: api/pywwt.client.html#pywwt.client.WWTClient.new_layer_group
.. _get_existing_layer: api/pywwt.client.html#pywwt.client.WWTClient.get_existing_layer
.. _change_mode: api/pywwt.client.html#pywwt.client.WWTClient.change_mode
.. _get_frame_list: api/pywwt.client.html#pywwt.client.WWTClient.get_frame_list
.. _get_layer_list: api/pywwt.client.html#pywwt.client.WWTClient.get_layer_list
.. _get_state: api/pywwt.client.html#pywwt.client.WWTClient.get_state
.. _move_view: api/pywwt.client.html#pywwt.client.WWTClient.move_view
.. _ui_settings: api/pywwt.client.html#pywwt.client.WWTClient.ui_settings

.. _WWTLayer: api/pywwt.layer.html
.. _activate: api/pywwt.layer.html#pywwt.layer.WWTLayer.activate
.. _set_properties: api/pywwt.layer.html#pywwt.layer.WWTLayer.set_properties
.. _set_property: api/pywwt.layer.html#pywwt.layer.WWTLayer.set_property
.. _get_properties: api/pywwt.layer.html#pywwt.layer.WWTLayer.get_properties
.. _get_property: api/pywwt.layer.html#pywwt.layer.WWTLayer.get_property
.. _update: api/pywwt.layer.html#pywwt.layer.WWTLayer.update
.. _delete: api/pywwt.layer.html#pywwt.layer.WWTLayer.delete

.. _convert_xyz_to_spherical: api/pywwt.utils.html#pywwt.utils.convert_xyz_to_spherical
.. _generate_utc_times: api/pywwt.utils.html#pywwt.utils.generate_utc_times
.. _map_array_to_colors: api/pywwt.utils.html#pywwt.utils.map_array_to_colors
.. _write_data_to_csv: api/pywwt.utils.html#pywwt.utils.write_data_to_csv

To use ``pywwt`` from Python, first import it:

.. code-block:: python

    import pywwt
    
Alternatively, you can import the ``pywwt`` classes and functions into the namespace:

.. code-block:: python

    from pywwt.mods import *

In this guide we'll be using the first method in all of the examples, but either works well.

Connecting to a WWT Client
~~~~~~~~~~~~~~~~~~~~~~~~~~

Connecting to a running WWT client on the current host is as simple as creating
a WWTClient_ instance:

.. code-block:: python

    my_wwt = pywwt.WWTClient()

If WWT is running on a separate host, and you have enabled access from
remote hosts, you can connect to it by specifying the hostname or IP address:

.. code-block:: python

    my_wwt = pywwt.WWTClient(host="192.168.1.3")

If the WWT client has not been started on the host you are attempting to connect
to, or if you have not enabled remote access, or if the firewall is blocking port
``5050`` on the host, you may get one of the following error messages:

.. code-block:: none

    WWTException: World Wide Telescope has not been started on this host or is
    otherwise unreachable.

.. code-block:: none

    WWTException: Error - IP Not Authorized by Client

If the version of WWT is not the required version, you will get this error message:

.. code-block:: none

    WWTException: World Wide Telescope is not the required version (>= 2.8)

.. note::

    Some tasks require loading files from disk. These will currently only work if the 
    current WWTClient_ instance and the WWT client itself are running on the same host.

Creating New Layers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The heart of ``pywwt`` (and the LCAPI) is the interaction with WWT's Layer Manager. Layers contain
data in WWT. There are three ways to create new layers from a WWTClient_ instance.

load
++++

You can use the load_ method to generate a new layer with data uploaded from a file:

.. code-block:: python

    my_layer = my_wwt.load("particles.csv", "Sun", "Vulcan")

where the file in this case is a CSV file with values separated by commas or tabs. The
second two arguments are the ``frame`` to load the data into, and the ``name`` for the new layer.
In addition to CSV files, the load_ command shape files (.shp), 3D model files (.3ds), and `WTML
files containing ImageSet references
<http://www.worldwidetelescope.org/Docs/WorldWideTelescopeDataFilesReference.html>`_.

load_ takes a number of keyword arguments, which may be used to customize the data in the layer.
These include options to control the color, the start and end date of the events, and options to
control the fading in and out of data.

.. code-block:: python

    my_layer = my_wwt.load("particles.csv", "Sun", "Vulcan", color="FFFFFFFF",
                           start_date="1/11/2009 12:00 AM", end_date="12/31/2010 5:00 PM",
                           fade_type="In", fade_range=2)

load_ returns a WWTLayer_ instance.

`LCAPI Reference: Load <http://www.worldwidetelescope.org/Developers/?LayerControlAPI#load>`_

new_layer
+++++++++

To create a new layer without loading data from a file, use the new_layer_ method:

.. code-block:: python

    new_layer = my_wwt.new_layer("Sky", "My Star", ["RA","DEC","ALT","color"])

where the first two arguments are the ``frame`` to create the layer and the ``name`` of the
new layer. The last argument is a list of ``fields`` that are the names of the data arrays
that will be loaded into the WWTLayer_ instance using an update_ call. new_layer_ also takes
the same keyword arguments as load_.

`LCAPI Reference: New <http://www.worldwidetelescope.org/Developers/?LayerControlAPI#new>`_

new_layer_group
+++++++++++++++

new_layer_group_ creates a new layer group, which is an organizational aid when using the layer manager. The user will
be able to collapse and expand groups in the Layer Manager, and have groups that are sub-sets of other groups.

.. code-block:: python

    my_wwt.new_layer_group("Sun", "my asteroids")

The first argument is the reference ``frame`` for the group and the second is the ``name`` of the group.

`LCAPI Reference: Group <http://www.worldwidetelescope.org/Developers/?LayerControlAPI#group>`_

get_existing_layer
++++++++++++++++++

Finally, to retrieve an already existing layer as a WWTLayer_ object, call get_existing_layer_:

.. code-block:: python

    minihalo_layer = my_wwt.get_existing_layer("minihalo")

Working With Layers
~~~~~~~~~~~~~~~~~~~

Once a WWTLayer_ object has been created, there are a number of options for setting the parameters
of the layer and working with its data.

update
++++++

update_ adds data to layers, removes data, and changes other aspects of the layer. The ``data`` to
be added is a dict of NumPy arrays or lists:

.. code-block:: python

    data = {}
    data["RA"] = ra_coord
    data["DEC"] = dec_coord
    data["ALT"] = alt_coord
    data["color"] = colors
    my_layer.update(data=data, purge_all=True, no_purge=False, show=True)

Where the keys of the dict must correspond to the names of the ``fields`` specified in the new_layer_
call that created this layer. ``purge_all`` controls whether or not all existing data will be cleared
from the layer. Setting ``no_purge`` to ``True`` will prevent data that has already occurred from being
deleted from the layer, which would happen by default. ``show`` controls whether the layer is shown or hidden.

`LCAPI Reference: Update <http://www.worldwidetelescope.org/Developers/?LayerControlAPI#update>`_

activate
++++++++

The activate_ method highlights the selected layer in the layer manager:

.. code-block:: python

    my_layer.activate()

`LCAPI Reference: Activate <http://www.worldwidetelescope.org/Developers/?LayerControlAPI#activate>`_

There are a number of properties associated with each layer, and there are methods for getting and setting
these properties. There is a
`list of properties <http://www.worldwidetelescope.org/Developers/?LayerControlAPI#Table_of_Properties>`_
for layers at the WWT website.

get_property
++++++++++++

get_property_ returns the value of a property given its ``property_name``:

.. code-block:: python

    prop = my_layer.get_property("CoordinatesType")

`LCAPI Reference: Getprop <http://www.worldwidetelescope.org/Developers/?LayerControlAPI#getprop>`_

get_properties
++++++++++++++

get_properties_ returns all of the properties for a layer in a Python dict:

.. code-block:: python

    prop_dict = my_layer.get_properties()

`LCAPI Reference: Getprops <http://www.worldwidetelescope.org/Developers/?LayerControlAPI#getprops>`_

set_property
++++++++++++

set_property_ sets a property with ``property_name`` to ``property_value``:

.. code-block:: python

    my_layer.set_property("AltUnit","MegaParsecs")

The ``property_name`` and ``property_value`` must both be strings.

`LCAPI Reference: Setprop <http://www.worldwidetelescope.org/Developers/?LayerControlAPI#setprop>`_

set_properties
++++++++++++++

set_properties_ sets a number of properties which have been organized into a dict of
{``property_name``,``property_value``} pairs:

.. code-block:: python

    props_dict = {"CoordinatesType":"Spherical",
                  "MarkerScale":"Screen",
                  "PointScaleType":"Constant",
                  "ScaleFactor":"16",
                  "ShowFarSide":"True",
                  "TimeSeries":"False",
                  "AltUnit":"MegaParsecs",
                  "RaUnits":"Degrees"}
    my_layer.set_properties(props_dict)

Each name and value must be a string.

`LCAPI Reference: Setprops <http://www.worldwidetelescope.org/Developers/?LayerControlAPI#Setprops>`_

delete
++++++

delete_ deletes the layer from the Layer Manager:

.. code-block:: python

    my_layer.delete()

If you try to call a method on the associated layer, you will get an error message:

.. code-block:: none

    WWTException: This layer has been deleted!

`LCAPI Reference: Delete <http://www.worldwidetelescope.org/Developers/?LayerControlAPI#delete>`_

Other Commands
~~~~~~~~~~~~~~

There are several remaining methods for WWTClient_ that may be used to control the appearance of the WWT client
and the layers.

change_mode
+++++++++++

change_mode_ changes the view to one of: Earth, Planet, Sky, Panorama, SolarSystem.

.. code-block:: python

    my_wwt.change_mode("SolarSystem")

`LCAPI Reference: Mode <http://www.worldwidetelescope.org/Developers/?LayerControlAPI#mode>`_

get_frame_list
++++++++++++++

get_frame_list_ returns a dictionary of the WWT client's reference frames:

.. code-block:: python

    frame_list = my_wwt.get_frame_list()

returns something like:

.. code-block:: python

    {'Adrastea': {'Enabled': 'True'},
     'Aegir': {'Enabled': 'True'},
     'Aitne': {'Enabled': 'True'},
     'Albiorix': {'Enabled': 'True'},
     ...
     'Umbriel': {'Enabled': 'True'},
     'Uranus': {'Enabled': 'True'},
     'Venus': {'Enabled': 'True'},
     'Ymir': {'Enabled': 'True'}}

`LCAPI Reference: LayerList <http://www.worldwidetelescope.org/Developers/?LayerControlAPI#layerlist>`_

get_layer_list
++++++++++++++

get_layer_list_ returns a dictionary of the WWT client's layers:

.. code-block:: python

    layer_list = my_wwt.get_layer_list()

returns something like:

.. code-block:: python

    {'2D Sky': {'Enabled': 'True',
      'ID': 'fffe96fc-b485-44bb-8f78-538e0f2348d4',
      'Type': 'SkyOverlays',
      'Version': '3'},
     '3d Solar System': {'Enabled': 'True',
      'ID': 'cb87eaec-534d-4490-b3d9-4d9013574895',
      'Type': 'SkyOverlays',
      'Version': '3'},
     'ISS Model  (Toshiyuki Takahei)': {'Enabled': 'False',
      'ID': '00000001-0002-0003-0405-060708090a0b',
      'Type': 'ISSLayer',
      'Version': '2'},
     'Overlays': {'Enabled': 'True',
      'ID': '531f48c6-f8f5-44db-bce5-b81301a25b60',
      'Type': 'SkyOverlays',
      'Version': '2'}}

`LCAPI Reference: LayerList <http://www.worldwidetelescope.org/Developers/?LayerControlAPI#layerlist>`_

get_state
+++++++++

get_state_ returns a dict of some of the details of the current view.

.. code-block:: python

    my_wwt.get_state()

returns something along the lines of:

.. code-block:: python

   {'ReferenceFrame': 'Sun',
    'ViewToken': 'GK484GJ28CH2E59766142GGGGIC8427AA1468BBD2D453FB0A22FA365486C3F21FB521FD2E8683FGGG',
    'ZoomText': '1.2 Mpc',
    'angle': '0',
    'lat': '48',
    'lng': '-12',
    'lookat': 'SolarSystem',
    'rotation': '0',
    'time': '4/1/2015 2:38:13 PM',
    'timerate': '1',
    'zoom': '600000000000'}

`LCAPI Reference: State <http://www.worldwidetelescope.org/Developers/?LayerControlAPI#state>`_

move_view
+++++++++

move_view_ changes the view depending on the supplied parameter:

.. code-block:: python

    my_wwt.move_view("ZoomIn")

where the parameter may be one of:

- ``"ZoomIn"``: Zoom in on the current view.
- ``"ZoomOut"``: Zoom out of the current view.
- ``"Up"``: Move the current view up.
- ``"Down"``: Move the current view down.
- ``"Left"``: Move the current view left.
- ``"Right"``: Move the current view right.
- ``"Clockwise"``: Rotate the view clockwise 0.2 of one radian.
- ``"CounterClockwise"``: Rotate the view counterclockwise 0.2 of one radian.
- ``"TiltUp"``: Angle the view up 0.2 of one radian.
- ``"TiltDown"``: Angle the view down 0.2 of one radian.
- ``"Finder"``: Currently unimplemented.

`LCAPI Reference: Move <http://www.worldwidetelescope.org/Developers/?LayerControlAPI#move>`_

ui_settings
+++++++++++

ui_settings_ changes user interface settings without altering the layer data:

.. code-block:: python

    my_wwt.ui_settings("ShowConstellationBoundries", "True")

To see the list of possible settings see the
`LCAPI section on uisettings <http://www.worldwidetelescope.org/Developers/?LayerControlAPI#uisettings>`_.

Standard Keyword Arguments
~~~~~~~~~~~~~~~~~~~~~~~~~~

Many of the ``pywwt`` methods take a standard set of keyword arguments that may be applied
along with that method's particular arguments.

- ``date_time`` (string): Sets the viewing clock to the given date and time, in UTC format, for example: "1/1/2000 12:02:46 AM"
- ``time_rate`` (float):	The accelerated time to render the visualization, as a multiple of 10.
- ``fly_to`` (list of floats): Sets the position of the view camera. Requires five floating point numbers, in this order:

 1. Latitude is in decimal degrees, positive to the North.
 2. Longitude is in decimal degrees, positive to the East.
 3. Zoom level varies from 360 (the most distant view) to 0.00023 (the closest view).
 4. Rotation is in radians, positive moves the camera to the left.
 5. Angle is in radians, positive moves the camera forward.
 6. (optional) The name of the frame to change the view to.

- ``instant`` (boolean): Used with the ``fly_to`` parameter, set this to ``True`` to specify that the camera should jump to the location, or ``False`` that the camera should smoothly pan and zoom to the location. Default
- ``autoloop`` (boolean): True sets the layer manager to auto loop.

The following methods take these keyword arguments:

- WWTClient:
    + change_mode_
    + load_
    + move_view_
    + new_layer_
    + new_layer_group_
    + ui_settings_

- WWTLayer:
    + activate_
    + set_properties_
    + set_property_
    + update_

An example call:

.. code-block:: python

    my_wwt.move_view("Clockwise", date_time="1/1/2000", time_rate=100.)

which would rotate the view clockwise, set the current date and time to 1/1/2000 at 12:00:00 AM UTC, and increase the
rate of the passage of time by a factor of 100.

`LCAPI Reference: General Parameters <http://www.worldwidetelescope.org/Developers/?LayerControlAPI#general_parameters>`_

Data Utilities
~~~~~~~~~~~~~~

``pywwt`` provides general utilities for generating and transforming data into formats suitable
for WWT.

convert_xyz_to_spherical
++++++++++++++++++++++++

convert_xyz_to_spherical_ takes a set of Cartesian coordinates and returns a dictionary of NumPy arrays
containing the coordinates converted to spherical coordinates:

.. code-block:: python

    sp_crd = pywwt.convert_xyz_to_spherical(x, y, z, is_astro=True, ra_units="degrees")

where ``x``, ``y``, and ``z`` are NumPy arrays corresponding to the Cartesian coordinates, assumed to have
an origin at (0,0,0). From this call, ``sp_crd`` will have ``"RA"``, ``"DEC"``, and ``"ALT"`` as fields. If
``is_astro`` is set to ``False``, the fields will be ``"LAT"``, ``"LON"``, and ``"ALT"``. ``ra_units`` controls
whether the ``"RA"`` coordinate will be in degrees or hours.

generate_utc_times
++++++++++++++++++

For data that does not have a time component, generate_utc_times_ will generate a list of times that may be
used by WWT:

.. code-block:: python

    num_steps = 100
    step_size = {"days":5, "hours":12, "minutes":5}
    start_time = "1/1/2013 12:00 AM"
    my_times = pywwt.generate_utc_times(num_steps, step_size, start_time=start_time)

The first two arguments, ``num_steps`` and ``step_size``, set the number of times and the step between the times.
``start_time`` is a keyword argument that defaults to the current system time if it is not specified. ``my_times``
will be a list of time strings.

map_array_to_colors
+++++++++++++++++++

map_array_to_colors_ takes a NumPy array of floats, and a Matplotlib colormap, and converts the floating-point
values to colors, which may be used as colors for event data in WWT:

.. code-block:: python

    colors = pywwt.map_array_to_colors(temperature, "spectral", scale="log", vmin=1., vmax=7.)

where the first two arguments are the NumPy array ``arr`` to be converted, and a string ``cmap`` representing the
Matplotlib colormap. The ``scale`` of the color map may be set to ``"linear"`` or ``"log"``, and the maximum and minimum
values of the data may be set by ``vmin`` and ``vmax``. If they are not set, they are set to the minimum and maximum
values of the array ``arr`` by default.

write_data_to_csv
+++++++++++++++++

write_data_to_csv_ takes a dict of NumPy arrays or lists of data and writes them to a file in CSV format, which may be
read in by load_:

.. code-block:: python

    particles = {}
    particles["x"] = x
    particles["y"] = y
    particles["z"] = z
    particles["color"] = colors
    pywwt.write_data_to_csv(particles, "my_particles.csv", mode="new")

The keyword argument ``mode`` may be set to ``"new"`` or ``"append"``.