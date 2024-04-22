---
jupytext:
  cell_metadata_filter: -all
  formats: md:myst,ipynb,py:percent
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.16.1
kernelspec:
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
---

# Working with Geographic Data

So, now we have seen how to build a map. But we are a Data Commons, and we work with **data**.  So let's start understanding data sources.

You are probably used to working with data in spreadsheets or in a tool like [AirTable](https://www.airtable.com). These tools organize data in **tabular format**  -- each dataset is a rectangle with a certain number of rows and columns.  In data science, we sometims call these structures **data frames**, and python has a special package to work with dataframes called [pandas](https://pandas.pydata.org). 

## Pandas Dataframes
Let's import the pandas package and create a simple dataframe.  We'll use a popular open dataset used in many examples on the web, called "iris", which records certain physical characteristics of 150 species of irises. 

:::{tip}
Notice that wer're using `import` a little differently here than the last time.  By adding `as pd`, we're telling python "import as usual, but instead of calling the pandas api with the name `pandas`, we'll use the name `pd`."  `pd` is the standard abbreviation used by most pythonistas.
:::

```{code-cell} ipython3
import pandas as pd
iris = pd.read_csv('https://raw.githubusercontent.com/mwaskom/seaborn-data/master/iris.csv')
iris
```

What happened here?  Well, we asked pandas to fetch a csv file on the web ( `https://raw.githubusercontent.com/mwaskom/seaborn-data/master/iris.csv`), and create a DataFrame object named "iris" from that CSV. And then we asked pandas to show us the the dataset. 

There is a lot to learn about pandas, but we'll ignore most of its complexities for now (we'll explore a tiny fraction of its capabilities later). Our real reason for introucing it is to show off an amazing extension of pandas called [geopandas](https://geopandas.org/en/stable/). Geopandas has a number of sophisticated features for working with geographic datasets.

## Geopandas Dataframes

Geopandas dataframes build on regular pandas. They can do everything pandas can do, but in adidtion, there are special mapping features that would take a lot of work for you to replicate with ordinary `pd.dataFrame` objects. The `GeoDataFrame` makes our lives a lot easier.  

Let's begin by acquiring the boundaries of all the countries in the world. The GIS company ESRI [provides such a dataset for free](https://hub.arcgis.com/datasets/esri::world-countries-generalized/explore), though they don't like to share the direct URL for it.  So instead, we have downloaded the dataset in a file near this textbook.

Unlike `iris`, this dataset is not stured as a CSV, but in a format called [GeoJSON](https://geojson.org/). Working with GeoJSON directly can be difficult and confusing, but geopandas makes it easy. Let's import the dataset and take a look:

```{code-cell} ipython3
import geopandas as gpd
countries = gpd.read_file('data/countries.geojson')
countries
```

ok, kind of cool. Now let's do something cooler.

```{code-cell} ipython3
countries.plot(edgecolor="white", color="green", linewidth=0.4)
```

Geopandas  made a map! And it's a bit hard to tell, but it _overlaid_ a layer of information with country borders on that map.  Here' let's include only the last 50 or so countries in the alphabet, by shortening the dataframe (and, especially, let's get rid of Antacrtica, so our map is a bit less confusing!).

```{code-cell} ipython3
countries.tail(50).plot( edgecolor="white", color="green", linewidth=0.4)
```

While we're here, let's just get rid of antarctica from our dataset. This is a little hard to read, and you probably have questions, but don't worry about them for now.

```{code-cell} ipython3
countries = countries[((countries.COUNTRY != 'Antarctica'))]
countries.plot()
```

## Merging datasets

OK, that was... fine, but not yet incredibly interesting.  What we really want is to show **the geographic dependencies of non-geographic variables.**  Like, how does wealth or population density vary by country?  

T0 do this, we will *merge* our ocuntry geometry data with a second dataset. One simple statistic, on which we have lots of data, is population.  Let's makea `choropleth` picture -- a map in which the color of each country is proportional to its population.

Fortunately, this data [is very easy to get](https://stats.oecd.org/Index.aspx?DataSetCode=EDU_DEM), and we've saved it in the file `data/population.csv`. It doesn't have any geographical geometry information, so we can't actually make it into a GeoDataFrame.  So we'll create a standards pandas DataFrame instead:

```{code-cell} ipython3
pop = pd.read_csv('data/world_population.csv')
pop.head(2)
```

Oh look, here's a stroke of luck. Remember our `countries` dataset from above:

```{code-cell} ipython3
countries.head(2)
```

**Both** datasets contain a column with the country names, and they appear to be identical!  This is gret news -- it means we can **merge** these datasets together, using the commons information as a "key"! Terrific. Now we just need to instruct `gpd` how to find that two columns -- which, you'll notice, have different names. Here's how we do that:

```{code-cell} ipython3
joined = gpd.GeoDataFrame(countries.merge(pop, left_on="COUNTRY", right_on='Country/Territory'))
joined.head(2)
```

As you can see, we have now combined the two datasets. GeoPandas has a quick and dirty way of showing data on a map, using colors on a spectrum across the range of values in one column. 

OK, let's work on that choropleth. Our dataset only has population numbers up to 2022, so we'll use that column as the basis of our choropleth. The instruction is simple:  we just pass the "column" argument to the `plot` method of the GeoDataFrame.

```{code-cell} ipython3
joined.plot(column='2022 Population', legend=True)
```

### Data Cleaning

+++

OK, that almost worked.  But there are a bunch of blank spots on the map.  It turns out that the names weren'texactly the same.  You could discover the errors yourself, but it's sort o a distraction, so we'll just show you what we found when we did this ourselves.  

```{code-cell} ipython3
countries = countries.replace("Congo DRC", "Democratic Republic of the Congo")
pop=pop.replace("DR Congo", "Democratic Republic of the Congo")
countries = countries.replace("Congo", "Republic of the Congo")
countries = countries.replace("Russian Federation", "Russia")
countries = countries.replace("Bosnia and Herzegovina", "Bosnia & Herzegovina")
joined = gpd.GeoDataFrame(countries.merge(pop, left_on="COUNTRY", right_on='Country/Territory'))
joined.plot(column='2022 Population', legend=True, figsize=(10,6))
```

### A tiny bit of math (areas, density, and growth rates)

+++

This is interesting. But what if the absolute value of the population isn't what we're interested in? For instance, what if we care more about population density?

Well, it's pretty easy to calculate density, and display it, but as you'll see here, there's kind of a problem: 

```{code-cell} ipython3
joined['density']=joined['2022 Population']/joined.area
joined['density']
joined.plot(column='density', legend = True, figsize=(12,6))
# joined.plot(column="Growth Rate", legend=True,figsize=(12,6))
```

The countries all look about the same, which is strange, because the scale goes form purple to yellow. 

THis is because there are several tiny island nations -- Monaco, Singapore, Gibraltar and the Maldives -- whose populatin density is way higher than anyone else's! They distort the scale, making everyone else look the same.  

There are lots of ways to try to adjust htis.  For instance, we could take these nations out of the dataset:

```{code-cell} ipython3
joined = joined[((joined['Country/Territory'] != 'Singapore')) & ((joined['Country/Territory'] != 'Monaco')) & ((joined['Country/Territory'] != 'Maldives')) & ((joined['Country/Territory'] != 'Gibraltar')) ]
joined.plot(column='density', legend = True, figsize=(12,6))
# joined.sort_values('density').tail()
```

alternatively, we can use logarithmic scale for the color distribution, which will also change the way the colors appear. 

```{code-cell} ipython3
joined.plot(column='density', legend=True,norm=matplotlib.colors.LogNorm(vmin=joined['density'].min(), vmax=joined['density'].max()), figsize=[9,6.5])
```

finally, let's try plotting growth rates instead.  We could calculate the growth rates ourselves form the underlying data, but let's take a shortcut and use the growth rates provided in the dataset itself: 

```{code-cell} ipython3
joined.plot(column='Growth Rate', legend=True, figsize=[9,6.5])
```
