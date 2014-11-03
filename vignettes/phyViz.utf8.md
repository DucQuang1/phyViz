---
title: "How to use the package **phyViz**"
author: "Lindsay Rutter, Susan Vanderplas"
output: 
  knitrBootstrap:::simple_document:
    toc: true
    main: true
    theme: cerulean
    highlight: idea
    clean_supporting: TRUE
---


<!--
%\VignetteEngine{knitr::knitr}
%\VignetteDepends{knitr}
%\VignetteIndexEntry{phyViz: Phylogenetic Visualization}

-->

**Description:** The **phyViz** package allows phylogeneticists to visualize and interpret their trees in multiple ways, each which come with their pros and cons. This vignette is intended to walk readers through the different options available with the package.

**Caution:** igraph must be used with version 0.7.0

# Preprocessing Pipeline:

There is a preprocessing pipeline that you must use before visualizing your phylogentic tree. First, you must load the necessary libraries. Note that loading the phyViz library should automatically load three required libraries (plyr, reshape2, igraph): 

<div class="row"><div class="panel panel-primary"><button class="btn btn-default btn-xs btn-primary"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="source r">library(phyViz)
library(ggplot2)
library(stringr)
library(knitr)</code></pre>
</div>
</div>


In the **phyViz** package, there is an example file containing phylogenetic tree of soybean varieties called **sbTree.rda**. It may be helpful to load that example file so that you can follow along with the commands and options introduced in this vignette. To ensure that you have uploaded the correct, raw **sbTree.rda** file, you can observe the first six lines of it:

<div class="row"><div class="panel panel-primary"><button class="btn btn-default btn-xs btn-primary"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="source r">data(sbTree)
head(sbTree)</code></pre>
</div>
<div class="panel panel-success"><button class="btn btn-default btn-xs btn-success"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="output r">##           child year yield year.imputed min.repro.year parent.type
## 1         5601T 1981    NA         TRUE            Inf      mother
## 2         Adams 1948  2734        FALSE           1960      mother
## 3          A.K. 1910    NA         TRUE           1912      mother
## 4 A.K. (Harrow) 1912  2665        FALSE           1955      mother
## 5        Altona 1968    NA        FALSE            Inf      mother
## 6         Amcor 1979  2981        FALSE            Inf      mother
##      parent
## 1 Hutcheson
## 2  Dunfield
## 3      <NA>
## 4      A.K.
## 5  Flambeau
## 6  Amsoy 71
</code></pre>
</div>
</div>


Now that the sbTree file has been loaded, it must now be converted into an **igraph** graph object. The function **treeToIG** requires a data frame structured such that each line represents an edge, and takes optional arguments *vertexinfo* (a list of columns of the data frame which provide information for the starting "child" vertex, or a separate data frame containing information for each vertex with the first column as the vertex name), *edgeweights*, which default to 1 if not provided, and *isDirected*, which describes whether the graph is directed (true) or undirected (false); the default is false. In this example, we want a graph with edge weights of 1 and an undirected graph because our goal is to determine whether two vertices (individuals) are related. Hence, we use the default conditions:

<div class="row"><div class="panel panel-primary"><button class="btn btn-default btn-xs btn-primary"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="source r">ig <- treeToIG(sbTree)</code></pre>
</div>
</div>


We can add the other data we have about each of the nodes in ig in several ways. The sbTree dataset contains information about each child; to add that data as is to the tree, we can use the same command as before, but specify the vertexinfo parameter as follows: 

<div class="row"><div class="panel panel-primary"><button class="btn btn-default btn-xs btn-primary"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="source r">ig <- treeToIG(sbTree, vertexinfo=c("year", "yield", "year.imputed", "min.repro.year"))</code></pre>
</div>
</div>


If we have separate data sets with node information, we can use that instead. For the example dataset, we must first obtain the node information: 

<div class="row"><div class="panel panel-primary"><button class="btn btn-default btn-xs btn-primary"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="source r">library(plyr)
nodes <- unique(sbTree[,1:5])

# get a data frame of all parents whose parents are not known (i.e. parents who are not listed as children as well)
extra.nodes <- unique(data.frame(child=sbTree$parent[!sbTree$parent%in%sbTree$child & !is.na(sbTree$parent)], stringsAsFactors=FALSE))

