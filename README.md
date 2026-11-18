# Binders (Incognito Recommended)

## Homepage and File Access

[![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/opeyemiman/GPSDistanceSpeed/main)

## Main Notebook
[![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/opeyemiman/GPSDistanceSpeed/main?urlpath=voila%2Frender%2FGPSDistanceSpeed.ipynb)

## GPSDISTANCESPEED
GPS Distance and Speed Calculator Help

### Purpose
This calculator reads consecutive GPS observations containing latitude, longitude, and numeric timestamps. 
For each pair of consecutive points, the calculator determines distance, elapsed time, and interval speed.


### CSV inputs
The calculator supports three input workflows:

Upload CSV: Select a CSV file from the computer. The entire file is copied into the editable text area.\n

Paste manually: Paste CSV text directly into the text area.

Load example: Load the built-in example CSV.


### CSV columns
The CSV must include at least:

A latitude column in decimal degrees; A longitude column in decimal degrees; A numeric timestamp column.

Example:

time,lat,lon
0,40.0000,-105.0000
1,40.0005,-105.0003
2,40.0011,-105.0007
3,40.0016,-105.0010
    
Timestamp differences are assumed to be seconds. Timestamps must increase from one data row to the next. 

### Latitude and longitude
Latitude indicates north-south angular position. Valid decimal latitude values range from -90 to +90 degrees.

Longitude indicates east-west angular position. Valid decimal longitude values range from -180 to +180 degrees.

Latitude and longitude are angular coordinates, not direct linear distances. One degree of longitude represents less ground distance as a location approaches either pole.

### WGS84 ellipsoidal geodesic
WGS84 means World Geodetic System 1984. WGS84 is the reference coordinate system commonly used by GPS. WGS84 approximates Earth as a flattened ellipsoid. 
Earth is slightly wider around the equator than the distance between the geographic poles. A geodesic is the shortest surface path between two coordinates on the selected Earth model.

The program calls:

forward_azimuth, reverse_azimuth, distance_m =
    WGS84_GEOD.inv(lon1, lat1, lon2, lat2)
    
The function returns forward direction, reverse direction, and ellipsoidal surface distance in meters. Only distance is needed to calculate speed.

WGS84 geodesic is not a map projection. A map projection transforms curved Earth coordinates into flat map coordinates. This option calculates distance directly on the WGS84 ellipsoid.

#### Benefits:

Appropriate for short and long segments.

Accounts for Earth's ellipsoidal shape.

Works globally.

Does not require a manually selected reference latitude.

Recommended for final or defensible distance results.

### Local Cartesian approximation
The local method treats each short segment as movement on a flat x-y plane. The program calculates latitude and longitude differences:

delta_latitude = latitude_2 - latitude_1
delta_longitude = longitude_2 - longitude_1
    
The program calculates latitude-dependent scale factors at the midpoint latitude of each segment.

dx = delta_longitude * meters_per_degree_longitude
dy = delta_latitude * meters_per_degree_latitude
    
The east-west displacement is dx. The north-south displacement is dy.

The distance is calculated with:

distance = sqrt(dx^2 + dy^2)
    
This is the Pythagorean theorem applied to a short local segment. This method is a local Cartesian approximation. It is not a complete formal Orthographic map projection. A formal Orthographic projection requires a projection center and specific spherical or ellipsoidal projection equations.

Benefits:

Simple calculation.
Computationally inexpensive.
Easy to interpret as local x-y movement.
Highly effective for closely spaced vehicle observations.
Limitations:

It is an approximation.
Error increases as individual segments become longer.
It should not be treated as one global flat coordinate system.
Special care is required near the geographic poles.
7. Which method should be selected?
Select WGS84 ellipsoidal geodesic when global validity, accuracy, consistency, or defensibility is most important. WGS84 is the default and generally recommended option.

Select Local Cartesian approximation for closely spaced vehicle GPS observations when a transparent local planar calculation is desired.

For one-second vehicle points separated by several meters, the difference between the two methods will normally be much smaller than ordinary consumer or vehicle GPS position noise.

8. Distance formulas
Local Cartesian distance:

d = sqrt(dx^2 + dy^2)
    
Cumulative distance:

cumulative_distance[i] =
    cumulative_distance[i - 1] + segment_distance[i]
    
Cumulative distance is the sum of all point-to-point segments. It may differ from road-network distance or vehicle odometer distance.

9. Speed formulas
delta_time_s = timestamp_2 - timestamp_1
speed_mps = segment_distance_m / delta_time_s
speed_ftps = speed_mps * 3.280839895
speed_kmh = speed_mps * 3.6
speed_mph = speed_mps * 2.236936292
    
The calculated speed is an interval speed between two GPS coordinates. It is not the same as instantaneous GPS Doppler speed reported directly by some GPS receivers.

10. Output columns
calculation_method: Selected distance method.
relative_time_s: Seconds since the first timestamp.
segment_duration_s: Seconds since the preceding row.
segment_distance: Distance from the preceding point.
cumulative_distance: Sum of completed segment distances.
speed: Segment distance divided by segment duration.
11. GPS measurement error
Selecting a more accurate distance formula does not eliminate location measurement error. GPS coordinates may move while a vehicle is stationary because of receiver error, satellite geometry, obstructed sky view, coordinate rounding, and signal reflections from buildings or other surfaces.

For example, a false 3-meter position change over one second produces:

3 meters / 1 second = 3 m/s
3 m/s = approximately 6.71 mph
    
For noisy data, consider adding a stationary threshold, maximum plausible speed, GPS accuracy filter, or smoothing procedure.

12. Date-line handling
The local method normalizes longitude differences to the range -180 to +180 degrees. This prevents a date-line crossing from being interpreted as nearly 360 degrees of east-west movement.

13. Results file
Processed output is written to results.csv. A download link appears after successful processing.
