# Abstract

In this paper we collected data regarding the New York City subway network and transformed it to a graph network format. The graph dataset was analysed to find the most important subway stations within the network. This was done both by looking at overall weighted degree and centrality as well as inter and intra cluster degree. Finally, we also tested the robustness of the network to find out if it can still function when under attack or with random failure. We found [...]

# Introduction

The New York City subway network is the largest in the U.S.(12<sup>d</sup> largest in the world), it includes 493 stations, 25 lines and is over 1,100km long<sup>1</sup>. Subway stations are essential in providing sustainable transportation in large cities all over the world, reducing traffic congestion, pollution and accidents due to their efficiency and large capacity. In 2019, c. 9.1 million passengers used the subway network on an average weekday. During the recent covid pandemic, this fell back to 3.0 million in 2020 and 5.2 million in 2021, but social distancing and limiting overcrowding of public transport have become increasingly important to look at. Subway networks are very inflexible in this regard. Bus stops can be added easily and even train networks can increase the capacity of train platforms (with proper infrastructure investment) but subways require a lot of planning and very large investment to expand. 

New York has been planning the expansion of its subway station from its onset. In   1929 there was already a proposal to expand the network and increase accessibility of northern New York to downtown new York by introduction of a subway line beneath Second Avenue. It took until 2004 before these plans were approved and in 2007 the work on the line started. In January 2017, the Metropolitan Transportation Authority (MTA) opened phase 1 of the Second Avenue Subway, a stretch of subway going from 63d street to 96d street. The purpose of the extension was to have a better connection between Harlem and the rest of New York, easing the congestion on (one of) the most overcrowded subway lines in the entire U.S., the Lexington avenue subway line<sup>2</sup>. Phase 1 of the project had a $4.4 billion cost, for just three new stops. Phase 2, pushing the line up to 125th street, has a projected cost of $6.3 billion<sup>3</sup>. MTA says the project should generate around 300,000 daily riders, shorten commute times by as much as 20 minutes and decrease overall crowding on the 4,5 and 6 lines<sup>4</sup>. 

A lot of work has been done in the field already. One of the oldest studies dates back to 2002, when Latora and Marchiori<sup>5</sup> did a study on the Boston subway system, showing that a closed transportation system can exhibit small-world behaviour. in 2010, Derrible and Kennedy<sup>6</sup> did a complexity and robustness analysis of 33 metro networks, finding that most metros are scale-free and small-worlds but that the scale-free property goes away in larger networks, with the inclusion of transfer hubs (more than three lines in one station). Other papers have assessed the robustness of metro networks to random failures and targeted attacks. Wang et al.<sup>7</sup> (2017) find that Rome and Tokyo are the most robust networks out of a sample of 33 metro networks. Rome, because of short transferring and Tokyo because of the large number of transfer stations on the city periphery. Forero-Ortiz et al.<sup>8</sup> (2020) did an assessment of flood risk in the Barcelona metro network in the context of climate change. They discovered that 3 out of the 26 stations reviewed were at high risk of flooding under current weather conditions, but 11 out of 26 will be at high risk in 20 years, given climate change. 

Building a sustainable subway infrastructure that keeps future expansion, robustness and increased passenger counts into account is key for a well-functioning underground public transport system. In this study, the robustness and potential pitfalls of the New York subway network are studied from a mathematical perspective, using the concepts of advanced network analysis. Additionally, clustering patterns will be uncovered to find out where potential future congestion might occur. 

# Data

## Data collection

In order to analyse the New York subway network, data had to be gathered from several sources, as no readily available network format was available online. Three sources were consulted from the MTA developer page<sup>9</sup> to develop the graph: i) General Transit Feed Specification (GTFS) including data such as routes, stops, transfers, trips and calendar information, ii) stations, and iii) complexes. The lexicon works as follows: A complex facilitates multiple stations connected with a passageway inside fare control and serves as a transfer station. There are 32 complexes included in the dataset. Not all stations are part of a complex, a Station serves one line and has 2 stops, one going in each direction. The dataset includes a total of 496 stations (which is 3 more than what is mentioned in the MTA annual report, but that one dates back 2 years). The data also includes time related data to measure how many times a trip is done each day, what trains pass each station and at what time. An overview of the data and how the different sources connect with each other can be found in appendix 1<sup>a</sup>. 

<sup>a</sup>The reader is referred to https://developers.google.com/transit/gtfs/reference for a more detailed explanation regarding the lexicon.

## The network

The graph is made by first aggregating the stations at the complex level. This way, all stations follow the same hierarchy and have the same top level, and clustering of stations can be studied more easily. Additionally, when checking for robustness of the network one can assume that if a part of a complex is out because of an attack on the network (or a random failure) that the other parts of the complex are also out. The complexes are then used as the nodes of the network. In total there are 445 remaining nodes after aggregating stations based on complexes and creating complexes for stand-alone stations. Next, the edges are created. Even though the subway can go in two directions, the decision was made to make the edges undirected in the network. This is done because every single track goes in both directions, so all links between the nodes are directed in both directions. 

Several properties are also included in the graph. First, the graph is weighted based on the number of trains that pass through the network every week. Both the nodes and the edges get this property. Second, the borough (The Bronx, Brooklyn, Manhattan, Queens and Staten Island) in which each of the nodes is located is included, and last the geographical coordinates (latitude and longitude) of the nodes. The below figure shows a first rendition of the network. The lay-out is based on the geographical coordinates, the colouring scheme is based on the boroughs, and the node and edge sizes are based on the number of trains that pass.

