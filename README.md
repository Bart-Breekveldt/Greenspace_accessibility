# Greenspace_accessibility
Greenspace_accessibility in global cities

I have integrated the WorldPoP data into the Modular ParksOSM Gravity Model. The WorldPoP-OSM Gravity Model follows the base structure of this model. I have also changed the procedure in WorldPoP extraction. Changes all all included in the WorldPoP-OSM Gravity Model.

1. The countries Russia, Canada, United States, China, Brazil and Australia are too big to load at once into Python before clipping the city out of it. I have used the QGIS raster functionality of mask clipping, which clips over

I have tried to do an extraction of worldpop from the WorldPoP GitHub: https://github.com/wpgp/wpgpDownloadPy. I have
1. Downloaded countries ISO3 codes.
2. Listed population files per country
3. Downloaded most recent and rich population files through the terminal
4. Transformed the GeoTIFF to a GeoDataFrame in geopandas
5. Here extracted Athens boundaries (for the sake of example) from Greece.
6. Get a bounding box for Athens and filtered the nodes on the x and y coordinates (to reduce the computational time for all the geometric functions. We go from 20 million cells to 12750.
7. Got Point geometries from the GeoTIFF's x and y coordinates
8. Get a square buffer of 50 meters to recreate a cell (50m is half the 100m resolution used)
9. Did an overlay analysis and only get the cells within over 99% area of the max and cells with population in it. (rounded)
10. The formatted 'popgrid' can be used for processing in the Modular ParkOSM Gravity Model

**MAIN ISSUE**: I get errors when I want to download GeoTIFF population grids for a specific country in the notebook, I have to do this in the terminal, which prevents the script in becoming entirely modular (you can download all in advance and then select them in the notebook, but this isn't efficient). In my mind we can do A get the extracting to work or B execute a terminal command in the notebook (but I haven't figured out how). I think B is simple if you know how to, but A is more sustainable.

ISSUE is at this piece of code from the GitHub
![afbeelding](https://user-images.githubusercontent.com/83957293/172495212-d5a75e34-c3ac-4caa-af3e-d29bbb9192e9.png)

I have added a file. I have tried both igraph and networkit packages according to this research https://www.timlrx.com/blog/benchmark-of-popular-graph-network-packages. Igraph was slower for the first 100K, except for Dublin. Networkit is very fast for route-finding alone, because it uses auto-multiprocessing, but all the capacity is taken when you do formatting and dissolving the routes, which adds hugely to the computation time. The data scores differed in osmnx and networkit, because osmnx is build for this kind of analysis and networkit is a more general network finding tool, I trust (way) more in the results of osmnx. You can find them in the file time taken. The main reason for these packages, Dublin which took ~ 7 hours in osmnx, is gone. In the ParkOSM it gets half the grid-parkentry combinations within 1000m euclidean threshold (which to process in the routing step). The process times now are as follows

Sparser networks with less combinations compute faster (networks in Europe are denser than the US and Bangladeshi ones)

Philadelphia ~ 11m for 320K combinations
Denver ~ 19m for 316K combinations
Ghent ~ 30m for 567K combinations
Amsterdam ~ 52m for 607K combinations
Dhaka ~ 4s for 7799 combinations
Dublin ~ 85m for 896K combinations

Pre-processing and summarizing the data both add around 5-10 minutes to the computation time.

The Modular ParksOSM Gravity Model follows the principles of the Modular Multiple Gravity Model. Improvements are
1. While loop within network pathfinding algorithm bugfix: lengths of lists differ when joining with information of the original df. Non-existing values for routes that can't be found are given as -1 (NaN gives problems in summarizing) to equal the route numbers it puts out
2. Made importing the park modular by importing them straight from OSM, results (Dublin still to add) in the popgrid_access ParkOSM and its adjusted version for comparing between thresholds.
3. Found that igraph is actually slower than networkx (file). I think this is because igraph is matrix based which becomes increasingly sparse when the number of nodes grow. I also compared with directly calling graph.nodes(), see csv time shortest path functions.

**Issue arose when importing WorldPoP**, the world can be downloaded in 1km resolution, not in 100m. Unable to use Python/R package or get sense of the REST API in which I have no experience. Please tell how to import WorldPoP data.

The Modular Multiple Gravity Model impr. decay builds on the principles of the Modular Multiple Gravity Model. Changes with this last model are:
v 2.0
1. Divided the workload into chunks of 250.000 routes at a time, 1.8 million (of grid-parkentry combinations in dense Urban Roads of Dublin) is too much at once. This was already done for finding parkentry-grid combinations (looking for combinations per 1000 grids) (Philadelphia ~ 55 million possible ones), but is now also done for calculating the network routes.
2. Added a while-loop to the block of calculating network routes. It searches for nearest neighbours if the original nodes route calculation grid to park and the reverse is not possible. This happens in the order: original grid-alternative park node, the reverse of the previous, original park-alternative grid node and its reverse. A max of 10 alternative grid nodes and 10 alternative park nodes, those which are closest in order to the original nodes, are searched for.
3. Calculated the adjusted radius-euclidean size score. This addresses the problem that the area of a circle (the catchment area) grows faster than the plain difference between the thresholds. The adjustment factor is calculated with 600m as base: (threshold area / 600m) / (threshold / 600m). The scores are adjusted with this factor and then summarized in 'radius-euclidean adjusted popgrid access' in fractions for the populations access to parks according to this scores. The advantage is comparability between thresholds, the disadvantage is that it can bend reality a bit which is plain network access distance. Therefore I calculate both scores side by side.
4. The scores of Dublin, Ireland are added to popgrid access csv file, for the adjusted version this still needs to happen.
5. Ordered print outputs of the network route calculation block.

v 1.5
1. Added preferenced parks for each grid and each distance decay variant, which is the park with the lowest plain network route score.
2. Added structured storage of output of grid_scores, park_scores, network routes, park_preferences and population access categorization scores.

v 1.0
1. Hectares per person added as a metric to all grid- and parkscore files for each city and gravity metric.
2. Removed selection of park-entry points because of too large errors to the original
3. Changed the Gravity model variants to plain entrance, gravity ** (1/2), gravity ** (1/3) and gravity ** (1/5). According to the paper "How do travel distance and park size influence urban park visits?", the combined relation of distance and park size with the number of visitors follows very close the distribution of gravity ** (1/3), the gravity ** (1/2) and gravity ** (1/5) are set for comparing lower and upper bounds plus the original plain entrance model (the difference between gravity ** (1/3) and gravity ** (1/4) is very small, so gravity ** (1/5) is taken
4. The original full gravity variant is removed. According to the mentioned paper this variant is totally unreasonable and it halves the executing time of the script as a whole. I will test the city of Dublin to this new variant. The full gravity model inflates the number of grids attracted to very large parks to many, which adds to less performance because the area increases faster than its diameter or radius.

The Modular Multiple Gravity Model builds on the principles of the gravity models. The same selections of parks, grids, park entry_points and pathfinding and return shortest routes and scores are used as in the previous Gravity models.

It is modular because you can give a set of inputs which run through the whole model. Inputs are a list of cities, distance thresholds, the max distance people walk from the park entry point, distance within which a park is considered one, which buffer-distance is taken to get road_nodes which form park entry points and the min park size can be set to exclude parks below it and the size to merge entry points to improve performance

Changes
1. Multiple cities and other assumptions can now manually be preset and changed
2. City boundaries are taken from OSMnx for modularity
3. Greenspace and grid info are named uniformely and put in maps for modularity of getting those data
4. Use of median as 1 to adjust for park size instead of mean
5. Get a popgrid_access_R (see attachment) for grouped population access in high, medium, low and no acccess to parks from grids.
6. These scores are normalized between 0 and 1.
7. Multiple cities and thresholds are used within one script. Taken here are Philadelphia, Denver, Ghent, Amsterdam and Dhaka Metropolitan (province and city also named Dhaka, city is small, like City of London). Thresholds taken are 300m and 1000m. 500m is for comparing.
8. The largest threshold is taken when calculating metrics, scores are awarded afterwards according to all thresholds.
9. Extracting roads is included in the script, this has huge advantages for modularity and I had to load the graph again anyway because saved edge and node files together in a graph gave errors in pathfinding in road networks.
10. Kill many entry points by merging entry points within 17.5 metres (best by visual inspection of Philadelphia) for computational performance. The centroid of the multipoint geom is dertemined, the point closest to this centroid is taken.

Important notes on this.
1. The road networks of Ghent and especially Dublin are really dense, within the same euclidean distance threshold they require more steps in the Dijkstra algorithm. Dublin was left out for now for this reason. Looping the network distance alone without any formatting was > 95% of the computational load of its Jupyter block. Philadelphia, Denver, Amsterdam and Dhaka require between 19 and 28 steps on average with a 10.000 networks run. Ghent 41 and Dublin 72. Ghent takes 3.28m per 10.000, Dublin 12.78m. The rest is between 0.7m and 1.0m per 10.000. When having hundreds of thousands of grid-parkentry combinations within the largest threshold, this becomes a problem. I aim to include Dublin in a model with imrpoved performance.
2. The merging of entry points score per category quite differs from the score when those entry points are unmerged. The absolute average difference is 8,17% on 300m and 500m thresholds. 1000m was computationally expensive to do, so this is not compared. This can be (one-way) slip lanes as center or the greater distance created when a closest entry point is merged and is now behind the threshold. 8.17% is too big, however this gets less with larger thresholds.
3. The high-medium-low-no scores are calculated from the threshold itself. Mathmatically the area of a circle (buffer) increases faster than its radius or diameter. The scores therefore generally improve at each larger threshold. It depends if preferable to adjust for this, if comparing between thresholds, like 300m and 1000m, it needs to, but if comparing the same thresholds between cities, it don't have to.
4. Philadelphia performed quite well over the whole script, most were slower. This caused computational performance hastles, which took a large amount of time.

Entrance model Time at 6-7 GB RAM average 12m run (Philadelphia)
Logarithmic Gravity model Time at 6-7 GB RAM average 15m run (Philadelphia)
(Absolute) Gravity model Time at 6-7 GB RAM average 25m run (Philadelphia)

The road extraction file extracts from OSMnx road networks of 15 carefully chosen world cities on which to run the greenspace accessibility model. The cities are a good representation of HDI, location in the world, gdp per capita, mean age, size and more. The research is mainly focussed on larger cities, these are overrepresented. Cities are chosen from data availability of 100 cities, with mostly western cities. Cities in countries with a large population like China, India and Brazil are chosen because policy will probably lead to comparable city design per country to some point, thus models will be more likely to be applicable. Initially these cities are Philadelphia, Denver, London, Madrid, Warsaw, Mexico City, Lima, Sao Paulo, Seoul, Beijing, Jakarta, Bangalore, Cairo, Nairobi, Lagos

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