# We may not have information for these extra nodes, but they still need to be included in the dataset
nodes <- rbind.fill(nodes, extra.nodes)
rm(extra.nodes)

# We can now specify our vertex information using the data frame nodes: 
ig <- treeToIG(sbTree, vertexinfo=nodes)</code></pre>
</div>
</div>


The ig object is used in many of the other functions included with this package. 


# Functions for Individual Vertices:

**phyViz** offers several back-of-the-envelope functions that you can use to get information for individual vertices. 

First, the function **isParent** can return a logical indicator of whether the second variety is a parent of the first variety.

<div class="row"><div class="panel panel-primary"><button class="btn btn-default btn-xs btn-primary"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="source r">isParent("Young","Essex",sbTree)</code></pre>
</div>
<div class="panel panel-success"><button class="btn btn-default btn-xs btn-success"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="output r">## [1] TRUE
</code></pre>
</div>
<div class="panel panel-primary"><button class="btn btn-default btn-xs btn-primary"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="source r">isParent("Essex","Young",sbTree)</code></pre>
</div>
<div class="panel panel-success"><button class="btn btn-default btn-xs btn-success"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="output r">## [1] FALSE
</code></pre>
</div>
</div>


Similarly, the function **isChild** can return a boolean variable to indicate whether or not the first variety is a child of the second variety.

<div class="row"><div class="panel panel-primary"><button class="btn btn-default btn-xs btn-primary"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="source r">isChild("Young","Essex",sbTree)</code></pre>
</div>
<div class="panel panel-success"><button class="btn btn-default btn-xs btn-success"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="output r">## [1] TRUE
</code></pre>
</div>
<div class="panel panel-primary"><button class="btn btn-default btn-xs btn-primary"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="source r">isChild("Essex","Young",sbTree)</code></pre>
</div>
<div class="panel panel-success"><button class="btn btn-default btn-xs btn-success"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="output r">## [1] FALSE
</code></pre>
</div>
</div>


It is also possible to quickly derive the year of a given variety using the **getYear** function:

<div class="row"><div class="panel panel-primary"><button class="btn btn-default btn-xs btn-primary"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="source r">getYear("Young",sbTree)</code></pre>
</div>
<div class="panel panel-success"><button class="btn btn-default btn-xs btn-success"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="output r">## [1] 1968
</code></pre>
</div>
<div class="panel panel-primary"><button class="btn btn-default btn-xs btn-primary"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="source r">getYear("Essex",sbTree)</code></pre>
</div>
<div class="panel panel-success"><button class="btn btn-default btn-xs btn-success"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="output r">## [1] 1962
</code></pre>
</div>
</div>


Fortunately, these example results make sense in that the "Young" variety is a child to the "Essex" variety by an age difference of six years.

In some cases, you may wish to have a list of all the parents of a given variety. This can be achieved using the **getParent** function:

<div class="row"><div class="panel panel-primary"><button class="btn btn-default btn-xs btn-primary"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="source r">getParent("Young",sbTree)</code></pre>
</div>
<div class="panel panel-success"><button class="btn btn-default btn-xs btn-success"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="output r">## [1] "Davis" "Essex"
</code></pre>
</div>
<div class="panel panel-primary"><button class="btn btn-default btn-xs btn-primary"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="source r">getParent("Tokyo",sbTree)</code></pre>
</div>
<div class="panel panel-success"><button class="btn btn-default btn-xs btn-success"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="output r">## [1] NA NA
</code></pre>
</div>
<div class="panel panel-primary"><button class="btn btn-default btn-xs btn-primary"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="source r">getYear("Tokyo", sbTree)</code></pre>
</div>
<div class="panel panel-success"><button class="btn btn-default btn-xs btn-success"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="output r">## [1] 1907
</code></pre>
</div>
</div>


We learn from this that "Essex" is not the only parent of "Young". We also see that as "Tokyo" is a grandparent of the dataset (with a very early year, for this particular dataset, of 1907), it does not have any documented parents.

Likewise, in other cases, you may wish to have a list of all the children of a given variety. This can be achieved using the **getChild** function:

