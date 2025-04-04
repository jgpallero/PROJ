.. _omerc:

********************************************************************************
Oblique Mercator
********************************************************************************

The Oblique Mercator projection is a cylindrical map projection that closes the
gap between the Mercator and the Transverse Mercator projections.

+---------------------+----------------------------------------------------------+
| **Classification**  | Conformal cylindrical                                    |
+---------------------+----------------------------------------------------------+
| **Available forms** | Forward and inverse, spherical and ellipsoidal           |
+---------------------+----------------------------------------------------------+
| **Defined area**    | Global, but reasonably accurate only within 15 degrees   |
|                     | of the oblique central line                              |
+---------------------+----------------------------------------------------------+
| **Alias**           | omerc                                                    |
+---------------------+----------------------------------------------------------+
| **Domain**          | 2D                                                       |
+---------------------+----------------------------------------------------------+
| **Input type**      | Geodetic coordinates                                     |
+---------------------+----------------------------------------------------------+
| **Output type**     | Projected coordinates                                    |
+---------------------+----------------------------------------------------------+


.. figure:: ./images/omerc.png
   :width: 500 px
   :align: center
   :alt:   Oblique Mercator

   proj-string: ``+proj=omerc +lat_1=45 +lat_2=55``


Figuratively, the cylinder used for developing the Mercator projection touches
the planet along the Equator, while that of the Transverse Mercator touches the
planet along a meridian, i.e. along a line perpendicular to the Equator.

The cylinder for the Oblique Mercator, however, touches the planet along a line
at an arbitrary angle with the Equator. Hence, the Oblique Mercator projection
is useful for mapping areas having their greatest extent along a direction that
is neither north-south, nor east-west.

The Mercator and the Transverse Mercator projections are both limiting forms of
the Oblique Mercator: The Mercator projection is equivalent to an Oblique Mercator
with central line along the Equator, while the Transverse Mercator is equivalent
to an Oblique Mercator with central line along a meridian.

For the sphere, the construction of the Oblique Mercator projection can be
imagined as "tilting the cylinder of a plain Mercator projection",
so the cylinder, instead of touching the equator, touches an arbitrary great circle
on the sphere. The great circle is defined by the tilt angle of the central line,
hence putting land masses along that great circle near the centre of the map,
where the Equator would go in the plain Mercator case.

The ellipsoidal case, developed by Hotine, and refined by Snyder  :cite:`Snyder1987`
is more complex, involving initial steps projecting from the ellipsoid to another
curved surface, the "aposphere", then projection from the aposphere to the skew
uv-plane, before finally rectifying the skew uv-plane onto the map XY plane.

.. note::
    In certain special circumstances of the ellipsoidal form of the Oblique
    Mercator it is possible for two different input coordinates to project to the
    same output coordinate. This can happen when the input coordinates are far away
    from the projection center. A work around is to use the spherical version of the
    projection instead. See an example in the Usage section below.


Usage
########

The tilt angle (azimuth) of the central line can be given in two different ways.
In the first case, the azimuth is given directly, using the option :option:`+alpha`
and defining the centre of projection using the options :option:`+lonc` and
:option:`+lat_0`.
In the second case, the azimuth is given indirectly by specifying two points on
the central line, using the options
:option:`+lat_1`, :option:`+lon_1`, :option:`+lat_2`, and :option:`+lon_2`.

Example: Verify that the Mercator, and Transverse Mercator (on a sphere),
projections are limiting forms of the Oblique Mercator

::

    $ echo 12 55 | proj +proj=merc +ellps=GRS80
    1335833.89 7326837.71

    $ echo 12 55 | proj +proj=omerc +alpha=90 +ellps=GRS80
    1335833.89 7326837.71

    $ echo 12 55 | proj +proj=omerc +alpha=0 +R=6400000
    766869.97 6209742.96

    # Same, with azimuth given indirectly via two points:
    $ echo 12 55 | proj +proj=omerc +lon_1=0 +lat_1=-1 +lon_2=0 +lat_2=0 +R=6400000
    766869.97 6209742.96

    $ echo 12 55 | proj +proj=tmerc +R=6400000
    766869.97 6209742.96

Example: Second case - indirectly given azimuth

