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

# Bring it All Together:  Fine-Tune your Folium Map

We've done so great!  Let's bring this all together, with a folium map that includes all the data we've done so far, with customized colors and controls.  

## Import Everything

```{code-cell} ipython3
import folium
import pandas as pd
import numbers
import geopandas as gpd
import branca.colormap as cm
from pyproj import Transformer
import xyzservices.providers as xyz
```

## Remake Our Data

This is a compressed version of the code from [our lest lesson](./geo-data.html).  If it is hard to follow, refer to that lesson.

```{code-cell} ipython3
pop = pd.read_csv('data/world_population.csv')
countries = gpd.read_file('data/countries.geojson')
countries = countries.replace("Congo DRC", "Democratic Republic of the Congo")
pop=pop.replace("DR Congo", "Democratic Republic of the Congo")
countries = countries.replace("Congo", "Republic of the Congo")
countries = countries.replace("Russian Federation", "Russia")
countries = countries.replace("Bosnia and Herzegovina", "Bosnia & Herzegovina")

joined = gpd.GeoDataFrame(countries.merge(pop, left_on="COUNTRY", right_on='Country/Territory'))
joined.head(2)
```

## Create our Map

Now we'll create a a map and add our data as a layer:

```{code-cell} ipython3
mymap = folium.Map(location=[30,0], zoom_start=3, min_zoom=3, max_bounds=True)
folium.TileLayer(xyz.CartoDB.Voyager).add_to(mymap)
folium.TileLayer(xyz.Esri.WorldGrayCanvas).add_to(mymap)
folium.TileLayer(xyz.CartoDB.PositronNoLabels).add_to(mymap)
folium.TileLayer(xyz.CartoDB.DarkMatterNoLabels).add_to(mymap)

l=folium.Choropleth(
    geo_data=joined,
    name="choropleth",
    data=joined,
    columns=["Country/Territory", "2022 Population"],
    key_on="feature.properties.Country/Territory",
    fill_color="YlGn",
    fill_opacity=0.7,
    line_opacity=0.2,
    use_jenks=True,
    legend_name="Population (millions)",
)
# l = folium.GeoJson(joined, column='2022 Population')
l.add_to(mymap)
mymap
```

## Reflections

This is just a start.  There's lots more work to do on this document, and lots more to each and learn.  I'd like to: 
- beta test this with staff
- stop working on this for now since I have other pressing deadlines
- think about whether this is an infrastucture we can use for other projects

