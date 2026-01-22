# Discriminant Analysis of Principal Components (DAPC)

## Background
DAPC is used to identify and describe genetic clusters. It shows differences between groups as best as possible while minimizing variation within the described clusters. 

## R script
### Libraries used:  
1) *vcfR* to read in the vcf file and convert it to a genlight object.
2) *adegenet* to execute the DAPC functions.

### Running the code
```
## Graphs to make these decisions are saved 
clusts <- find.clusters(genlight_obj, max.n.clust = 40, n.pca=38, n.clust=3)

## look at cluster assignments
names(clusts)
head(clusts$grp, 10)
clusts$grp


## number of pca and da were chosen based on graphs assessed outside of this run
dapc1 <- dapc(genlight_obj, clusts$grp, n.pca=4, n.da=2)
dapc1


## Make the scatter plot and manually save
scatter.dapc(dapc1, solid=1, pch=16, cex=1,
             col = c("#FFAD72", "#3FA0FF", "black"),
             posi.da = "topright",
             clabel = FALSE,
             cellipse = 0)
```


