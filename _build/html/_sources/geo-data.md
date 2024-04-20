---
jupytext:
  cell_metadata_filter: -all
  formats: md:myst
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
Let's import the pandas package and create a simple dataframe.  We'll use a popular open dataset used in many examples on the web, called "iris".

:::{tip}
Notice that wer're using `import` a little differently here than the last time.  By adding `as pd`, we're telling python "import as usual, but instead of calling the pandas api with the name `pandas`, we'll use the name `pd`.  pdf is the standard abbreviation used by most pythonistas.
:::


```{code-cell}
import pandas as pd
iris = pd.read_csv('https://raw.githubusercontent.com/mwaskom/seaborn-data/master/iris.csv')
iris
```

What happened here?  Well, we asked pandas to fetch a csv file on the web ( `https://raw.githubusercontent.com/mwaskom/seaborn-data/master/iris.csv`), and create a DataFrame object named "iris" from that CSV. And then we asked pandas to show us the the dataset. 

There is a lot to learn about pandas, but we'll ignore most of its complexities for now (we'll explore a tiny fraction of its capabilities later). Our real reason for introucing it is to show off an amazing extension of pandas called [geopandas](https://geopandas.org/en/stable/). Geopandas has a number of sophisticated features for working with geographic datasets.

## Geopandas Dataframes

Geopandas dataframes build on regular pandas. They can do everything pandas can do, but in adidtion, there are special mapping features that would take a lot of work for you to replicate with ordinary `pd.dataFrame` objects. The `GeoDataFrame` makes our lives a lot easier.  

Let's begin by acquiring the boundaries of all the ocuntries in the world. The GIS company ESRI [provides such a dataset for free](https://hub.arcgis.com/datasets/esri::world-countries-generalized/explore), though they don't like to share the direct URL for it.  So instead, we have downloaded the dataset in a file near this textbook.

Unlike `iris`, this dataset is not stured as a CSV, but in a format called [GeoJSON](https://geojson.org/). Working with GeoJSON directly can be difficult and confusing, but geopandas makes it easy. Let's import the dataset and take a look:

```{code-cell}
import geopandas as gpd
countries = gpd.read_file('data/countries.geojson')
countries
```

ok, kind of cool. Now let's do something cooler.

```{code-cell}
countries.plot(edgecolor="white", color="green", linewidth=0.4)
```
Geopandas  made a map! And it's a bit hard to tell, but it _overlaid_ a layer of information with country borders on that map.  Here' let's include only the last 20 or so countries in the alphabet, by shortening the dataframe (and, especially, let's get rid of Antacrtica!).

```{code-cell}
countries.tail(40).plot( edgecolor="white", color="green", linewidth=0.4)
```
## Merging datasets

OK, that was... fine, but not yet incredibly interesting.  What we really want is to show **the geographic dependencies of non-geographic variables.**  Like, how odes wealth or population density vary by country?  

T0 do this, we will *merge* our ocuntry geometry data with a second dataset. One simple statistic, on which we have lots of data, is population.  Let's makea `choropleth` picture -- a map in which the color of each country is proportional to its population.

Fortunatley, this data [is very easy to get](https://stats.oecd.org/Index.aspx?DataSetCode=EDU_DEM), and we've saved it in the file `data/population.csv`. It doesn't have any geographical geometry information, so we can't actually make it into a GeoDataFrame.  So we'll create a standards pandas DataFrame instead: 

```{code-cell}
pop = pd.read_csv('data/world_population.csv')
pop.head(2)
```

Oh look, here's a stroke of luck. Remember our `countries` dataset from above:

```{code-cell}
countries.head(2)
```

**Both** datasets contain a column with the country names, and they appear to be identical!  This is gret news -- it means we can **merge** these datasets together, using the commons information as a "key"! Terrific. Now we just need to instruct `gpd` how to find that two columns -- which, you'll notice, have different names. Here's how we do that:

```{code-cell}
joined = gpd.GeoDataFrame(countries.merge(pop, left_on="COUNTRY", right_on='Country/Territory'))
joined.head(2)
```
As you can see, we have now combined the two datasets. GeoPandas has a quick and dirty way of showing data on a map, using colors.

```{code-cell}
joined.plot(column='2022 Population')

```
