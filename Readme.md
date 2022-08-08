# Capital Metro Disruption Management Analysis

### Table of Contents

1. [General Synopsis](#Project-Context)
2. [Purpose](#Purpose)
3. [Data](#data)
   1. [Data Selection](#data-selection)
   2. [Disruption Model](#disruption-model)
4. [Analysis & Application](#analysis-&-application)
   1. [Statistics](#statistics)
   2. [ArcGIS Pro](#arcgis-pro)
   3. [AGOL & Instant App](#agol-&-instant-app)
5. [Conclusion](#conclusion)
6. [Appendix](#appendix)

## Project Context

Public transit systems are essential to the modern city. The robustness of a given public transit network is key to its success. A robust public transit network manages both passenger flow and vehicle patterns successfully at high orders of efficiency. Austin recently joined the top ten cities by population in 2022 with over 1 million people currently living here. With a 27.54% increase in population in the last decade, Austin is one of the fastest growing major cities in the United States. A growing city needs to have these management systems not only to adjust for changes in population, but also to indicate when and how to adjust. A growing population changes much about the metropolitan ecosystem in which public transit operates. Navigating and responding to these changes is necessary to maintain a smooth and effective public transit network.

Part of this management system monitors bus punctuality. When multiple buses are on the same route, a bus that is too early, or too late, can compound into a problem called bus bunching or clumping. This occurs when two or more transit vehicles running the same route are arriving at the same stops in tandem. Not only can this result in one bus running the route empty for quite a while, but also causes passengers to have to wait between buses for a longer period. 

CapMetro Austin requires analysis tools for vehicle location monitoring systems that can be regularly updated to determine incidents of bus bunching and potential causes. Preventing bus bunching keeps customers from waiting long periods at stops between bus arrivals and keeps buses at operating capacities. 

## Purpose

This project was designed to pilot an application for active management of public transit routes to prevent late stage disruptions in meeting scheduled stop times. Using a set of seven consecutive days of vehicle location history logs for CapMetro transit routes, the pilot application was constructed through a combination of Jupyter Notebooks, ArcGIS Pro, ESRI MapView, and ESRI Instant App. The pilot application was launched with public access and will not include any potentially sensitive information.

## Data

### Data Selection
Vehicle location history files (not included in repository) were acquired from CapMetro. The data was explored to determine candidate fields for detection of delay and disruption events. The five most active routes were used for the purpose of this project and were selected from the original dataset as shown below.

<figure style="text-align: center;">
<img src="/Presentation/Visuals/Entries_by_Route.png" alt="Routes by # of Records">
</figure>

### Disruption Model
Disruption indices were created for the selected routes using a series of algorithms described by Federico Malucelli and Emanuele Tresoldi in their paper "Delay and disruption management in local public transportation via real-time vehicle and crew rescheduling: a case study". DOI: [10.1007/s12469-019-00196-y](https://link.springer.com/article/10.1007/s12469-019-00196-y)

Malucelli and Tresoldi's pairwise model was selected from the available models presented in their paper, described below:

Lets consider the gap $x(q)$ in the planned $(v_e (q))$ and observed $(v_o (q))$ headways:
$$ x(q) = v_o (q) - v_e (q) $$

A negative value is an early pass and a positive value is a delay.

Malucelli and Tresoldi then define the function of $f(x(q))$ of the gap as:
$$
f(x(q))=
\begin{cases}
-\alpha x(q) & \quad \text{if $x(q) < -\theta_1$}\\
0 & \quad \text{if $-\theta_1 \leq x(q) < \theta_2$}\\
\beta x(q) & \quad \text{if $\theta_2 \leq x(q) < \theta_3$}\\
\gamma x(q)+\delta & \quad \text{if $\theta_3 \leq x(q)$}\\
\end{cases}
$$

Where $\theta_1$, $\theta_2$, $\theta_3$ and $\alpha, \beta, \gamma, \delta$ are suitable parameters. The function is 0 if the pass is regular and is not 0 if the pass is irregular. We can ignore contributions of earliness on the index by setting $\alpha = 0$. Likewise, if we set all coefficients to 0 and $\delta = 1$ we are left with the simple index the Azienda Trasporti Milanesi used at the writing of their paper. Values where $x(q) \geq \theta_3$ are intended to penalize large gaps more than the equivlent sum of small gaps.

Thus the index of regularity based on Malucelli and Tresoldi's piece-wise function is:
$$
I(PW)= \sum_{q \in P} f(x(q))
$$

Where $P$ is the set of all passes in a given period.

In consideration of time, we will only be constructing the piece-wise function from Malucelli and Tresoldi and using its related index for the purposes of analysis.

## Analysis & Application
### Statistics

The resulting indicies are less prone to skew from outliers and gives a standard scale to inspect bus 'lateness' and the accrual of delays. Below is a single day for the 801 line.

<figure style="text-align: center;">
<img src="/Presentation/Visuals/801_1-day_index_stats.png" alt="Route 801: 1-day Disruption Index">
</figure>

The mean index for route 801 on that particular Tuesday appears to reach its peak around 02:00PM. However, when we look at the rest of the week, we can see that the times at which a route reaches its peak disruption index is highly variable. For this reason, I intended to explore more real-time solutions to the problem.

<figure style="text-align: center;">
<img src="/Presentation/Visuals/801_index_stats.png" alt="Route 801: 7-day Disruption Index">
</figure>

### ArcGIS Pro

The workflow in ArcGIS Pro proceeded as follows: For each route; routes and stops are isolated into their own features. Indices were imported and joined to stop locations by the nextstopid field from the vehicle location history. Time was added to the new features and set to 5 minute intervals.

The resulting indices appear on the map for the selected time and time frame for all visible routes. Routes can be removed from the list of visible routes to reduce the amount of data on the map at one time.

<figure style="text-align: center;">
<img src="/Presentation/Visuals/Index_notime.png" alt="Route 801: All Disruption Index Points">
</figure>

### AGOL & Instant App

Map layers, with time enabled, were uploaded to MapViewer in a new map. Symbology for routes and disruption indices were reconfigured. Table elements were given aliases for readability. Time was enabled on the MapViewer and configured to its smallest (1h) interval.

A 'slider' template Instant App was created and configured to use the resulting map. Tables can be inspected by clicking on the disruption index points. Details regarding the time, stop and index value are available. Idealy, in a proprietary version of this app, bus ID and driver information would also be included in this informational table.

<figure style="text-align: center;">
<img src="/Presentation/Visuals/7f5cd3cc76b106dcd8030654b00b5d16.gif" alt="Application Timelapse">
</figure>

## Conclusion

This pilot shows the capability of displaying indices of delay and disruption for individual elements of public transit. With some modification, this application can be connected to a live feed of data and present real-time values of delay. The application can be used as an 'early warning system' for public transit when individual elements start becoming delayed to prevent incidents of 'bus bunching'.

Malucelli and Tresoldi outline several more models which may be more effective than the model presented here and include elements of driver shift behavior and short stops. CapMetro should assess the available model parameters for what best fits their current transit ecosystem.

## Appendix

### Documentation

[[link]](https://docs.google.com/presentation/d/1AW4Q_LKwa2--QCRk7GQ32kaJkKIo08IPth7FNVE0GgQ/edit?usp=sharing) Presentation

### Works Cited
Malucelli, Federico & Tresoldi, Emanuele. (2019). Delay and disruption management in local public transportation via real-time vehicle and crew re-scheduling: a case study. Public Transport. 11. 10.1007/s12469-019-00196-y. 

## File Structure
```
.
├── 01_New_Data                                     # Assimilated Data Folder
│   └── Python_Output
│       ├── Route_7_Disruption.csv
│       ├── Route_10_Disruption.csv
│       ├── Route_20_Disruption.csv
│       ├── Route_801_Disruption.csv
│       └── Route_803_Disruption.csv
├── Deliverables                                    # Deliverable files
│   ├── 2022_DMP_CapMetroDAM_Reynolds.docx
│   ├── 2022_SOW_CapMetroDMA_Reynolds.docx
│   └── 2022_WBS_CapMetroDMA_Reynolds.gantter
├── Presentation                                    # Images generated and used for Presentation
│   ├── Images
│   |   ├── Emanuele-Tresoldi.jpg
|   |   └── Federico-Malucelli.jpg
|   └── Visuals
│       ├── 10_route_stats.png
│       ├── 20_route_stats.png
│       ├── 7_route_stats.png
│       ├── 7f5cd3cc76b106dcd8030654b00b5d16.gif
│       ├── 801_1-day_index_stats.png
│       ├── 801_index_stats.png
│       ├── 801_route_stats.png
│       ├── 803_route_stats.png
|       ├── Entries_by_Route.png
|       ├── Index_notime.png
|       ├── Routes.png
|       ├── Stops_on_Route.png
|       ├── Layout.png
│       └── b72868f87e897c4fa7d59f10f052450c.gif
├── Python                                          # Jupyter Notebooks
│   ├── Disruption_Analysis_Compilation.ipynb
│   └── EDA.ipynb
├── .gitignore
└── Readme.md
```