<div class="row"><div class="panel panel-primary"><button class="btn btn-default btn-xs btn-primary"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="source r">getChild("Tokyo",sbTree)</code></pre>
</div>
<div class="panel panel-success"><button class="btn btn-default btn-xs btn-success"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="output r">## [1] "Ogden"    "Volstate"
</code></pre>
</div>
<div class="panel panel-primary"><button class="btn btn-default btn-xs btn-primary"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="source r">getChild("Ogden",sbTree)</code></pre>
</div>
<div class="panel panel-success"><button class="btn btn-default btn-xs btn-success"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="output r">##  [1] "D55-4090"       "D55-4159"       "D55-4168"       "N45-745"       
##  [5] "Ogden x CNS"    "C1069"          "C1079"          "D51-2427"      
##  [9] "Kent"           "N44-92"         "N48-1101"       "Ralsoy x Ogden"
</code></pre>
</div>
</div>


We find that even though the "Tokyo" variety is a grandparent of the dataset, it only had two children. However, one of its children, "Ogden", produced twelve children.


# Functions for Pairs of Vertices:

Say you have a pair of vertices, and you wish to determine the degree of separation of the shortest path between them, where edges represent parent-child relationships. You can accomplish that with the **getDegree** function.

<div class="row"><div class="panel panel-primary"><button class="btn btn-default btn-xs btn-primary"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="source r">getDegree("Tokyo", "Ogden", ig, sbTree)</code></pre>
</div>
<div class="panel panel-success"><button class="btn btn-default btn-xs btn-success"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="output r">## [1] 1
</code></pre>
</div>
<div class="panel panel-primary"><button class="btn btn-default btn-xs btn-primary"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="source r">getDegree("Tokyo", "Holladay", ig, sbTree)</code></pre>
</div>
<div class="panel panel-success"><button class="btn btn-default btn-xs btn-success"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="output r">## [1] 7
</code></pre>
</div>
</div>


As expected, the shortest path between the "Tokyo" and "Ogden" varieties has a value of one, as they are a direct parent-child relationship. However, the shortest path between "Tokyo" and one of its great-reat-great-etc-granchildren, "Holladay" has a much higher degree. Note that degree calculations in this case are not limited to one linear string of parent-child relationships; cousins and siblings and products thereof will also have computatable degrees via nonlinear strings of parent-child relationships.

# Functions for the Whole Tree:

There are many parameters about the tree that you may wish to know that cannot easily be obtained through images and tables. With the use of the **igraph** package, the below function **getBasicStatistics** can be used to return a list of information about typical graph theoretical measurements of the whole tree. For instance, is the whole tree connected? If not, how many separated components does it contain? In addition to these parameters, the **getBasicStatistics** function will also return the number of nodes, the number of edges, the average path length, the graph diameter, etc.:

<div class="row"><div class="panel panel-primary"><button class="btn btn-default btn-xs btn-primary"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="source r">getBasicStatistics(ig)</code></pre>
</div>
<div class="panel panel-success"><button class="btn btn-default btn-xs btn-success"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="output r">## $isConnected
## [1] FALSE
## 
## $numComponents
## [1] 11
## 
## $avePathLength
## [1] 5.333746
## 
## $graphDiameter
## [1] 13
## 
## $numNodes
## [1] 230
## 
## $numEdges
## [1] 340
## 
## $logN
## [1] 5.438079
</code></pre>
</div>
</div>


In this case, we learn that our tree is actually not all connected by parent-child edges, and that instead, it is composed of four separate components. We see that the average path length of the tree is `{r} getBasicStatistics(ig)$avePathLength`, that the graph diameter is `{r} getBasicStatistics(ig)graphDiameter`, and that the logN value is `{r} getBasicStatistics(ig)logN`. We also see that the number of nodes in the tree is `{r} getBasicStatistics(ig)$numNodes` that are connected by `{r} getBasicStatistics(ig)$numEdges` edges.

# Visualization of Pathway between Two Vertices:

As this data set deals with soy bean lineages, it may be useful for agronomists to track how two varieties are related to each other via parent-child relationships. Then, any dramatic changes in protein yield, SNP varieties, and other measures of interest can be tracked across the genetic timeline, and pinpointed to certain varieties along the way.

The **phyViz** software can allow you to select two varieties of interest, and view the shortest pathway between them. You can produce a neat visual that informs you of all the varieties involved in the path between the two varieties of interest, as well as the years of all varieties involved in the path.

