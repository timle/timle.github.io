---
layout: default
---

Nate Silver Matlab Style Plots
=====================

Inspired by this post:
[https://www.dataorigami.net/blogs/fivethirtyeight-mpl]


I thought I would try and replicate the fivethirtyeight style within MATLAB. My goal was to require no touch-ups in post (i.e. Illustrator). What you see here is the direct output from MATLAB.  

By far, the largest complication was that MATLAB does not offer an easy way to remove the axis lines while retaining the tick labels and a grid. So the bulk of the code deals with drawing those elements manually.  

For alignment, I stuck with using the default of relative proportion. This is good if you plan on outputting different size plots, but makes alignment a little trickier. This is trade off between fivethirtyeight fidelity, and my own requirements.  

I used the excellent [export_fig](https://github.com/ojwoodford/export_fig) for rendering the png files.  


It's still a fairly crude implementation. I'd eventually like to set up automatically generated values to cut down on the number of options that need to be specified. Currently there are 31 options (!).


## Example 1
![Example 1](https://raw.githubusercontent.com/timle/ns_matlab_style_plots/master/ex1%2012-Jul-2014_low_.png "Example 1")

## Example 2
![Example 1](https://raw.githubusercontent.com/timle/ns_matlab_style_plots/master/ex2%2012-Jul-2014_low_.png "Example 2")





## How To


[Repo](https://github.com/timle/ns_matlab_style_plots)


The main function is:
```
ns17_plotter.m
```


Two examples (above) can be run via:
```
ns17_examples.m
```

