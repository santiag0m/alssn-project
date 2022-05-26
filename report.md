# Abstract

In this report, we have collected data regarding the New York City subway network and transformed it into a graph network format. The graph dataset was analysed to find the most important subway stations within the network. This was done both by looking at overall weighted degree and centrality and inter and intracluster degree. Finally, we also tested the robustness of the network to find out if it can still function when under attack or with random failure. We find that parts of the subway network are more fragile (The Bronx, upper Manhattan, Queens) while others are relatively robust (lower Manhattan and Brooklyn). The resulting graph is not entirely scale-free due to the relatively limited number of transfer stations present in the network, specifically those more fragile areas.

# Introduction

The New York City subway network is the largest in the U.S.(12<sup>d</sup> largest in the world), it includes 493 stations, 25 lines and is over 1,100km long<sup>1</sup>. Subway stations are essential in providing sustainable transportation in large cities worldwide, reducing traffic congestion, pollution and accidents due to their efficiency and large capacity. In 2019, c. 9.1 million passengers used the subway network on an average weekday. During the recent covid pandemic, this fell back to 3.0 million in 2020 and 5.2 million in 2021, but social distancing and limiting overcrowding of public transport have become increasingly important. Subway networks are very inflexible in this regard. While subway networks can increase the capacity of train platforms (with proper infrastructure investment), they still require significant planning and a substantial investment to expand. 

New York has been planning the expansion of its subway station from its onset. In   1929 there was already a proposal to expand the network and increase the accessibility of northern New York to downtown new York by introducing a subway line beneath Second Avenue. It took until 2004 before these plans were approved, and in 2007 the work on the line started. In January 2017, the Metropolitan Transportation Authority (MTA) opened phase 1 of the Second Avenue Subway, a stretch of subway going from 63d street to 96d street. The purpose of the extension was to have a better connection between Harlem and the rest of New York, easing the congestion on (one of) the most overcrowded subway lines in the entire U.S., the Lexington Avenue subway line<sup>2</sup>. Phase 1 of the project cost $4.4 billion for just three new stops. The second phase involves pushing the line up to 125th Street, which has a projected cost of $6.3 billion<sup>3</sup>. MTA says the project should generate around 300,000 daily riders, shorten commute times by 20 minutes and decrease overall crowding on the 4,5 and 6 lines<sup>4</sup>.

A lot of work has been done in the field already. One of the oldest studies dates back to 2002, when Latora and Marchiori<sup>5</sup> studied the Boston subway system, showing that a closed transportation system can exhibit small-world behaviour. in 2010, Derrible and Kennedy<sup>6</sup> did a complexity and robustness analysis of 33 metro networks, finding that most metros are scale-free and small-worlds but that the scale-free property goes away in larger networks, with the inclusion of transfer hubs (more than three lines in one station). Other papers have assessed the robustness of metro networks to random failures and targeted attacks. Wang et al.<sup>7</sup> (2017) found that Rome and Tokyo are the most robust networks out of a sample of 33 metro networks. Rome, because of its short transfer routes and Tokyo because of the large number of transfer stations on the city periphery. Forero-Ortiz et al.<sup>8</sup> (2020) did an assessment of flood risk in the Barcelona metro network in the context of climate change. They discovered that 3 out of the 26 stations reviewed were at high risk of flooding under current weather conditions, but 11 out of 26 will be at increased risk in the next 20 years, given climate change. 

Building a sustainable subway infrastructure that considers future expansion, robustness and increased passenger counts is critical for a well-functioning underground public transport system. In this study, the robustness and potential pitfalls of the New York subway network are studied from a mathematical perspective, using the concepts of advanced network analysis. Additionally, we will study clustering patterns to determine where potential future congestion might occur. 

# Data

## Data collection


Data had to be gathered from several sources to analyse the New York subway network, as no readily available network format was available online. Three sources were consulted from the MTA developer page<sup>9</sup> to develop the graph: i) General Transit Feed Specification (GTFS), including data such as routes, stops, transfers, trips and calendar information, ii) stations, and iii) complexes. The lexicon works as follows: A complex facilitates multiple stations connected with a passageway inside fare control and serves as a transfer station. There are 32 complexes included in the dataset, but not all stations are part of a complex. A Station serves one line and has two stops, one going in each direction. The dataset includes a total of 496 stations. The data also comprises time-related data to measure how many trains pass through a station weekly, which routes pass through each station and at what time. An overview of the data and how the different sources connect with each other can be found in appendix 1<sup>a</sup>. 

<sup>a</sup>The reader is referred to https://developers.google.com/transit/gtfs/reference for a more detailed explanation regarding the lexicon.

## The network