![image](https://user-images.githubusercontent.com/101331875/170033278-af98b507-6543-4985-b28e-97211176e481.png)
Figure 1: Graph lay-out using Gephi's Geo Layout module. Edge weights and node sizes are based on # of trains.

Note that there are also edges between nodes that are further away. These edges represent direct lines that skip a couple of stations (Staten Island for example clearly has this for two stations, going in a direct line to the last stop at the ferry). This has implications for the robustness analysis later on. It seems fair to assume that, if there is a node failure or a coordinated attack, not only can people no longer use the station, trains can also no longer pass through that station (or at least, this is the implicit assumption of this paper). So, a separate graph is made where these edges are removed to test for the robustness of the network.

# Methodology

The remainder of the paper will analyse the main characteristics of the network and the robustness of the network. 

## Graph characteristics

 Degree and Betweenness centrality of the network are analysed, as well as some metrics taken from R. Guimerà et al.<sup>9</sup> (2004) from their paper on the global air transformation network. The methods used focus on characteristics of the nodes within the communities they belong to. One of the metrics is the within community degree (z-score): 

z<sub>i</sub> = (k<sub>i</sub>-avg(k<sub>s<sub>i</sub></sub>)) / σ<sub>s<sub>i</sub></sub>

with k the degree, i the node and s<sub>i</sub> the community to which i belongs. And the second one is a form of inter-community degree (participation coefficient):

P<sub>i</sub> = 1 - ∑ <sub>s</sub> (k<sub>s,i</sub> / k<sub>i</sub>)

with k<sub>s,i</sub> the degree of node i with regards to nodes in a different community s, and k<sub>i</sub> the total degree of i. Note that, contrary to R. Guimerà et al. we use the weighted degree for all our calculations. This gives a more fair representation of the actual importance of the nodes. Although, in this (relatively) small, closed network one can expect that the weighted degree is highly correlated to the degree, since degree is typically high for transfer complexes, and these complexes are made to handle a lot of trains. The weights used for the edges are the number of trips that happen on a weekly basis, across that edge. 

## Robustness

...

# Results

## Graph characteristics

The highest weighted degree nodes in the graph are mainly part of Manhattan. Only Atlantic Avenue station lies in Brooklyn. Compared to the unweighted degree, some difference can be spotted. One can see that there are more nodes included that are not part of Manhattan. Looking at betweenness centrality still gives a different top 10. and Times square, the most important node in terms of (weighted) degree is only on the eight spot in the centrality ranking. The differences do make sense from a practical viewpoint. Manhattan is the busiest part of New York and most boroughs have trains going to Manhattan, explaining all the large weighted degree nodes, additionally, most trains pass by the busiest parts of town (like times square). In terms of unweighted degree, the transfer stations are showing up highest, since they have most connections in and out of the stations. Finally, centrality is highest at stations that funnel people from the outer boroughs to Manhattan. One cna also see the importance of the 149st, 125st, 86st and Lexington Avenue / 59 st stations. Lowering the strain on these stations is precisely what the MTA had in mind with its second avenue project. 

![image](https://user-images.githubusercontent.com/101331875/170252822-b501c975-9a74-4f66-86ab-74e49d95b7a9.png)
Table 1-3: the top 10 nodes based on weighted degree, degree and centrality, respectively.

![image](https://user-images.githubusercontent.com/101331875/170256879-09bb22b5-76ec-41aa-95b7-b790b790db7a.png)
Figure 2: The geographical representation of the top 10 nodes depending on the metric used.

Next, we can look at these stations within their communities, using the z-score and participation-coefficient. For the purpose of this report we will limit ourselves to a predefined geographically inspired clustering, namely the boroughs of New York to which these stations belong. As mentioned before, there are 5 boroughs in New York: The Bronx, Brooklyn, Manhattan, Queens and Staten Island. They can also be seen on the first image included above. 

The z-score has a correlation of 0.79 to the weighted degree and 0.53 to centrality, this makes sense, as high centrality across the network seems to coincide with stations at key positions to enter manhattan, while the z-score is related to key stations within the boroughs. The stations with the largest z-score are also split fairly well across the network. However, we can see that Brooklyn is represented a lot more strongly in the highest z-scores compared to the highest overal (weighted) degree. 

![image](https://user-images.githubusercontent.com/101331875/170271662-abcbbfb7-16c9-46fe-a2ea-7496adb43f19.png)
Table 4-5: the top 10 nodes based on z-scores and participation scores respectively.

Figures 3 and 4 give a closer look at Brooklyn, which has a very dense structure. A lot of large metro stations are only streets apart from each other. Most of these stations lie fairly close to the East River, which provide a lot of access points from Brooklyn to Manhattan. Additionally, this part of the network is responsible for funneling most of Brooklyn, as well as a part of Queens to lower Manhattan. 

![image](https://user-images.githubusercontent.com/101331875/170275990-0a56f3db-fa80-4705-91fc-57d7d9a67c59.png)
Figure 3: A closer look at Brooklyn. The size of the nodes is proportional to the z-score and the top 10 nodes are named. 

![image](https://user-images.githubusercontent.com/101331875/170277482-8ad52b05-1d50-4b14-b76c-f5aeafafe432.png)
Figure 4: part of the Brooklyn subway system



## Robustness


# Conclusion



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

# Appendix

## Appendix 1: Data schema

![Data_schema](https://github.com/santiag0m/alssn-project/blob/main/data_schema.png)