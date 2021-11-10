---
layout: post
title:  "Fractals for the New Blog Template"
date:   2021-11-07 20:15:00 +0200
background: '/img/bg-post.jpg'
---

For a while now I was thinking I should update the template I'm using for this blog to something a bit more aesthetically pleasing. After a bit of research I settled on a [Start Bootstrap theme](https://github.com/StartBootstrap/startbootstrap-clean-blog-jekyll). It's clean, easy to navigate and allows you to add title images to the main pages :). Since this is sort of a mathy blog, I decided to go with a mathy theme for said images, namely fractals!

I specifically went for the [Newton fractal](https://en.wikipedia.org/wiki/Newton_fractal) which allows you to effortlessly generate infinitely many distinct patterns. The idea behind the generation process is pretty simple: take a complex polynomial equation and run the Newton algorithm for that equation starting from various points in the complex plane (e.g. taken on an equally spaced grid). If you then color each point based on which root the Newton algorithm converges to (if it converges) you will often get a fractal pattern of infinitely nesting regions of different colors. Here are two examples of what can be obtained for a quintic and a cubic polynomial respectively (roots are shown as red dots):

<center>
	<table>
		<th><center>$x^5 - 3jx^3 - (5 + 2j)x^2 + 3x + 1$</center></th>
		<tr>
			<td><center>
				<div style="text-align:center;margin:20px 0px">
				<img class="img-fluid" src="/assets/fractals/quintic_w_roots.jpg">
				</div>
			</center></td>
		</tr>
		<th><center>$x^3 - 2x + 2$</center></th>
		<tr>
			<td><center>
				<div style="text-align:center;margin:20px 0px">
				<img class="img-fluid" src="/assets/fractals/cubic_w_roots.jpg">
				</div>
			</center></td>
		</tr>
	</table>
</center>

It's worth noting that fractality always happens at the boundaries between compact regions where all points converge to the same root. In the second image, corresponding to the cubic equation, I've also highlighted in black regions where Newton's algorithm doesn't actually converge, but rather cycles between two distinct values (0 and 1 in this case).

### Code optimization

One other useful thing to note is that leveraging NumPy can really decrease memory (and possibly CPU) usage. In my case the original code I wrote was building a dictionary of the form {complex_pt: Newton(complex_pt, *args)} out of the results of evaluating Newton's algorithm. For a grid of 1024x1024 pts, the memory use looked something like this (profiling done with guppy3, tables only show largest consumers):

<center>
	<table>
		<tr>
			<center>Total usage (bytes): 320.481.742</center>
		</tr>
		<tr>
			<th><center>Count</center></th>
			<th><center>Size</center></th>
			<th><center> % of total</center></th>
			<th><center>Kind</center></th>
		</tr>
		<tr>
			<td><center>2119504</center></td>
			<td><center>119541472</center></td>
			<td><center>37</center></td>
			<td><center>tuple</center></td>
		</tr>
		<tr>
			<td><center>896</center></td>
			<td><center>84407840</center></td>
			<td><center>26</center></td>
			<td><center>dict</center></td>
		</tr>
		<tr>
			<td><center>2097304</center></td>
			<td><center>50335296</center></td>
			<td><center>16</center></td>
			<td><center>float</center></td>
		</tr>
		<tr>
			<td><center>1048579</center></td>
			<td><center>33554528</center></td>
			<td><center>10</center></td>
			<td><center>complex</center></td>
		</tr>
		<tr>
			<td><center>811711</center></td>
			<td><center>22733464</center></td>
			<td><center>7</center></td>
			<td><center>int</center></td>
		</tr>
	</table>
</center>

The unusually large number of tuples came out of how I built the grid of points:

```python
def make_evaluation_grid(re_start, re_end, im_start, im_end, w=600, h=400):
    range_re = re_end - re_start
    range_im = im_end - im_start
    return {(re_start + (x / w) * range_re, im_start + (y / h) * range_im): (x, y) 
    									for x in range(0, w) for y in range(0, h)}
```

Using numpy arrays instead of dicts and tuples reduced memory consumption by an order of magnitude:

<center>
	<table>
		<tr>
			<center>Total usage (bytes): 52.723.229</center>
		</tr>
		<tr>
			<th><center>Count</center></th>
			<th><center>Size</center></th>
			<th><center> % of total</center></th>
			<th><center>Kind</center></th>
		</tr>
		<tr>
			<td><center>70</center></td>
			<td><center>33562054</center></td>
			<td><center>64</center></td>
			<td><center>numpy.ndarray</center></td>
		</tr>
		<tr>
			<td><center>41838</center></td>
			<td><center>5631164</center></td>
			<td><center>11</center></td>
			<td><center>str</center></td>
		</tr>
		<tr>
			<td><center>31115</center></td>
			<td><center>2742520</center></td>
			<td><center>5</center></td>
			<td><center>tuple</center></td>
		</tr>
	</table>
</center>

Therefore, a good conclusion of this whole exercise is not just that fractals make for pretty pictures, but also that NumPy is really powerful when it comes to reducing memory use.

As usual, the code is available on [GitHub](https://github.com/traian-d/fractals). As a bonus I also added an implementation of the Mandelbrot set.