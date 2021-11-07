---
layout: post
title:  "A Look at City Layouts"
date:   2020-07-25 20:01:00 +0200
background: '/img/bg-post.jpg'
---


Recently, as I was looking for a route on Google Maps, I sort of spontaneously realized that Amsterdam, the city I live in, can have unusually convoluted paths between two points that are quite close to one another on a straight line. Just take a look at the following craziness:

<div style="text-align:center;margin:20px 0px">
<img class="img-fluid" src="/assets/city_layouts/long_way_amsterdam.png">
</div>

To get from one side to the other of a canal that is about 30 m wide you have to take a detour of about 1 km. That's amusingly inefficient! How is it possible for something like this to arise? In this particular case it's quite clear the presence of canals makes things necessarily more complicated for land travelers. Is this, however, a thing that occurs often in Amsterdam? What about other cities without canals? Is there, perhaps, a way to quantify and/or visualize a street network's 'convolutedness'? Answers to these questions and others follow below :)

### General approach

The way I went about it was quite simple: sample a bunch of geographic coordinates from within a given city and (somehow) compute the street-wise distance between all of these points. These distances can then be compared to the straight line (Euclidean) distance in order to get a sense of how much longer the street path is, compared to the most effective route possible. As it happens this was not that hard to do, since I was already a bit familiar with [OSMnx](https://osmnx.readthedocs.io/en/stable/), a Python package that allows you to work with OpenStreetMap data quite effectively.

#### Sampling locations

While the first thought that comes to mind is to simply randomly sample some nodes from an OSMnx street graph, this in fact produces a completely non-uniform sample that will lead to really biased results. It's better therefore to sample locations uniformly from a grid. This can be done by first taking points on a grid and then finding the nearest graph nodes to these points. A bit of care needs to be applied to make sure there's no inadvertent clustering, but things tended to turn out OK for the cities I looked at. An illustration of the idea is as follows:

<table>
	<th><center>A grid of uniformly spaced points, within Amsterdam city boundaries</center></th>
	<tr>
		<td>
			<div style="text-align:center;margin:20px 0px">
			<img class="img-fluid" src="/assets/city_layouts/amsterdam_grid.png">
			</div>
		</td>
	</tr>
	<th><center>The corresponding closest points on the street network</center></th>
		<tr>
		<td>
			<div style="text-align:center;margin:20px 0px">
			<img class="img-fluid" src="/assets/city_layouts/amsterdam_street_pts.png">
			</div>
		</td>
	</tr>
</table>

Afterwards, you can use OSMnx to compute the street-wise shortest distance between all pairs of points in the sample. This step is quite computationally intensive, so it's a good idea to not sample from a grid that is too dense.

#### Making sense of things

Once street-wise and Euclidean distances have been computed, you can use their ratio as an indication of how convoluted a route between two points is. More so, since for every point on the grid a few hundred distances have been computed, you can actually average out these ratios in order to get a sense of how convoluted a point's neighborhood is. I did this by applying a kernel density estimator to the points, with observations weighted by their respective distance ratios. Overlapping the estimator on the street grid should hopefully show areas where the distance ratio is unusually high and hence where routes are really tangled and long. 

### Results

I ran this analysis on three cities: Amsterdam, Bucharest (my home town) and Manhattan. My prior expectation (that seems to have been confirmed) was that Amsterdam would be the most tangled one, Bucharest would be sort of in-between and Manhattan would be the easiest one to move around in (there's probably a reason why they actually named a distance after it :D). Results are as follows:

#### Amsterdam

Taking a look at some histograms, it seems that in Amsterdam the street distance distribution is slightly bi-modal. As will become apparent from the kernel density estimate, this is most likely because the northern part of the city (Amsterdam Noord) is quite poorly linked by streets. 

It's also important to note that the distance ratios can be very high, sometimes even larger than 30 (i.e. the street distance is more than 30 times longer than the straight line distance).

<div style="text-align:center;margin:20px 0px">
<img class="img-fluid" src="/assets/city_layouts/amsterdam_histograms.png">
</div>

A kernel density estimate shows clearly what was already suspected, namely that Noord is not well linked to the rest of the city. For pedestrians this is somewhat less of an issue since there is a free ferry service that runs quite often, but drivers do have to take huge detours sometimes.

<div style="text-align:center;margin:20px 0px">
<img class="img-fluid" src="/assets/city_layouts/amsterdam_with_weights.png">
</div>

As a benchmark, I also included a kernel density estimate with unweighted observations. In this case the density is quite uniform (with small clusters around the sampled points). The difference between the two images clearly shows the impact of adding weights to the estimator:

<div style="text-align:center;margin:20px 0px">
<img class="img-fluid" src="/assets/city_layouts/amsterdam_no_weights.png">
</div>


#### Bucharest

In Bucharest things look better. The distance ratios are acceptable and the histograms of the distances themselves are unimodal. The longer right tail present in the distance histograms of all cities is, I think, due to the fact that points at the edge of a city tend to have fewer close neighbors and are therefore further away from all other points in the sample. This is, I think, simply a hallmark of the data.

<div style="text-align:center;margin:20px 0px">
<img class="img-fluid" src="/assets/city_layouts/bucharest_histograms.png">
</div>

Since there are quite few areas with very large distance ratios, in the case of Bucharest I made an estimate only for areas with a ratio >= 1.25.

<div style="text-align:center;margin:20px 0px">
<img class="img-fluid" src="/assets/city_layouts/bucharest_with_weights_dist_geq_125.png">
</div>

#### Manhattan

Manhattan is, as expected, the easiest to move about it. Distance histograms are again skewed (reflecting perhaps also the length and narrowness of the island), but distance ratios look quite good.

<div style="text-align:center;margin:20px 0px">
<img class="img-fluid" src="/assets/city_layouts/manhattan_histograms.png">
</div>

Even a weighted density estimate ends up being quite uniform in this case. The only way to detect some sort of hotspots is to do an estimate only for places with distance ratios >= 1.5. 

<table>
	<tr>
		<td>
			<div style="text-align:center;margin:20px 0px">
			<img class="img-fluid" src="/assets/city_layouts/manhattan_with_weights.png">
			</div>
		</td>
		<td>
			<div style="text-align:center;margin:20px 0px">
			<img class="img-fluid" src="/assets/city_layouts/manhattan_with_weights_dist_geq_15.png">
			</div>
		</td>
	</tr>
</table>

The hotspot towards the north of the island seems to correspond to the entrance to the George Washington Bridge. The ramps and other infrastructure surrounding the bridge seem hard to traverse for pedestrians.

<div style="text-align:center;margin:20px 0px">
<img class="img-fluid" src="/assets/city_layouts/manhattan_gw_bridge.png">
</div>

### Conclusion

It seems that a kernel density estimate of the distance ratios for uniformly sampled points might be a useful way to assess not only connectivity differences between cities, but also potential issues within a given city. As was expected, Amsterdam tends to yield the most twisted street routes (largely due to its canals and especially due to the IJ), Bucharest is pretty well connected (with the exception of some peripheral hotspots) and Manhattan takes the crown - for this one it's actually hard to find a place where the street distance is different from -well- the Manhattan distance.

As usual, a Jupyter notebook and related Python functions are available on [GitHub](https://github.com/traian-d/city_routes).