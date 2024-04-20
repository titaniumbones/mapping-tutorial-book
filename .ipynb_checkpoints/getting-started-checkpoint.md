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

# How We make Maps

This interface should (I hope!) allow you to get started with Python and mapping.  There are many different ways to make maps in Python, and we will only explore one path today. In later work we may switch to a different framework, but most of the concepts will apply to other tools, even end-user tools like MapBox or StoryMaps. 

(section-label)=
## Imports: Build your toolbox

Python is the most widely-used prgramming language for scientific computing, and has a wide variety of tools for working with data. We get access to those tools by `import` ing them into our project. I won'texplain every one of these tools in this tutorial; imagine, in stead, that you have just started an apprenticeship at an auto mechanic shop.  There are many tools in the building, but you will only learn to use a few of them at first. 

```{code-cell} 
import folium
```
When we `import` a library, that library becomes available to us, and we can use its many attributes. A library generally provides a numbero f functions and objects (you can google to learn more about that that means) which you access using so-called "dot notation". These are often called the library's API. So for instance, folium has an API entrypoint called `Map`. Once the library is imported, we can access it by issuing the command

``` python
folium.Map()
```

(see next section)

### Interactive maps with Folium

[Folium](https://python-visualization.github.io/folium/latest/index.html) is the keystone of everything we will be learning.  It is a bridge between the python universe and JavaScript tools that work in the browser, so it allows us to build web-based interactive maps easily, without having to learn the complex syntax of JavaScript. 

Making a map is almost trivially easy:

```{code-cell}
sample_map = folium.Map()
sample_map
```
The first lines creates a "variable" called `sample_map` which is an "instance" of an object of class `folium.Map`. The second line, when issued as the final line in a Jupyter notebook cell like this one, will generate and display the map. 

This map is perfectly functional.  However, we can customize it, and make it more useful, by "passing arguments" when we create it:

```{code-cell}
sample_map = folium.Map(tiles='cartodbpositron',
                        location=[45.5019,-73.5674],
                        start_zoom=8
)
sample_map
```
