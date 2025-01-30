# Description

The tool identifies, segments and classifies roofs falling within the specified Area of Interest, basing on surface aspect computed from DSM and using footprints to define roof outer limits. Segmented roof parts are labelled with respect to 4 aspect classes, corresponding to NE, SE, SW and NW. The implemented technique is derived from the work described in this paper https://towardsdatascience.com/estimating-solar-panel-output-with-open-source-data-bbca6ea1f523 

# Input data requirements

* Area of Interest (AoI)
* Building footprints
* Digital Surface Model (DSM)

# Output data requirements

Vector segmentation of roof parts, having the following attributes:

* orientamento: average roof part aspect classification with respect to cardinal directions 
* orientamento_gradi: average roof part aspect in degrees from North to East
* pendenza_gradi: average roof part slope in percentage (0 = horizontal, 100 = at 45°, increasing to infinity = vertical)
* area_mq: area of the horizontal projection of the roof part

# Objective

Segment building footprints into sub-parts, basing on their average slope, in order to be enable further analyses on: roof part visibility from the ground, roof part average irradiation.

# Use of resource

* use the “Clip raster from extent” tool to clip the “DSM” raster layer to the “AoI– Area of Interest” vector layer
* use the “clip” tool to clip the input “Building footprint” vector layer to the “AoI– Area of Interest” vector layer
* use the “buffer” tool on the “footprints” vector layer to apply a 5 meter buffer and be sure to include the entire extension of the slopes
* use the “dissolve” tool on the buffered “footprints” vector layer
* extract the clipped “DSM” raster layer on the buffered and dissolved “footprints” vector layer, using the “clip raster with mask” tool (add the “keep input resolution” option and uncheck “match the extent”). This is to eliminate the “noise” that the other elements would generate in the subsequent steps
* with this DSM reduced to only the buffered buildings
   * calculate the cardinal orientation of the slopes (in degrees) using the “GDAL aspect” tool
   * calculate the slope of the slopes (in percentage) through the "GDAL slope" tool
* classify the aspect values with the "raster calculator" tool:
   * (logical_or(A == 360, A <= 90)) * 1 + ((A >= 90) * (A < 180)) * 2 + ((A >= 180) * (A < 270)) * 3 + ((A >= 270) * (A < 360)) * 4* 
* on the previous result, use the "sieve" tool by setting the threshold to 5 to eliminate the clusters that are too small
* on the previous result, use the "polygonize (from raster to vector)" tool with the aim of having a polygon per slope based on the homogeneous aspect value for contiguous pixels of the same slope
* use the "clip" tool to clip the "roof parts" vector layer just obtained from the polygonization tool with respect to the previously clipped vector layer obtained
* use the "Dissolve adjacent polygons" plugin to eliminate the small clippings
* determine the following attributes to associate with the individual roofs:
   * orientation_degrees: average value of the "Aspect" raster layer on the "Roof" output vector layer obtained with the "zonal statistics" tool
   * slope_degrees: average value of the "Slope" raster layer on the "Roof" output vector layer obtained with the "zonal statistics" tool
   * roof area: area of the horizontal projection of the roof, obtained with the "add geometry attributes" tool on the "Roof" layer
* use the "reorganize fields" tool to eliminate and reclassify the fields intended for the final "roof segmentation" vector layer
* Possible improvements:
   * To manage the problem of roofs that still fall in class boundary zones, you can try to make two different and parallel 4-class classifications, the second slightly rotated with respect to the first, then comparing/merging the results building by building, keeping "the best solution". Possible clue indicating two different classifications rotated 45° from each other: https://www.youtube.com/watch?v=W5ls8LjXeH8&t=777s 

--------------------------------------------------------------------------------------------------------
