# Greenspace_accessibility
Greenspace_accessibility in global cities

The Modular Multiple Gravity Model builds on the principles of the gravity models. 

It is modular because you can give a set of inputs which run through the whole model. Inputs are a list of cities, distance thresholds, the max distance people walk from the park entry point, distance within which a park is considered one, which buffer-distance is taken to get road_nodes which form park entry points and the min park size can be set to exclude parks below it



Entrance model Time at 6-7 GB RAM average 12m run (Philadelphia)
Logarithmic Gravity model Time at 6-7 GB RAM average 15m run (Philadelphia)
(Absolute) Gravity model Time at 6-7 GB RAM average 25m run (Philadelphia)

The road extraction file extracts from OSMnx road networks of 15 carefully chosen world cities on which to run the greenspace accessibility model.

The 2 km buffer is 1000m metre for equal park access for each grid within city boundaries and 1000 metre extra for clean shortest route calculation, which may exceed the initial 1000m park buffer.

For Denver, Colorado, United States, parts of the Adams and Amarapoe networks are added to the model to get a 'boxy' shape. Otherwise the fastest real world routes (i.e. from south east Denver to the Airport via Aurora) may not be available. For this box a buffer is also created for the equal park access.

The multiple Gravity model is the gravity model in which four kinds of gravity are measured: No gravity in which park size doesn't play any role at all (this is the entrance based model), full gravity (increase in park size increases attractiveness), attractive-difference halved, and attractive-difference halved twice. I have ditched the logarithmic gravity model in this part, which difference over linear park size increase or decrease was not equal, it deviates especially at the tails of the distribution of park size.

For these four variants five measures summed for each grid and park. They are averaged over all grids or parks to compare cities.

1. Network score: 1000 - network_cost
2. Population score: network score multiplied by the origin grid population
3. Walkable park score: park size that can be reached within the threshold, assuming someone will walk 500m at max from the park entry point (the walkable park area)
4. Walkable park population score: walkable park score multiplied by the origin grid population
5. The number of parks that can be reached within the threshold (1000m network distance here)

The Gravity model 1000m is the entrance model with a preference added to the size of the park reachable within 500m of the park access point. The attactiveness of a park is adjusted for this size. The mean size means a service area of 1000m at the average reachable park area. When the reachable park distance is twice the average, a travel distance of 2000m is accepted, its route distance divided by 2 to calculate the scoresn and vice versa (see also the parks_gravity screenshot)

The logarithmic Gravity model solves the problem of overattractiveness of large destination areas, the area-sizes are compared logarithmically, which means the factor which adjusts the thresholds as described above is much smaller. The screenshots contain the 'preference' of grid-centroids, which park is closest (most attractive) according to the adjusted area-weighted route-distance. Grids are not tied to this park, multiple parks can be within the adjusted 1000m-range. The logarithmic is better in my opinion, it may have a bias towards smaller parks. This may be solved by adding an exponent.

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

The centroid model 1000m threshold is also trained on Philadelphia, Pennsylvania, United States. This is an earlier model compared to the entrance model which main difference is at the centroid model the road node closest to the park centroid is taken to calculate the park access, while at the entrance model all road nodes within a 25 metre buffer access is calculated.

The entrance model is much more massive than the centroid, performance upgrades are also part of the centroid model.