To produce and view this plot, two functions must be called in the order presented below (**getPath** and **plotPath**). We next introduce each of these two functions.

The **getPath** function determines the shortest path between the two inputted vertices, and takes into account whether or not the graph is directed. If there is a path, the list of vertices of the path (and their associated years) will be returned. For a directed graph, the direction matters. However, **getPath** will check both directions and return the path if it exists. The third parameter indicates the logical boolean of whether or not the graph is directed. Below, we look at all three possibilities (undirected and directed in reverse orders) between two varieties:

<div class="row"><div class="panel panel-primary"><button class="btn btn-default btn-xs btn-primary"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="source r">getPath("Brim","Bedford", ig, sbTree, isDirected=FALSE)</code></pre>
</div>
<div class="panel panel-success"><button class="btn btn-default btn-xs btn-success"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="output r">## $pathVertices
## [1] "Brim"    "Young"   "Essex"   "T80-69"  "J74-40"  "Forrest" "Bedford"
## 
## $yearVertices
## [1] "1977" "1968" "1962" "1975" "1975" "1973" "1978"
</code></pre>
</div>
</div>


To compute paths where direction matters, we must have a directed graph
<div class="row"><div class="panel panel-primary"><button class="btn btn-default btn-xs btn-primary"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="source r">dirgraph <- treeToIG(sbTree, vertexinfo=nodes, isDirected = TRUE)
getPath("Brim", "Bedford", dirgraph, sbTree, isDirected=TRUE)</code></pre>
</div>
<div class="panel panel-info"><button class="btn btn-default btn-xs btn-info"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="message r">## Warning: There is no path between those two vertices
</code></pre>
</div>
<div class="panel panel-success"><button class="btn btn-default btn-xs btn-success"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="output r">## list()
</code></pre>
</div>
</div>


<div class="row"><div class="panel panel-primary"><button class="btn btn-default btn-xs btn-primary"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="source r">getPath("Bedford", "Brim", dirgraph, sbTree, isDirected=TRUE)</code></pre>
</div>
<div class="panel panel-info"><button class="btn btn-default btn-xs btn-info"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="message r">## Warning: There is no path between those two vertices
</code></pre>
</div>
<div class="panel panel-success"><button class="btn btn-default btn-xs btn-success"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="output r">## list()
</code></pre>
</div>
</div>


We can derive from the empty list returned in the last two of the three commands that the varieties "Brim" and "Bedford" are not connected by a linear sequence of parent-child relationships. Rather, they are derived from a branch as some point, as siblings and/or cousins. Unless you are working with a dataset that must be analyzed as a directed graph, it is best to use the **getPath** function with the undirected specification (F) as the third parameter.

As such, we save the path between these two varieties to a variable called **path**:

<div class="row"><div class="panel panel-primary"><button class="btn btn-default btn-xs btn-primary"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="source r">path <- getPath("Brim","Bedford", ig, sbTree, isDirected=F)</code></pre>
</div>
</div>


Now that we have a **path** object that consists of two lists (the variety names and years), we can plot the relationship between the two using plotPath(path)

<div class="row"><div class="panel panel-primary"><button class="btn btn-default btn-xs btn-primary"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="source r">plotPathDF <- plotPath(path)</code></pre>
</div>
</div>


Indeed, as predicted above, the image verifies that the two varieties "Brim" and "Bedford" are cousin-like relationships, and are not connected by a linear string of parent-child relationships. Also, as mentioned above, in this plot, the x-axis represents the years, meaning that the center of the text box for each variety represents its corresponding year.

If, for some reason, you are looking at *directed* phylogenetic trees, you will need to use the true logical condition for the third parameter of the function **getPath**. The example below will hopefully reassure you that if you use this condition on a pathway composed of linear parent-child relationships, then either ordering of the first two parameters will work:

<div class="row"><div class="panel panel-primary"><button class="btn btn-default btn-xs btn-primary"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="source r">dirgraph <- treeToIG(sbTree, vertexinfo=nodes, isDirected = TRUE)

path <- getPath("Narow", "Tokyo", dirgraph, sbTree, isDirected=TRUE)
plotPathImage <- plotPath(path)
plotPathImage</code></pre>
</div>
<div class="row"><div class="col-md-offset-3 col-md-6"><a href="#" class="thumbnail">![](/var/folders/vn/lhzbs8ds6xbg965nnfcj6ftm0000gn/T/RtmpFNfXBj/preview-6847dd05f30.dir/phyViz_files/figure-html/unnamed-chunk-18-1.png) </a>
</div>
</div>
</div>


