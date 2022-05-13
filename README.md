# Greenspace_accessibility
Greenspace_accessibility in global cities

Time at 6-7 GB RAM average 12m run

The entrance model 1000m threshold is trained on Philadelphia, Pennsylvania, United States.
Input files are parks from parknet, city boundaries, road_network, edges and nodes from OSM and 250m population grids

Comments:
-	Buffer of 25 meters for calling two parks as one.
-	After which exclude parks less than 0.04 ha.
-	Only include population grids exclusively in the Philadelphia boundaries (size <0.99 of max size, size deviates slightly with lon-lat)
-	Working with the entrance nodes with 25m buffer of the road network of the parks.
-	Parks accessible within one kilometer of driving
-	Buffer of one kilometer outside or parks and the road network to the Philadelphia boundaries to ensure equal opportunities for all population grids.
-	Only calculate the Dijkstra algorithm on parks within 1km Euclidean distance.
-	Any score is 1000 minus the distance of the Dijkstra’s algorithm
-	Scores are aggregated on grids, parks and population.
-	Population grids with a population of 0 are exluded, there are no people which need park access. Working and recreational use of parks may be a topic of a further study.
-	Try both ways from grid to park and vice versa, we may encounter one way, walking and cycling often can be done both ways, parking in US cities is often available near the park. If both don't work (2 / 228000), use the next nearest park node for the path calculation
-	Use all roads, because close park access is most preferable for walking / cycling