::

    $ echo 12 55 | proj +proj=omerc +lon_1=-1 +lat_1=1 +lon_2=0 +lat_2=0 +ellps=GRS80
    349567.57 6839490.50

    # Same, with directly given azimuth, (via: echo 0 0 1 -1|geod -I -f %.7f +ellps=GRS80):
    $ echo 12 55 | proj +proj=omerc +alpha=-45.1880402 +ellps=GRS80
    349567.57 6839490.50


Example: An approximation of the Danish "System 34" from :cite:`Rittri2012`

::

    $ echo 10.536498003 56.229892362 | cs2cs +proj=longlat +ellps=GRS80 +to +proj=omerc +axis=wnu +lonc=9.46 +lat_0=56.13333333 +x_0=-266906.229 +y_0=189617.957 +k=0.9999537 +alpha=-0.76324 +gamma=0 +ellps=GRS80
    200000.13   199999.89

The input coordinate represents the System 34 datum point "Agri Bavnehoj", with coordinates
(200000, 200000) by definition. So at the datum point, the approximation is off by about 17 cm.
This use case represents a datum shift from a cylinder projection on an old, slightly
misaligned datum, to a similar projection on a modern datum.

Example: Non-unique results of the ellipsoidal form

::

    proj +proj=omerc +lat_0=0.377041113875403 +lonc=33.8250934444081 +alpha=8.16321575614333 +gamma=0 +k=1 +x_0=0 +y_0=0 +ellps=WGS84 << EOF
    -146.23 -78.19112222222222
    -145.02309217 -78.19112222
    EOF
    875464.92       -11348616.86
    875464.92       -11348616.86

Here we see that two coordinates with roughly a difference in latitude of about
a degree results in the same projected coordinate. As mention above, a work around
is to use the spherical form instead:

::

    proj +proj=omerc +lat_0=0.377041113875403 +lonc=33.8250934444081 +alpha=8.16321575614333 +gamma=0 +k=1 +x_0=0 +y_0=0 +ellps=sphere << EOF
    -146.23 -78.19112222222222
    -145.02309217 -78.19112222
    EOF
    891266.38       -11376037.53
    863563.62       -11374939.17



Parameters
################################################################################


Central point and azimuth method
--------------------------------------------------------------------------------

.. option:: +alpha=<value>

    Azimuth of centerline clockwise from north at the center point of the line.
    If :option:`+gamma` is not given then :option:`+alpha` determines the value of
    :option:`+gamma`.

.. option:: +gamma=<value>

    Azimuth of centerline clockwise from north of the rectified
    bearing of centre line. If :option:`+alpha` is not given, then
    :option:`+gamma` is used to determine :option:`+alpha`.

    If specifying only :option:`+gamma` without :option:`+alpha`, the maximum
    value of the absolute value of :option:`+gamma` is a function of the
    absolute value of :option:`+lat_0`, equal to :math:`90° - |\phi_0|` on a
    sphere and slightly above on a non-spherical ellipsoid.

.. option:: +lonc=<value>

    Longitude of the projection centre. Note that this value is used to
    override the :option:`+lon_0` parameter, so the latter should not be
    specified as it would get ignored.

.. option:: +lat_0=<value>

    Latitude of the projection centre.

Two point method
--------------------------------------------------------------------------------

.. option:: +lon_1=<value>

    Longitude of first point.

.. option:: +lat_1=<value>

    Latitude of first point.

.. option:: +lon_2=<value>

    Longitude of second point.

.. option:: +lat_2=<value>

    Latitude of second point.

Optional
--------------------------------------------------------------------------------

.. option:: +no_rot

    No rectification (not "no rotation" as one may well assume).
    Do not take the last step from the skew uv-plane to the map
    XY plane.

    .. note:: This option is probably only marginally useful, but remains for (mostly) historical reasons.

.. option:: +no_off

    Do not offset origin to center of projection.

.. include:: ../options/k_0.rst

.. include:: ../options/x_0.rst

.. include:: ../options/y_0.rst

Caveats
#######

Note for the two-point method no rectification is done,

::

    echo 0 0|proj -I +proj=omerc +R=6400000 +lonc=-87 +lat_0=42 +alpha=0
    87dW    42dN
    echo 0 0|proj -I +proj=omerc +R=6400000 +lonc=-87 +lat_0=42 +alpha=0 +no_rot
    87dW    0dS
    echo 0 0|proj -I +proj=omerc +R=6400000 +lon_1=-87 +lat_1=42 +lon_2=-87 +lat_2=43
    87dW    0dN

Thus, just as was noted above regarding +no_rot, the two-point method
itself is also probably only marginally useful.