# Visualization of Pathway between Two Vertices Superimposed on Tree:

If you are curious to see the above-demonstrated shortest pathway between two vertices of interest, only now superimposed over all the varieties and edges in the whole tree, **phyViz** has a function (**plotPathOnTree**) that can achieve just that. 

<!--
The internal process is as follows: 
First, the **buildSpreadTotalDF** function creates a dataframe where the varieties are shuffled such that their overlap is minimized, even though the x-axis position will represent years. (Note: **binVector** will be explained in more detail at the end of this section).

```
binVector <- c(1,4,7,10,2,5,8,11,3,6,9,12)
spreadTotalDF <- buildSpreadTotalDF(ig, binVector)
head(spreadTotalDF)
```

Next, the **buildMinusPathDF** function takes the **ig** object and the **path** object (from the **getPath** function) as inputs. From these objects, it creates a data frame object of the label, x, and y values of all nodes in the tree. However, the data frame object does not include the labels of the path varieties, as they will be treated differently.

```
plotMinusPathDF <- buildMinusPathDF(path, ig)
head(plotMinusPathDF)
```

Third, the **buildEdgeTotalDF** function takes the **ig** object and creates a data frame object of the edges between all parent-child relationships in the graph.

```
edgeTotalDF <- buildEdgeTotalDF(ig)
head(edgeTotalDF)
```

Fourth, the **buildPlotTotalDF** function takes the **ig** object and the **path** object to create a data frame object of the text label positions for the varieties in the path, as well as the edges only in the varieties in the path.

```
plotTotalDF <- buildPlotTotalDF(path, ig)
head(plotTotalDF)
```

Now that we have generated the data frames needed to construct this type of visual object, we use them as input parameters to the **plotPathOnTree** function. 

-->
The outputted image will correctly position the node labels with x-axis representing the node year, and y-axis representing the node path index. Light grey edges between two nodes represent parent-child relationships between those nodes. To enhance the visual understanding of how the path-of-interest fits into the entire graph structure, the nodes within the path are labelled in boldface, and connected with light-green boldfaced edges.

<div class="row"><div class="panel panel-primary"><button class="btn btn-default btn-xs btn-primary"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="source r">plotTotalImage <- plotPathOnTree(path=path, ig=ig)
plotTotalImage</code></pre>
</div>
<div class="row"><div class="col-md-offset-3 col-md-6"><a href="#" class="thumbnail">![](/var/folders/vn/lhzbs8ds6xbg965nnfcj6ftm0000gn/T/RtmpFNfXBj/preview-6847dd05f30.dir/phyViz_files/figure-html/unnamed-chunk-19-1.png) </a>
</div>
</div>
</div>


Even though the edges of the large tree in this image are criss-crossing in all directions, that was not the focus, and hence the edges not belonging to the green path-of-interest are softly colored in a light gray. The highlight of this image was to keep all varieties in line with their appropriate year (corresponding to the x-axis), and to mitigate any overlap of the text of the varieties. Achieving this can be difficult, especially if your dataset has many varieties scrunched into a narrow set of years. That was the case with this dataset. As can be seen in the image above, most of the hundreds of varieties are associated with years between 1960 and 1975.

And this is where the explanation of the variable **binVector** (used as input parameters to the function **spreadTotalDF**) comes into play. As there is no uniform solution for this complicated problem, the **phyViz** package offers you the flexbility to changes these variables until you can optimize this image so that the text of the nodes of your tree overlap as little as possible. This can be done with a trial-and-error process by tweaking **binVector**'s order and length at the start.

Specifically, the length of **binVector** indicates how many bins of equal sizes should be allocated (where bins separate the vertices into groups of years). Theo order of **binVector** can be altered to avoid spatial collisions in labels.

The **binVector** will determine the order that increasing y index positions are repeatedly assigned to. For instance, if binVector = c(1,4,7,10,2,5,8,11,3,6,9,12), then y-axis position one will be assigned to a variety in the first bin of years, y-axis position two will be assigned to a variety in the fourth bin of years, ...., and y-axis position thirteen will be assigned again to a variety in the first bin of years. This will be repreated until all varieties from all bins have been assigned in that order. This vector can help minimize overlap of the labelling of varieties, as varieites from the same bins (near each other on the x-axis for years) will not have consecutive y-axis values.