The graph is made by first aggregating the stations at the complex level. This way, all stations follow the same hierarchy and have the same top-level, and clustering of stations can be studied more easily. Additionally, when checking for the robustness of the network, one can assume that if a part of a complex is out because of an attack on the network (or a random failure), the other parts of the complex are also out. The complexes are then used as the nodes of the network. There are 445 remaining nodes after aggregating stations based on complexes and creating complexes for stand-alone stations. Next, the edges are created. Even though the subway can go in two directions, the decision was made to make the edges undirected in the network. This is done because every single track goes in both directions, so all links between the nodes are directed in both directions. 

Several properties are also included in the graph. First, the graph is weighted based on the number of trains that pass through the network every week. Both the nodes and the edges get this property. Second, the borough (The Bronx, Brooklyn, Manhattan, Queens and Staten Island) in which each of the nodes is located is included, and last, the geographical coordinates (latitude and longitude) of the nodes. The below figure shows a first rendition of the network. The layout is based on the geographical coordinates, the colouring scheme is based on the boroughs, and the node and edge sizes are based on the number of trains that pass.

![image](https://user-images.githubusercontent.com/101331875/170033278-af98b507-6543-4985-b28e-97211176e481.png)

Figure 1: Graph layout using Gephi's Geo Layout module. Edge weights and node sizes are based on # of trains.

Note that there are also edges between nodes that are further away. These edges represent direct lines that skip a couple of stations (Staten Island, for example, clearly has this for two stations, going in a direct line to the last stop at the ferry). The existence of these edges has implications for the robustness analysis later on. It seems fair to assume that, if there is a node failure or a coordinated attack, not only can people no longer use the station, but trains can also no longer pass through that station (or at least, this is the implicit assumption this paper). So, a separate graph is made where these edges are removed to test for the robustness of the network.

# Methodology

The remainder of the paper will analyse the main characteristics of the network and the robustness of the network. 

## Graph characteristics

Degree and Betweenness centrality of the network are analysed, as well as some metrics from R. Guimerà et al.<sup>9</sup> (2004) from their paper on the global air transformation network. The methods used focus on the characteristics of the nodes within the communities they belong to. One of the metrics is the within community degree (z-score):

z<sub>i</sub> = (k<sub>i</sub>-avg(k<sub>s<sub>i</sub></sub>)) / σ<sub>s<sub>i</sub></sub>

with k the degree, i the node and s<sub>i</sub> the community to which i belongs. And the second one is a form of inter-community degree (participation coefficient):

P<sub>i</sub> = 1 - ∑ <sub>s</sub> (k<sub>s,i</sub> / k<sub>i</sub>)

with k<sub>s,i</sub> the degree of node i with regards to nodes in a different community s, and k<sub>i</sub> the total degree of i. Note that, contrary to R. Guimerà et al. we use the weighted degree for all our calculations. This gives a more fair representation of the actual importance of the nodes. Although, in this (relatively) small, closed network one can expect that the weighted degree is highly correlated to the degree, since degree is typically high for transfer complexes, and these complexes are made to handle a lot of trains. The weights used for the edges are the number of trips that happen on a weekly basis, across that edge. 

## Robustness

To assess the robustness of the network, we focus on the main component (ignoring Staten Island) and delete nodes one by one. More robust networks should take longer to break down into different components, while more weakly connected networks break down faster. Robustness impacts two types of situations: i) random failures of the network and ii) targeted attacks on the network. Robustness against random failures can be assessed by deleting random nodes from the network. Targeted attacks will attempt to cause most damage to the network. To find out which deletions cause most damage to the network we attempt several methods: i) deleting the most central nodes, ii) the nodes with the highest z-score, iii) nodes with the highest weighted degree, and nodes with the highest non-weighted degree.

# Results

## Graph characteristics

The highest weighted degree nodes in the graph are mainly part of Manhattan. Only Atlantic Avenue station lies in Brooklyn. Compared to the unweighted degree, some difference can be spotted. One can see that there are more nodes included that are not part of Manhattan. Looking at betweenness centrality still gives a different top 10. and Times square, the most important node in terms of (weighted) degree is only on the eight spot in the centrality ranking. The differences do make sense from a practical viewpoint. Manhattan is the busiest part of New York and most boroughs have trains going to Manhattan, explaining all the large weighted degree nodes, additionally, most trains pass by the busiest parts of town (like times square). In terms of unweighted degree, the transfer stations are showing up highest, since they have most connections in and out of the stations. Finally, centrality is highest at stations that funnel people from the outer boroughs to Manhattan. One can also see the importance of the 149st, 125st, 86st and Lexington Avenue / 59 st stations. Lowering the strain on these stations is precisely what the MTA had in mind with its second avenue project. 

