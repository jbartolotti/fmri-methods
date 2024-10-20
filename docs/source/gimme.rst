GIMME, Group Iterative Multiple Model Estimation 
=============


GIMME is a model-building approach from within a unified structural equation modeling (uSEM) framework. uSEMs are a type of model that identify lagged and contemporaneous relations between nodes. Failing to account for lagged relations in fMRI can lead to spurious contemporaneous connections (Gates et al., 2010, c.f. Beltz & Gates 2017 and c.f. Gates & Molenaar 2012)

Shared information is used to select directed connections for all individuals (without concatenating individuals' timeseries), but edge weights are estimated separately for each individual. In addition, individual-level edges are allowed. 

Compared to fixed network patterns across individuals, individual networks and edge weights better reflect heterogeneity present in many fMRI samples. Compared to individual models derived in isolation, the shared connections reliably recover brain relationships despite individual variability, and facilitate comparing edge weights across all individuals.

.. _modelfitting:

Model Fitting Method
-----------

1. Identify all Group-level connections

2. Identify all Subgroup-level connections (if applicable)

3. Identify Individual-level connections for each individual

Group Connections
~~~~~~~~~~~~~~~

Starting from null models for each individual, gimme identifies the connection that significantly improves model fit for the greatest number of individuals (Bonferroni-corrected alpha). If there is a tie, then the connection that has the highest average improvement to model fit is selected. If the selected connection significantly improves model fit for >= 75% of individuals (user-defined % threshold), then that connection is added to *every* individual model.

Using the new base model, gimme searches for the next connection that significantly improves model fit for the greatest number of individuals. This process repeats until no connections are identified that improve model fit for >= 75% of individuals.

After identifying a group-level model, group connections are pruned. If any connection is now significant in < 75% of individuals, it is removed (if multiple group connections meet this criteria, the connection with the lowest z value is selected). Pruning continues from the new model until no group connections are identified for pruning.

Subgroup Connections
~~~~~~~~~~~~~

If subgroup membership is defined *a priori*, then subgroup connections are identified within each subgroup separately, using the same procedure as group connection identification. The subgroup threshold is allowed to differ from the group threshold. 

After subgroup connections are identified in all subgroups, group connections are pruned again.

If subgroup membership is not defined *a priori*, subgroups are identified using the community detection method Walktrap (Pons & Latapy 2006, c.f. Lane & Gates 2017), and then subgroup connections are identified as above.

Individual Connections
~~~~~~~~~~~~~~~~~

Using the group and subgroup paths as a starting model, individual models are estimated. For each individual, the connection that most increases model fit is identified. Any previously-identified non-significant individual connections are pruned (group and subgroup connections cannot be pruned at this step). 

Individual connections are added iteratively until "excellent fit" is achieved. Excellent fit is achieved when two of the four model fit indexes are true:

1. Root mean square of approximation (RMSEA) < .05

2. Non-normed fit index (NNFI) > .95

3. Comparitive fit index (CFI) > .95

4. Standardize root mean square residual (SRMR) < .05

.. _data:

Data Recommendations
------------

**Recommended timecourse length:** 200 timepoints yields accurate recovery of both path presence and direction in simulated data; 50 timepoints is sufficient for path presence (92-100% recovery), but poor direction recovery (Gates & Molenaar, 2012).

**Recommended sample size:** Minimum 10 per subgroup (Gates & Molenaar, 2012)

**Recommended nodes:** 5-15 recommended, up to 3-20 (Beltz & Gates 2017; Lane & Gates 2017). More than 20 is possible but increases computation time.

**Recommended group-connection threshold:** 75% (majority threshold for neuroimaging research; van den Heuvel & Sporns 2011, c.f. Lane & Gates 2017)

Timecourses **can** be different lengths between participants.

Missing rows (i.e. discrete timepoints) are fine, up to a limit (estimation of lagged edges suffers when over 20% of the measurements are missing, Ranking & Marsh 1985, c.f. Beltz & Gates 2017). If a value is missing, the whole row must be missing (i.e. across all ROIs). 

Mark missing timepoints as NA; do not manually omit them. Deleting them disrupts estimation of lagged effects.

Missing columns (i.e., ROIs) in a single dataset will cause an error. If one individual is missing one ROI, you will need to drop that individual or that ROI from the model.

.. _interpretation:

Interpretation of GIMME Results
---------------

For Group connections, a beta weight value exists for each individual. Thus, individual beta weights can be compared between groups or associated with other individual difference measures. Non-group / non-subgroup connections cannot be treated this way; unestimated individual connections cannot be replaced with zero. Specify Group connections *a priori* if you wish to analyze individual beta weights. Specifying a connection *a priori* forces its addition to the base model.

The presence or absence of individual connections can be compared, e.g. the number of inter-hemispheric connections in an individual.

Graph theoretical metrics can be applied, e.g. identifying hubs.

.. _resources:

Further Resources
-----------

gimme R package: https://cran.r-project.org/web/packages/gimme/index.html

gimme developer website: https://tarheels.live/gimme/


External Tutorials
~~~~~~~~~~~~~~~~

https://tarheels.live/gimme/tutorials/

Beltz, A. M., & Gates, K. M. (2017). Network mapping with GIMME. Multivariate behavioral research, 52(6), 789-804. [10.1080/00273171.2017.1373014](https://www.doi.org/10.1080/00273171.2017.1373014)

Lane, S. T., & Gates, K. M. (2017). Automated selection of robust individual-level structural equation models for time series data. Structural Equation Modeling: A Multidisciplinary Journal, 24(5), 768-782. [10.1080/10705511.2017.1309978](https://www.doi.org/10.1080/10705511.2017.1309978)

Algorithm Development
~~~~~~~~~~~~~~~~~

Gates, K. M., Fisher, Z. F., & Bollen, K. A. (2019). Latent variable GIMME using model implied instrumental variables (MIIVs). Psychological methods. [10.1037/met0000229](https://www.doi.org/10.1037/met0000229)

Henry, T. R., Feczko, E., Cordova, M., Earl, E., Williams, S., Nigg, J. T., … & Gates, K. M. (2019). Comparing directed functional connectivity between groups with confirmatory subgrouping GIMME. Neuroimage, 188, 642-653. [10.1016/j.neuroimage.2018.12.040](https://www.doi.org/10.1016/j.neuroimage.2018.12.040)

Gates, K. M., Lane, S. T., Varangis, E., Giovanello, K., & Guiskewicz, K. (2017). Unsupervised classification during time-series model building. Multivariate behavioral research, 52(2), 129-148. [10.1080/00273171.2016.1256187](https://www.doi.org/10.1080/00273171.2016.1256187)

Gates, K. M., & Molenaar, P. C. (2012). Group search algorithm recovers effective connectivity maps for individuals in homogeneous and heterogeneous samples. NeuroImage, 63(1), 310-319. [10.1016/j.neuroimage.2012.06.026](https://www.doi.org/10.1016/j.neuroimage.2012.06.026)

Applications
~~~~~~~~~~~~~~~

Gates, K. M., Molenaar, P. C., Hillary, F. G., & Slobounov, S. (2011). Extended unified SEM approach for modeling event-related fMRI data. NeuroImage, 54(2), 1151-1158. [10.1016/j.neuroimage.2010.08.051](https://www.doi.org/10.1016/j.neuroimage.2010.08.051)

Hillary, F. G., Medaglia, J. D., Gates, K. M., Molenaar, P. C., & Good, D. C. (2014). Examining network dynamics after traumatic brain injury using the extended unified SEM approach. Brain imaging and behavior, 8(3), 435-445. [10.1007/s11682-012-9205-0](https://www.doi.org/10.1007/s11682-012-9205-0)