The two examples below will show the importance of selecting an appropriate number of bins and order of bins to minimize overlap: 

<div class="row"><div class="panel panel-primary"><button class="btn btn-default btn-xs btn-primary"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="source r">plotTotalImage <- plotPathOnTree(path=path, ig=ig, binVector=1:12)</code></pre>
</div>
</div>


In this case, even though the number of bins is as large as the clearer-labelled image above, the order of the bins is such that varieties of consecutive x-values (years) will also have consecutive y-values (indices), and hence will be likely to overlap for years with many varieties.

<div class="row"><div class="panel panel-primary"><button class="btn btn-default btn-xs btn-primary"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="source r">plotTotalImage <- plotPathOnTree(path=path, ig=ig, binVector=1:2)</code></pre>
</div>
</div>


In this example, we see that with such a small number of bins chosen, the y-axis position designation will be similar to its original random state, and there is again much overlap in text variety labels in the year ranges where varieties occur the most.


# Generating Pairwise Distance Matrix between Set of Vertices:

It may also be of interest to scientists studying phylogenetics to generate heat maps where the color of each index of the map indicates the distance or years between two vertices.

The package **phyViz** also provides functions for that purpose. Specifically, for a given set of vertices, a heat map of the distance (degree of the shortest path) between all pairs of vertices can be constructed with the **plotDegMatrix** function:


<div class="row"><div class="panel panel-primary"><button class="btn btn-default btn-xs btn-primary"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="source r">varieties <- c("Beeson", "Brim", "Dillon", "Narow", "Zane", "Hood", "York", "Calland", "Columbus", "Crawford", "Kershaw", "Kent","Bragg", "Davis","Tokyo","Hagood","Young","Essex","Holladay","Cook","Century","Pella","Forrest","Gasoy","Cutler")
heatMapDegreeImage <- plotDegMatrix(varieties,ig,sbTree)
heatMapDegreeImage</code></pre>
</div>
<div class="row"><div class="col-md-offset-3 col-md-6"><a href="#" class="thumbnail">![](/var/folders/vn/lhzbs8ds6xbg965nnfcj6ftm0000gn/T/RtmpFNfXBj/preview-6847dd05f30.dir/phyViz_files/figure-html/unnamed-chunk-22-1.png) </a>
</div>
</div>
</div>


In a similar function, **plotYearMatrix**, the difference in years between all pairwise combinations of vertices can be constructed and viewed:

<div class="row"><div class="panel panel-primary"><button class="btn btn-default btn-xs btn-primary"><span class="glyphicon glyphicon-chevron-left"></span>
</button>

<pre ><code class="source r">varieties <- c("Beeson", "Brim", "Dillon", "Narow", "Zane", "Hood", "York", "Calland", "Columbus", "Crawford", "Kershaw", "Kent","Bragg", "Davis","Tokyo","Hagood","Young","Essex","Holladay","Cook","Century","Pella","Forrest","Gasoy","Cutler")
heatMapYearImage <- plotYearMatrix(varieties,sbTree)
heatMapYearImage</code></pre>
</div>
<div class="row"><div class="col-md-offset-3 col-md-6"><a href="#" class="thumbnail">![](/var/folders/vn/lhzbs8ds6xbg965nnfcj6ftm0000gn/T/RtmpFNfXBj/preview-6847dd05f30.dir/phyViz_files/figure-html/unnamed-chunk-23-1.png) </a>
</div>
</div>
</div>


Running this function on this particular set of vertices shows that most combinations of varieties are only a few decades apart in years, with only very few sets of vertices showing difference in years on the order of five or six decades.


# Conclusion:

The various options of the **phyViz** package can offer you different ways of visually interpreting your trees that each come with advantages and disadvantages. While some visualizations connect nodes to their years, other visualizations lax this idea, and instead connect nodes based on their generation surrounding a given node. And still other visualizations from **phyViz** will allow you to either focus on one pathway, or view that pathway superimposed across the entire tree for reference. Hopefully, the package will have something to offer to you in your data analysis.