![image](https://user-images.githubusercontent.com/101331875/170252822-b501c975-9a74-4f66-86ab-74e49d95b7a9.png)
Table 1-3: the top 10 nodes based on weighted degree, degree and centrality, respectively.

![image](https://user-images.githubusercontent.com/101331875/170256879-09bb22b5-76ec-41aa-95b7-b790b790db7a.png)
Figure 2: The geographical representation of the top 10 nodes depending on the metric used.

Next, we can look at these stations within their communities, using the z-score and participation-coefficient. For the purpose of this report we will limit ourselves to a predefined geographically inspired clustering, namely the boroughs of New York to which these stations belong. As mentioned before, there are 5 boroughs in New York: The Bronx, Brooklyn, Manhattan, Queens and Staten Island. They can also be seen on the first image included above. 

The z-score has a correlation of 0.79 to the weighted degree and 0.53 to centrality, this makes sense, as high centrality across the network seems to coincide with stations at key positions to enter Manhattan, while the z-score is related to key stations within the boroughs. The stations with the largest z-score are found mainly in Manhattan and Brooklyn. Queens and The Bronx seem to be underrepresented making these boroughs less robust and more prone to congestion and targeted attacks. 

![image](https://user-images.githubusercontent.com/101331875/170271662-abcbbfb7-16c9-46fe-a2ea-7496adb43f19.png)
Table 4-5: the top 10 nodes based on z-scores and participation scores respectively.

Figures 3 and 4 give a closer look at Brooklyn, which has a very dense structure. A lot of large metro stations are only streets apart from each other. Most of these stations lie fairly close to the East River, which provide a lot of access points from Brooklyn to Manhattan. Additionally, this part of the network is responsible for funnelling most of Brooklyn, as well as a part of Queens to lower Manhattan. 

![image](https://user-images.githubusercontent.com/101331875/170275990-0a56f3db-fa80-4705-91fc-57d7d9a67c59.png)

Figure 3: A closer look at Brooklyn. The size of the nodes is proportional to the z-score and the top 10 nodes are named. 

![image](https://user-images.githubusercontent.com/101331875/170277482-8ad52b05-1d50-4b14-b76c-f5aeafafe432.png)
Figure 4: part of the Brooklyn subway system<sup>10</sup>

The inter-community metric of the participation-score for a subway network seems less insightful at first sight. The highest participation score is 0.5, these are nodes that have an equal amount of edges going to another community as they have going to their own community. For a subway network one can expect that these are stations that have 2 stops, one to a different borough, and one to the current one. While this seems less insightful, it is actually interesting that none of the larger complexes have more lines going to another borough. When we look at lower Manhattan and Brooklyn (Figure 5), we can see that there are a lot of connections from Brooklyn to downtown Manhattan and that there are also a lot of nodes to support these connections. One can surmise that this adds to the robustness of the network. If all connections from Manhattan to Brooklyn would end up in Atlantic Avenue station, for example, it would be very easy to attack the network by focussing on that one station, and by doing so immediately cutting of the better part of Brooklyn from Manhattan. 

![image](https://user-images.githubusercontent.com/101331875/170294028-f16cbecb-b01c-4c16-87ff-d4bd71959a93.png)

Figure 5: Some of the nodes with a participation coefficient of 0.5 connecting Brooklyn to Manhattan.

Combining the insights of the z-score and participation coefficient analysis can be done by dividing the nodes into several roles depending on their scores on both metrics, as proposed by Guimerà et al. Figures 6 and 7 show the result of the clustering based on both metrics. 

![image](https://user-images.githubusercontent.com/101331875/170305460-52a4e5fd-9aeb-46b3-902c-15433c202889.png)

Figures 6&7: the network based on z-score (size of nodes) and participation coefficient clustering.

Yellow and brown nodes are the within community hubs (transfer stations / complexes), while other nodes are non-hubs. The black and red nodes are "ultraperipheral" and "peripheral" nodes: small nodes with all edges staying within the communities. The green nodes are "connector" nodes, these have low z-scores but connect the different communities with each other. 

There is only a limited number of hubs present in the system (c. 10), which has implications for robustness (see later). The brown nodes are the most important ones according to the z-score / participation analysis. These serve both a connecting roll as well as being a hub within the community. Interestingly, these nodes seem to be the main drivers of traffic from the Bronx (Grand Concourse) and northern Queens (Court square) to Manhattan. Both of these nodes seem to come together in Lexington square, one of the most central nodes in the network and also the reason for the second avenue expansion. 

## Robustness

First, the scale-free property of the network is assessed. A scale-free network is a network whose degree distribution follows a power law. This is contrary to the degree distribution of a random network, which follows a Poisson distribution. The general idea behind this, is that a large number of clusters is not something that happens in random networks. There is a disproportionate number of small degree nodes and only a limited number of smaller clusters, and large numbers are near impossible. This contradicts real-life, in actuality, networks tend to follow power laws, where the degree distribution is proportional to k<sup>-γ</sup> where γ is the degree exponent. 

We assessed this property for the New York subway network and found that, interestingly, new York is not completely scale-free and does lean towards random network behaviour (see figures 8 and 9 below). There are a lot of nodes that have two edges in the network (which makes sense for a railroad network, 1 line in and 1 line out) and a limited number of nodes with higher degree. Note, as a point of criticism, that the size of the network is fairly limited, making it difficult to find a statistically rigorous value for γ. So the conclusion regarding scale-freeness or no should be taken with a grain of salt. 

![image](https://user-images.githubusercontent.com/101331875/170454085-b6b08b57-d12b-4c43-bf75-afcf5fde7326.png)
Figures 8 & 9: the actual and prediction distribution of degree

The fact that the new York network leans more towards a network with a scaling factor has implications for its robustness. With a limited number of transfer stations present in the network it is easier to attack the network in order to break the structure. Translated to the subway network: an electricity outage or other infrastructure related outages in a key position could well cause for entire boroughs of New York to be disconnected from Manhattan. We saw earlier how Brooklyn seems relatively save from this, with a large number of transfer stations as well as many connector nodes. Queens, and even more so, The Bronx, should be more fragile in this regard. There are only a limited number of transfer stations in these areas that could really paralyze the network should they falter. 

The results of the robustness analysis can be seen in figures 10 and 11. Initially, the number of nodes deleted was compared to the number of disconnected components that are created. This is obviously not a fair assessment. Otherwise it would be easy to get a large impact by cutting off the second station at the tail end of every route, thus creating a disconnected component in the first station. The analysis was therefore expanded to take into account the affected number of trains in each component that was disconnected from the network. 

[Figures 10 & 11]

[Discussion of the results]

# Discussion

We investigated the New York subway network by creating a weighted graph using information on the station complexes, the routes and the weekly number of trips done throughout the network. We find that the New York subway network is fairly robust in some parts, but more fragile in others. The initial part of the analysis shows that lower Manhattan and Brooklyn have a lot of stations with high connectivity within and between their communities. The Bronx, upper Manhattan and Queens on the other hand are more fragile, having a couple of very central nodes with low degree. This clearly supports the need for the second avenue project that is currently happening to increase the accessibility of these neighbourhoods to downtown Manhattan, additionally, the network would be well of with more transfer stations in some of these locations to further increase the robustness of the network.

There are several limitations of this report and suggestions for further research that can be done. A major limitation is that we used the boroughs, a social construct, as basis for the clustering analysis. Alternatively, one can consider actual clustering methods, modularity based clustering for example, and compare these artificial communities to the boroughs. A more in depth analysis of robustness is also advised for follow-up research, specifically focussed on several areas of the network. In this way, the robustness of the lower Manhattan and Brooklyn part versus the upper Manhattan, The Bronx, and Queens part can be analysed more rigorously. Second, alternative methods of public transportation like busses and trains can also be included in the network as fail saves to outages in the subway network. Third, while we supplemented the unweighted analysis of public transportation by using weights for the number of weekly trains that pass through each station, one can also look at passenger information by analysing data from turnstiles. This way the congestion of both stations as well as trains can be analysed more in depth. 

# References

<sup>1</sup>Annual report MTA 2020: https://new.mta.info/document/43191

<sup>2</sup>https://www.nytimes.com/2017/01/01/nyregion/as-second-avenue-subway-opens-a-train-delay-ends-in-happy-tears.html

<sup>3</sup>https://www.nytimes.com/2022/01/31/nyregion/second-avenue-subway-harlem.html?smid=url-share

<sup>4</sup>https://new.mta.info/project/second-avenue-subway-phase-2

<sup>5</sup>V. Latora and M. Marchiori (2002). Is the Boston subway a small-world network? Physica A, 314, pp. 109-113

<sup>6</sup>S. Derrible & C. Kennedy (2010). The complexity and robustness of metro networks. Physica A, 389, pp. 3678-3691.

<sup>7</sup>X. Wang, et al. (2017). Multi-criteria robustness analysis of metro networks. Physica A: Statistical Mechanics and its Applications. 474. 10.1016/j.physa.2017.01.072. 

<sup>8</sup>Forero-Ortiz E, Martínez-Gomariz E, Cañas Porcuna M, Locatelli L, Russo B (2020). Flood Risk Assessment in an Underground Railway System under the Impact of Climate Change—A Case Study of the Barcelona Metro. Sustainability; 12(13):5291. 

<sup>8</sup>http://web.mta.info/developers/developer-data-terms.html#data

<sup>9</sup>R. Guimerà et al. (2005) The worldwide air transportation network: Anomalous centrality, community structure, and cities’ global roles. PNAS; 102(22): 7794-7799.

<sup>10</sup>Screenshot taken from https://map.mta.info/

# Appendix

## Appendix 1: Data schema

![Data_schema](https://github.com/santiag0m/alssn-project/blob/main/data_schema.png)
