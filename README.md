# two_stage_clustering
(Pseudo) Code for processing and running two stage clustering algorithm

### Step 1: Data processing  
- Remove non-wear and sleep time  
- Include adherent days (10 or more hours of awake wear time)  
- Include participants with 4 or more adherent days  
- Aggregate 15 second epochs to sedentary minutes/hour  
- Adjust start times based on awake times   
  
### Step 2: Day clusters  
- Restrict analysis to first 14 complete hours for each day  
- To cluster days with similar 14-hour pattern, run kmeans (kml fn. in R) on daily data (df):	  

```r
dat.ld <- clusterLongData(traj= df) ## create cluster object containing daily records  
kml(object=dat.ld, nbClusters=c(2,3,4,5,6,7), nbRedrawing=5). ## implement kmeans 
```         
- Use Calinski-Harabasz criteria to select the optimal # of clusters  
- Extract optimal day clusters       
  
```r
dat.cluster = data.frame(df["id"], clust = dat.ld@c4[[1]]@clusters) ## create data set (dat.cluster) with day clusters for each participant daily record  
```        
- DAY CLUSTER OUTPUT: A SB day-cluster assignment for each daily SB record
  
### Step 3: Participant sedentary patterns  
- For each participant, extract her day cluster assignments   
    - e.g.7-day clusters:,AAABBBB  
                                
```r
pt.cls = with(dat.cluster, table(id, clust)) ## count number of day clusters per participant
```

- Divide day cluster assignments by the number of available valid days for each participant
    - e.g., for a participant with record AAABBBB, the proportions are 
                                            3/7 for A; 4/7 for B; 0/7 for C and D
```r
pt.cls2 <- t(apply(pt.cls, 1, function(x) {x / sum(x)})) ## derive proportions
```

- Run hierarchical clustering (hclust fn in R) on proportion of days

```r
hc <- hclust(dist(pt.cls2), "complete") ## hierarchical clustering to cluster participants
```
- Use silhouette criteria to select the optimal # of clusters
- Extract participant sedentary patterns

```r
cutree(hc, k=4) ## extract patterns from hierarchical clustering
```
