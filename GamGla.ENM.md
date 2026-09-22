|             |                                                                                                                                                                                                   |
|:-----------:|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Author**: | Nina Casillas                                                                                                                                                                                     |
|  **Info**:  | Scripts from the manuscript "Climatic marginality and anthropogenic habitat change jointly shape extinction patterns in the threatened steppic species *Gampsocleis glabra* (Orthoptera: Ensifera)" |

# Table of Contents
<details>
<summary>🟥 Clean Occurrences</summary>  
&nbsp;&nbsp;&nbsp;🟧 C1 - Dedup, remove uncertain<br>
&nbsp;&nbsp;&nbsp;🟧 C2 - Move occs outside of soil rasters to nearest pixel<br>
&nbsp;&nbsp;&nbsp;🟧 C3 - Thinning<br>
</details>

<details>
<summary>🟥 Variables</summary>    
&nbsp;&nbsp;&nbsp;🟧 Cropping Vars  
</details>

<details>
<summary>🟥 KUENM</summary>
&nbsp;&nbsp;&nbsp;🟧 Var selection<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;🟨 CorrPlot<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;🟨 Var importance w/ KUENM<br>
&nbsp;&nbsp;&nbsp;🟧 M_variables<br>
&nbsp;&nbsp;&nbsp;🟧 Calibration & Evaluation<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;🟨 In GamGlaManu.Rmd<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;🟩 Calibration<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;🟩 Evaluation<br>
&nbsp;&nbsp;&nbsp;🟧 G_variables<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;🟨 Mid Holocene adjusted values<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;🟨 Crop sets<br>
&nbsp;&nbsp;&nbsp;🟧 Final Models<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;🟨 In GamGlaManu.Rmd<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;🟩 Create<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;🟩 Evaluate<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;🟩 MOP<br>
&nbsp;&nbsp;&nbsp;🟧 MOP Binary & Cleanup<br>
&nbsp;&nbsp;&nbsp;🟧 Final Thresholded & Mop clipped Model<br>
</details>

<details>
<summary>🟥 Ellipsoid envelope model with ellipsenm</summary>
&nbsp;&nbsp;&nbsp;🟧 Crop vars<br>
&nbsp;&nbsp;&nbsp;🟧 Ellipsenm<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;🟨 Occurrences<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;🟨 Calibration<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;🟨 Model<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;🟨 Projections<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;🟨 MOP<br>
&nbsp;&nbsp;&nbsp;🟧 Threshold suitability & CPH<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;🟨 Plot CPH<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;🟨 Fisher exact test<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;🟨 Mann-whitney U<br>
&nbsp;&nbsp;&nbsp;🟧 Final model<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;🟨 Suitability MOP binary & cleanup<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;🟨 Final model<br>
&nbsp;&nbsp;&nbsp;🟧 Mahalanobis dist<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;🟨 Threshold<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;🟨 Histogram<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;🟨 Violin plot<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;🟨 Normality test<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;🟨 Mann-Whitney u<br>
</details>

<details>
<summary>🟥 Land Use</summary>
&nbsp;&nbsp;&nbsp;🟧 Buffer groups and crop land use variables<br>
&nbsp;&nbsp;&nbsp;🟧 Linearity<br>
&nbsp;&nbsp;&nbsp;🟧 Normality w/ qqplot<br>
&nbsp;&nbsp;&nbsp;🟧 Non parametric McSweeny and Porter ANCOVA<br>
</details>
🟥 Maxent x Ellipsoid Envelop model  

***

# 🟥 Clean Occurrences
## 🟧 C1 - Dedup, remove uncertain
```r
# reading the occurrences Total = 704
Occs <- read.csv("D:/ENM/GamGla/Occs/Allunclean.csv")
colnames(Occs)
# ordering occs to prioritize records with status
Occs <- Occs[order(Occs$Source, (Occs$Source=='Open Source')),]
Occs$status <- factor(Occs$status, levels = c("extinct", "extant"))
Occs <- Occs[order(Occs$status),]
# save and read in the ordered data
write.csv(Occs, "D:/ENM/GamGla/Occs/Allunclean_ordered.csv", row.names = F)
Occs <- read.csv("D:/ENM/GamGla/Occs/Allunclean_ordered.csv")
colnames(Occs)
# remove duplicates = 686 unique
Occs_rem <- Occs[duplicated(Occs$Coordinate.latlon), ]
Occs <- Occs[!duplicated(Occs$Coordinate.latlon), ]
# remove occs with uncertainty more than 10 km
Occs_rem <- rbind(Occs_rem, Occs[which(Occs$CoordinateUncertaintyInMeters > 10000), ])
Occs <- Occs[-which(Occs$CoordinateUncertaintyInMeters > 10000), ]
# writing cleaned occs C1 = 677
dir.create("D:/ENM/GamGla/Occs/Manu")
write.csv(Occs, "D:/ENM/GamGla/Occs/Manu/Occs_C1.csv", row.names = F)
```

## 🟧 C2 - Move occs outside of soil rasters to nearest pixel
```r
setwd("D:/ENM/GamGla/Manu")
# reading the occurrences Total = 677
occs <- read.csv("D:/ENM/GamGla/Occs/Manu/Occs_C1.csv")
# stack vars
library(raster)
library(terra)
library(rgdal)
library(kuenm)
var_list <- raster::stack(list.files("D:/ENM/Variables/Soil/agg1000m_reprojqgis.2.5m", pattern = ".tif",
                                     full.names = TRUE))
rvars <- terra::rast(var_list)

# buffer for temp M_variables
b_50 <- ellipsenm::buffer_area(occs, longitude = "decimalLongitude", latitude = "decimalLatitude", buffer_distance = 50)
plot(b_50)
writeOGR(obj=b_50, dsn="b_50", layer="b_50", driver="ESRI Shapefile", overwrite_layer = T)
# reading as a spatvector
buff_shape <- terra::vect("b_50/b_50.shp")
# checking the crs is correct for terra::crop
terra::crs(buff_shape)
terra::crs(rvars)
terra::same.crs(buff_shape, rvars)

#'[ Crop ]
buff_cmrvars <- terra::crop(rvars, buff_shape, mask = TRUE)
buff_brcmrvars <- brick(buff_cmrvars)
plot(buff_cmrvars[[1]])
# write
dir.create('b_50_varcropped')
rnames <- paste0("b_50_varcropped/", names(buff_brcmrvars), ".tif")
# save files
sav <- lapply(1:nlayers(buff_brcmrvars), function(x) {
    raster::writeRaster(buff_brcmrvars[[x]], filename = rnames[x], format = "GTiff")})

# Correct occs in holes
# raster
var <- raster("b_50_varcropped/phh2o.tif")
plot(var)
# 10km limit for coordinate uncertainty -> 10 km limit.distance
# 3 occurrences moved
occs1 <- kuenm::kuenm_toclosest(occs, longitude = "decimalLongitude", latitude = "decimalLatitude",
                                raster.layer = var, limit.distance = 10)
# plot to visualize the difference
# save plot
jpeg("moved_occs.jpeg", width = 1080, height = 400, units='px', quality = 85)
raster::plot(rvars[[1]], ext = var)
movedoccs1 <- subset(occs1, occs1$condition == "Moved")
terra::points(movedoccs1[, 3:4], col = "red", cex = 2, pch = 3)
terra::points(movedoccs1[, 19:20], col = "blue", cex = 2, pch = 3)
dev.off()

# Test for lost occs
cropshape <- raster("D:/ENM/Variables/Soil/agg1000m_reprojqgis.2.5m/phh2o.tif")
v <- extract(cropshape, occs1[,3:4])
# extracting values from the raster
dt <- cbind(occs1, v)
# keeping rows with coordinates intersecting with soil
dt <- dt[!is.na(dt$v),]
# remove tmp column
dt <- subset(dt, select=-c(v))
# save C2 = 677
write.csv(dt, "D:/ENM/GamGla/Occs/Manu/Occs_C2.csv", row.names = F)
```

## 🟧 C3 - Thinning

>[!NOTE]
> Run `ellipsenm::thin_data()` in R v.4.3.1, then continue in R v.4.5.2

```r
# reading occs
Occs_thin <- read.csv("D:/ENM/GamGla/Occs/Manu/Occs_C2.csv")
Occs_thin <- ellipsenm::thin_data(Occs_thin, longitude = "decimalLongitude", latitude = "decimalLatitude", thin_distance =3)
# writing it to csv C3 = 365
# 312 were removed
write.csv(Occs_thin, "D:/ENM/GamGla/Occs/Manu/Occs_C3.csv", row.names = F)
```

# 🟥 Variables
## 🟧 Cropping Vars
```r
setwd("D:/ENM/GamGla/Manu")
species <- "Gampsocleis glabra"

var_list <- raster::stack(list.files("D:/ENM/Variables/Climatic/wc2.1_2.5m", pattern = ".tif",
                                     full.names = TRUE))
soils <- raster::stack(list.files("D:/ENM/Variables/Soil/agg1000m_reprojqgis.2.5m", pattern = ".tif",
                                  full.names = TRUE))
var_list <- raster::stack(var_list, soils)
rvars <- rast(var_list)

#'[ BUFFER SHAPEFILE]
b <- read.csv('../Occs/Manu/Occs_C3.csv')
names(b)
b$species <- species
b_50 <- ellipsenm::buffer_area(b, longitude = "decimalLongitude", latitude = "decimalLatitude", buffer_distance = 50)
plot(b_50)
writeOGR(obj=b_50, dsn="b_50", layer="b_50", driver="ESRI Shapefile", overwrite_layer = T)
# read as spatvector
buff_shape <- terra::vect("b_50/b_50.shp")
# check crs for terra::crop
terra::crs(buff_shape)
terra::crs(rvars)
terra::same.crs(buff_shape, rvars)

#'[ Crop ]
buff_cmrvars <- terra::crop(rvars, buff_shape, mask = TRUE)
buff_brcmrvars <- brick(buff_cmrvars)
plot(buff_cmrvars[[1]])
# prep save
dir.create('D:/ENM/GamGla/Distribution/buffer_cut_vars')
rnames <- paste0("D:/ENM/GamGla/Distribution/buffer_cut_vars/", names(buff_brcmrvars), ".tif")
# save
sav <- lapply(1:nlayers(buff_brcmrvars), function(x) {
    raster::writeRaster(buff_brcmrvars[[x]], filename = rnames[x], format = "GTiff")})
```

# 🟥 KUENM
```r
setwd("D:/ENM/GamGla/Manu")

#'[ Model Calibration]
library(ellipsenm)
library(raster)
library(rgeos)
library(rgdal)
library(viridis)

species <- "Gampsocleis glabra"
occs <- read.csv('D:/ENM/GamGla/Occs/Manu/Occs_C3.csv')
colnames(occs)

# split data
occs$species <- species
occs$longitude <- occs$decimalLongitude
occs$latitude <- occs$decimalLatitude
occs <- subset(occs, select = c("species", "longitude", "latitude"))

occs$check <- paste(occs[,2], occs[,3], sep = "_")
train <- occs[sample(nrow(occs), round((length(occs[,1])/4 *3))), ] # 75-25 split, train-test
test <- occs[!occs[,4] %in% train[,4], ]
occs$check <- NULL
train$check <- NULL
test$check <- NULL

write.csv(occs, paste0(species, "_joint.csv"), row.names = FALSE)
write.csv(train, paste0(species, "_train.csv"), row.names = FALSE)
write.csv(test, paste0(species, "_test.csv"), row.names = FALSE)

# independent
independent <- read.csv("D:/ENM/GamGla/Occs/independent/independent.csv")
head(independent)
independent$species <- species
independent$longitude <- independent$Longitude
independent$latitude <- independent$Latitude
independent <- subset(independent, select = c("species", "longitude", "latitude"))
head(independent)
write.csv(independent, paste0(species, "_ind.csv"), row.names = FALSE)
```

## 🟧 Var selection
```r
library(kuenm)

# reading rasters callibration area
vars <- stack(list.files("../Distribution/buffer_cut_vars/", pattern = ".tif",
                         full.names = TRUE))
crs(vars) <- '+proj=longlat +datum=WGS84 +no_defs'

reg_mult <- c(seq(0.1, 1, 0.1), 1)
maxent_path = "C:/maxent"
plot(vars[[1]])
var_explore <- explore_var_contrib(occs, M_variables = vars, maxent.path = maxent_path, reg.mult = reg_mult,
                                   plot=TRUE, verbose = FALSE)

var_res <-  var_explore[["Contribution"]]
var_res$Permutation.Importance <- (as.data.frame(var_explore[["Permutation_importance"]]))$Permutation_importance
var_res$jacknife.Training.With <- (as.data.frame(var_explore[["Jackknife_results"]][["Training_gain_with_without"]]))$Training_gain_with
var_res$jacknife.Training.Without <- (as.data.frame(var_explore[["Jackknife_results"]][["Training_gain_with_without"]]))$Training_gain_without
# save results
write.csv(var_res, "Var_importance_All.csv", row.names = F, quote = F)

# save plot
jpeg("plots/Variable_Contribution.jpeg", width = 600, units='px', quality = 100)
plot_contribution(contribution_list = var_explore, col.cont = "steelblue", col.imp = "lightblue",col.with = "navy",
                  col.without = "darkgrey", col.all = "navy")
dev.off()
```
### 🟨 CorrPlot
```r
cors <- variable_correlation(vars, correlation_limit = 0.75,
                             corrplot = F, magnify_to = 1.5)
# creating a csv of variable correlation
write.csv(cors, "Data/Vars_correlation_matrix.csv")

library(corrplot)
corrplot::corrplot(cors, method = c("square"), type = c("full"), is.corr = TRUE, bg = "white",
                   title = "", diag = TRUE, mar = c(0, 0, 0, 0), addgrid.col = NULL,
                   addCoef.col = NULL, addCoefasPercent = FALSE, order = c("hclust"),
                   hclust.method = c("complete"), addrect = 5)
# take one from each box to make sets. Keep in mind bio of animal
```

### 🟨 Var importance w/ KUENM
```r
library(kuenm)

# results table
var_res <- data.frame()

# First selection
    s1 <- vars[[c("cfvo", "wc2.1_2.5m_bio_14", "nitrogen",  "phh2o", "sand")]]
    s2 <- vars[[c("clay", "wc2.1_2.5m_bio_17", "ocd", "wc2.1_2.5m_bio_10",	"wc2.1_2.5m_bio_15")]]
    s3 <- vars[[c("silt", "wc2.1_2.5m_bio_12", "ocs", "wc2.1_2.5m_bio_5", "wc2.1_2.5m_bio_2")]]
    s4 <- vars[[c("cfvo", "wc2.1_2.5m_bio_13", "soc", "phh2o", "wc2.1_2.5m_bio_4")]]
    s5 <- vars[[c("clay", "wc2.1_2.5m_bio_16", "cec", "wc2.1_2.5m_bio_10", "wc2.1_2.5m_bio_7")]]
    s6 <- vars[[c("silt", "bdod", "wv0033", "wc2.1_2.5m_bio_5", "sand")]]
    s7 <- vars[[c("cfvo", "wc2.1_2.5m_bio_3", "nitrogen", "phh2o", 	"wc2.1_2.5m_bio_15")]]
    s8 <- vars[[c("clay", "wc2.1_2.5m_bio_1", "ocd", "wc2.1_2.5m_bio_10", "wc2.1_2.5m_bio_2")]]
    s9 <- vars[[c("silt", "wc2.1_2.5m_bio_11", "ocs", "wc2.1_2.5m_bio_5",  "wc2.1_2.5m_bio_4")]]
    s10 <- vars[[c("cfvo", "wc2.1_2.5m_bio_6", "soc", "phh2o", "wc2.1_2.5m_bio_7")]]
    
set_list <- list(s1 = s1, s2 = s2, s3 = s3, s4 = s4, s5 = s5, s6 = s6, s7 = s7, s8 = s8, s9 = s9, s10 = s10)

for (sn in names(set_list)) {
    s <- set_list[[sn]]
    ex <- explore_var_contrib(occs, M_variables = s, maxent.path = maxent_path, reg.mult = reg_mult,
                                      plot = FALSE, verbose = TRUE)
    vs <-  ex[["Contribution"]]
    vs$Permutation.Importance <- (as.data.frame(ex[["Permutation_importance"]]))$Permutation_importance
    vs$jacknife.Training.With <- (as.data.frame(ex[["Jackknife_results"]][["Training_gain_with_without"]]))$Training_gain_with
    vs$jacknife.Training.Without <- (as.data.frame(ex[["Jackknife_results"]][["Training_gain_with_without"]]))$Training_gain_without
    vs$Set <- sn
    var_res <- rbind(var_res, vs)
    
    # save plot
    jpeg(paste0("plots/Variable_Contribution_", sn, ".jpeg"), height = 300, width = 600, units='px', quality = 90)
    plot_contribution(contribution_list = ex, col.cont = "steelblue", col.imp = "lightblue",col.with = "navy",
                      col.without = "darkgrey", col.all = "navy")
    dev.off()
    }

# Second selection of sets
    ss1 <- vars[[c("silt", "wc2.1_2.5m_bio_14", "wv0033", "wc2.1_2.5m_bio_10", "wc2.1_2.5m_bio_4")]]
    ss2 <- vars[[c("silt", "wc2.1_2.5m_bio_3", "ocs", "phh2o", "wc2.1_2.5m_bio_7")]]
    ss3 <- vars[[c("clay", "wc2.1_2.5m_bio_12", "ocs", "wc2.1_2.5m_bio_5", "wc2.1_2.5m_bio_4")]]
    ss4 <- vars[[c("clay", "wc2.1_2.5m_bio_17", "wv0033", "wc2.1_2.5m_bio_5", "wc2.1_2.5m_bio_7")]]
    ss5 <- vars[[c("silt", "wc2.1_2.5m_bio_1", "ocs", "phh2o","wc2.1_2.5m_bio_4")]]
    ss6 <- vars[[c("silt", "wc2.1_2.5m_bio_3", "ocs", "wc2.1_2.5m_bio_10", "wc2.1_2.5m_bio_15")]]
    ss7 <- vars[[c("wc2.1_2.5m_bio_3", "wc2.1_2.5m_bio_10", "wc2.1_2.5m_bio_15")]]
    ss8 <- vars[[c("wc2.1_2.5m_bio_3", "wc2.1_2.5m_bio_5", "wc2.1_2.5m_bio_15")]]
    ss9 <- vars[[c("wc2.1_2.5m_bio_14", "wc2.1_2.5m_bio_10", "wc2.1_2.5m_bio_4")]]

sset_list <- list(ss1 = ss1, ss2 = ss2, ss3 = ss3, ss4 = ss4, ss5 = ss5, ss6 = ss6, ss7 = ss7, ss8 = ss8, ss9 = ss9)

for (ssn in names(sset_list)) {
    ss <- sset_list[[ssn]]
    ex <- explore_var_contrib(occs, M_variables = ss, maxent.path = maxent_path, reg.mult = reg_mult,
                                      plot = FALSE, verbose = TRUE)
    vs <-  ex[["Contribution"]]
    vs$Permutation.Importance <- (as.data.frame(ex[["Permutation_importance"]]))$Permutation_importance
    vs$jacknife.Training.With <- (as.data.frame(ex[["Jackknife_results"]][["Training_gain_with_without"]]))$Training_gain_with
    vs$jacknife.Training.Without <- (as.data.frame(ex[["Jackknife_results"]][["Training_gain_with_without"]]))$Training_gain_without
    vs$Set <- ssn
    var_res <- rbind(var_res, vs)
    
    # save plot
    jpeg(paste0("plots/Variable_Contribution_", ssn, ".jpeg"), height = 300, width = 600, units='px', quality = 90)
    plot_contribution(contribution_list = ex, col.cont = "steelblue", col.imp = "lightblue",col.with = "navy",
                      col.without = "darkgrey", col.all = "navy")
    dev.off()
    }
```

## 🟧 M_variables
```r
library(kuenm)
library(terra)

    vars <- raster::stack(list.files("D:/ENM/GamGla/Distribution/buffer_cut_vars/", pattern = ".tif",
                                     full.names = TRUE))
    crs(vars) <- '+proj=longlat +datum=WGS84 +no_defs'
    rvars <- rast(vars)
    
    # read shapefile as spatvector
    buff_shape <- terra::vect("b_50/b_50.shp")
    #'[ Crop Current]
    buff_cmrvars <- terra::crop(rvars, buff_shape, mask = TRUE)
    buff_brcmrvars <- brick(buff_cmrvars)
    
    # split sets
    ss1 <- buff_brcmrvars[[c("silt", "wc2.1_2.5m_bio_14", "wv0033", "wc2.1_2.5m_bio_10", "wc2.1_2.5m_bio_4")]]
    ss2 <- buff_brcmrvars[[c("silt", "wc2.1_2.5m_bio_3", "ocs", "phh2o", "wc2.1_2.5m_bio_7")]]
    ss3 <- buff_brcmrvars[[c("clay", "wc2.1_2.5m_bio_12", "ocs", "wc2.1_2.5m_bio_5", "wc2.1_2.5m_bio_4")]]
    ss4 <- buff_brcmrvars[[c("clay", "wc2.1_2.5m_bio_17", "wv0033", "wc2.1_2.5m_bio_5", "wc2.1_2.5m_bio_7")]]
    ss5 <- buff_brcmrvars[[c("silt", "wc2.1_2.5m_bio_1", "ocs", "phh2o","wc2.1_2.5m_bio_4")]]
    ss6 <- buff_brcmrvars[[c("silt", "wc2.1_2.5m_bio_3", "ocs", "wc2.1_2.5m_bio_10", "wc2.1_2.5m_bio_15")]]
    ss7 <- buff_brcmrvars[[c("wc2.1_2.5m_bio_3", "wc2.1_2.5m_bio_10", "wc2.1_2.5m_bio_15")]]
    ss8 <- buff_brcmrvars[[c("wc2.1_2.5m_bio_3", "wc2.1_2.5m_bio_5", "wc2.1_2.5m_bio_15")]]
    ss9 <- buff_brcmrvars[[c("wc2.1_2.5m_bio_14", "wc2.1_2.5m_bio_10", "wc2.1_2.5m_bio_4")]]
    
    csset_list <- list(ss1 = ss1, ss2 = ss2, ss3 = ss3, ss4 = ss4, ss5 = ss5, ss6 = ss6, ss7 = ss7, ss8 = ss8, ss9 = ss9)
    
    # write to M_variables
    dir.create("M_variables")
    
    for (cssn in names(csset_list)) {
      num <- sub("ss", "", cssn)
      css <- csset_list[[cssn]]
      dir.create(paste0("M_variables/", "Set_", num))
      cssnames <- paste0("M_variables/", "Set_", num, "/", names(css), ".tif")
      sav <- lapply(1:nlayers(css), function(x) {
        raster::writeRaster(css[[x]], filename = cssnames[x], format = "GTiff")})
    }
```

## 🟧 Calibration & Evaluation
```r
setwd("D:/ENM/GamGla/Manu")
library(kuenm)
# Loading the file & rename whatever
species <- "Gampsocleis glabra"
file_name <- "GamGlaManu"
kuenm_start(file.name = file_name)
```
### 🟨 In GamGlaManu.Rmd

> #### 🟩 Calibration
>```r
>species <- "Gampsocleis glabra"
>occ_joint <- paste0(species, "_joint.csv")
>occ_tra <- paste0(species, "_train.csv")
>M_var_dir <- "M_variables"
>batch_cal <- "Candidate_models"
>out_dir <- "Candidate_Models"
>reg_mult <- c(seq(0.1, 1, 0.1))
>f_clas <- "all"
>args <- NULL
>maxent_path <- "C:/maxent"
>wait <- FALSE
>run <- TRUE
>
>kuenm_cal(occ.joint = occ_joint, occ.tra = occ_tra, M.var.dir = M_var_dir, 
>          batch = batch_cal, out.dir = out_dir, reg.mult = reg_mult, 
>          f.clas = f_clas, args = args, maxent.path = maxent_path, 
>          wait = wait, run = run)
>
>```
>
> #### 🟩 Evaluation
>```r
>occ_test <- paste0(species, "_test.csv")
>out_eval <- "Calibration_results"
>threshold <- 5
>rand_percent <- 50
>iterations <- 500
>kept <- TRUE
>selection <- "OR_AICc"
>
>cal_eval <- kuenm_ceval(path = out_dir, occ.joint = occ_joint, occ.tra = occ_tra, 
>                        occ.test = occ_test, batch = batch_cal, out.eval = out_eval, 
>                        threshold = threshold, rand.percent = rand_percent, 
>                        iterations = iterations, kept = kept, selection = selection)
>```

## 🟧 G_variables
### 🟨 Mid Holocene adjusted values
```r
#### 🟩 Correct temp scale
library(gtools)

# mid Holocene GCMs
mh_gcms <- list("MIROC-ESM.mrmidbi_2-5m", "BCC-CSM1-1.bcmidbi_2-5m")

for (gcm in mh_gcms) {
    vars <- terra::rast(gtools::mixedsort(list.files(paste0("D:/ENM/Variables/Climatic/", gcm, "/"), pattern = ".tif", full.names = TRUE)))
    vars_temp <- vars[[1:11]] # only the temp vars are scaled * 10
    vars_prec <- vars[[12:19]] # precipitation vars
    # scale down the temp vars
    vars_temp <- vars_temp/10
    # checking scale
    vars[[1]]
    vars_temp[[1]] # seems right
    # combine again
    vars_fixed <- c(vars_temp, vars_prec)
    names(vars_fixed)
    # save files
    dir.create(paste0("D:/ENM/Variables/Climatic/", gcm, "/corrected.values"))
    terra::writeRaster(vars_fixed, filename = file.path(paste0("D:/ENM/Variables/Climatic/", gcm, "/corrected.values/", names(vars_fixed), ".tif")))
}
```
### 🟨 Crop sets ####
```r
  library(terra)
  library(raster)
  
  # buffer
  buff_shape <- terra::vect("D:/ENM/GamGla/Distribution/Restricted_accessible/Restricted_accessible.shp")
  
  gcms <- c("current", "MIROC-ESM.mrmidbi_2-5m", "BCC-CSM1-1.bcmidbi_2-5m")
  vpaths <- list("current" = "D:/ENM/Variables/Climatic/wc2.1_2.5m",
                 "MIROC-ESM.mrmidbi_2-5m" = "D:/ENM/Variables/Climatic/MIROC-ESM.mrmidbi_2-5m/corrected.values",
                 "BCC-CSM1-1.bcmidbi_2-5m" = "D:/ENM/Variables/Climatic/BCC-CSM1-1.bcmidbi_2-5m/corrected.values")
  
  vtags <- list("current" = "wc2.1_2.5m_bio_",
                "MIROC-ESM.mrmidbi_2-5m" = "mrmidbi",
                "BCC-CSM1-1.bcmidbi_2-5m" = "bcmidbi")

  for(gcm in gcms) {
    vpath <- vpaths[[gcm]]
    vtag <- vtags[[gcm]]
    print(gcm)
    print(vpath)
    print(vtag)
    
    var_list <- raster::stack(list.files(vpath, pattern = ".tif",
                                         full.names = TRUE))
    soils <- raster::stack(list.files("D:/ENM/Variables/Soil/agg1000m_reprojqgis.2.5m", pattern = ".tif",
                                      full.names = TRUE))
    print(ext(var_list)) ; print(ext(soils))
    # crop
    soils <- terra::crop(soils, var_list)
    var_list <- raster::stack(var_list, soils)
    terra::crs(var_list) <- '+proj=longlat +datum=WGS84 +no_defs'
    rvars <- terra::rast(var_list)
    # standardize names
    print(names(rvars))
    names(rvars) <- gsub(vtag, "wc2.1_2.5m_bio_", names(rvars))
    print(names(rvars))
    
    # split sets
    ss2 <- rvars[[c("silt", "wc2.1_2.5m_bio_3", "ocs", "phh2o", "wc2.1_2.5m_bio_7")]]
    ss5 <- rvars[[c("silt", "wc2.1_2.5m_bio_1", "ocs", "phh2o","wc2.1_2.5m_bio_4")]]
    ss7 <- rvars[[c("wc2.1_2.5m_bio_3", "wc2.1_2.5m_bio_10", "wc2.1_2.5m_bio_15")]]
    
    ssgsets <- list(ss2 = ss2, ss5 = ss5, ss7 = ss7)
    
    for(ssgsetn in names(ssgsets)) {
      ssg <- ssgsets[[ssgsetn]]
      num <- sub("ss", "", ssgsetn)
      
      buff_cmrvars <- terra::crop(ssg, buff_shape, mask = TRUE)
      buff_brcmrvars <- brick(buff_cmrvars)
      plot(buff_brcmrvars)
      # write to G_variables
      dir.create(paste0("G_variables/", "Set_", num, "/", gcm), recursive = TRUE)
      ssgnames <- paste0("G_variables/", "Set_", num, "/", gcm, "/", names(ssg), ".tif")
      # save files
      sav <- lapply(1:nlayers(buff_brcmrvars), function(x) {
        raster::writeRaster(buff_brcmrvars[[x]], filename = ssgnames[x], format = "GTiff")})
    }
  }
```

## 🟧 Final Models
### 🟨 In GamGlaManu.Rmd
> #### 🟩 Create
>```r
>batch_fin <- "Final_Models/"
>mod_dir <- "Final_Models/"
>rep_n <- 10
>rep_type <- "Bootstrap"
>jackknife <- FALSE
>out_format <- "logistic"
>project <- TRUE
>G_var_dir <- "G_variables2"
>ext_type <- "all"
>write_mess <- TRUE
>write_clamp <- TRUE
>wait1 <- TRUE
>run1 <- TRUE
>args <- NULL
>
>kuenm_mod(occ.joint = occ_joint, M.var.dir = M_var_dir, out.eval = out_eval, 
>          batch = batch_fin, rep.n = rep_n, rep.type = rep_type, 
>          jackknife = jackknife, out.dir = mod_dir, out.format = out_format, 
>          project = project, G.var.dir = G_var_dir, ext.type = ext_type,
>          write.mess = write_mess, write.clamp = write_clamp, 
>          maxent.path = maxent_path, args = args, wait = wait1, run = run1)
>```
>
> #### 🟩 Evaluate
>```r
>occ_ind <- paste0(species, "_ind.csv")
>replicates <- TRUE
>out_feval <- "Final_Models_evaluation"
>
>fin_eval <- kuenm_feval(path = mod_dir, occ.joint = occ_joint, occ.ind = occ_ind, 
>                        replicates = replicates, out.eval = out_feval, 
>                        threshold = threshold, rand.percent = rand_percent,
>                        iterations = iterations)
>```
>#### 🟩 MOP
>```r
>is_swd <- FALSE
>sets_var <- c("Set_2", "Set_5")
>G_var_dir <- "G_variables2"
>M_var_dir = "M_variables"
>out_mop <- "MOP_res"
>percent <- 10
>paral <- TRUE
>
>kuenm_mmop(G.var.dir = G_var_dir, M.var.dir = M_var_dir, is.swd = is_swd,
>           sets.var = sets_var, out.mop = out_mop, percent = percent, 
>           parallel = paral)
>```

## 🟧 MOP Binary & Cleanup
```r
setwd("D:/ENM/GamGla/Manu")

#'[Binary mop results and multiply by the final thresholded model.] #
  library(kuenm)
  library(raster)
  gcms <- c("current", "MIROC-ESM.mrmidbi_2-5m", "BCC-CSM1-1.bcmidbi_2-5m")
  
for(gcm in gcms) {
  # load thresholded model
  Thresh_Mod <- raster(paste0("Binary/Binary_5percent_", gcm, "_M_1_F_lh_Set_5_EC.tif"))
  # Binary MOP
  MOP <- raster(paste0("MOP_res/Set_5/MOP_10%_", gcm, ".tif"))
  MOP <- MOP>0 
  # Binary Mop * thresholded final model
  fm <- MOP * Thresh_Mod
  plot(MOP, main = paste0(gcm, "mop"))
  plot(Thresh_Mod, main = paste0(gcm, " thresholded model"))
  plot(fm, main  = paste0(gcm, " final model"))
  # save raster
  raster::writeRaster(fm, filename = paste0("MOP_res/MOP10%_Thresh5%_Set5_", gcm, "_Binary.tif"), format = "GTiff", overwrite=T)
  # save plots
  library(devtools)
  
  # mop
  jpeg(paste0("plots/MOP10_Set5_", gcm, "_Binary.jpeg"), width = 1800, height = 1500, units='px', quality = 100)
  plot(MOP, main = paste0(gcm, " mop"), box=F, axes=F, col=c("lightgrey", "#636060"))
  dev.off()
  # thresholded model
  jpeg(paste0("plots/Thresh5_Set5_", gcm, "_Binary.jpeg"), width = 1800, height = 1500, units='px', quality = 100)
  plot(Thresh_Mod, main = paste0(gcm, " thresholded model"), box=F, axes=F, col=c("lightgrey", "#636060"))
  dev.off()
  # final model
  jpeg(paste0("plots/Final_MOP10_Thresh5_Set5_", gcm, "_Binary.jpeg"), width = 1800, height = 1500, units='px', quality = 100)
  plot(fm, main  = paste0(gcm, " final model"), box=F, axes=F, col=c("lightgrey", "#636060"))
  dev.off()
}  
```

## 🟧 Final Thresholded & Mop clipped Model
```r
setwd("D:/ENM/GamGla/Manu")
dir.create("Final_Model_Clipped")
gcms <- c("current", "MIROC-ESM.mrmidbi_2-5m", "BCC-CSM1-1.bcmidbi_2-5m")

for(gcm in gcms) {
m10.th5 <- terra::rast(paste0("MOP_res/MOP10%_Thresh5%_Set5_", gcm, "_Binary.tif")) # mop & thresholded binary
mod <- terra::rast(paste0("Final_Models/M_1_F_lh_Set_5_EC/Gampsocleis_glabra_", gcm, "_median.asc")) # final model proj to current
fm <- ifel(m10.th5 == 0, NA, mod) # final model, masked with unsuitable & extrapolative areas
countries <- rnaturalearth::ne_countries(scale = "small", returnclass = "sv") # country outline
plot(m10.th5)
plot(mod)
plot(fm)
# save raster
raster::writeRaster(fm, filename = paste0("Final_Model_Clipped/MOP10%_Thresh5%_Set5_", gcm, "_Binary.tif"), overwrite=T)
}
```

# 🟥 Ellipsoid envelope model with ellipsenm
## 🟧 Crop vars
```r
dir.create('D:/ENM/GamGla/Manu/ellipsenm')
setwd('D:/ENM/GamGla/Manu/ellipsenm')

library(terra)
library(kuenm)
library(raster)

# buffer
buff_shape <- terra::vect("D:/ENM/GamGla/Distribution/Restricted_accessible/Restricted_accessible.shp")

gcms <- c("current", "MIROC-ESM.mrmidbi_2-5m", "BCC-CSM1-1.bcmidbi_2-5m")
vpaths <- list("current" = "D:/ENM/Variables/Climatic/wc2.1_2.5m",
               "MIROC-ESM.mrmidbi_2-5m" = "D:/ENM/Variables/Climatic/MIROC-ESM.mrmidbi_2-5m",
               "BCC-CSM1-1.bcmidbi_2-5m" = "D:/ENM/Variables/Climatic/BCC-CSM1-1.bcmidbi_2-5m")

vtags <- list("current" = "wc2.1_2.5m_bio_",
              "MIROC-ESM.mrmidbi_2-5m" = "mrmidbi",
              "BCC-CSM1-1.bcmidbi_2-5m" = "bcmidbi")

outpcas <- list("current" = "G_variables/current",
               "MIROC-ESM.mrmidbi_2-5m" = "G_variables/MIROC-ESM.mrmidbi_2-5m",
               "BCC-CSM1-1.bcmidbi_2-5m" = "G_variables/BCC-CSM1-1.bcmidbi_2-5m")
for(gcm in gcms) {
  vpath <- vpaths[[gcm]]
  vtag <- vtags[[gcm]]
  outpca <- outpcas[[gcm]]
  print(paste0("Starting ", gcm, "  __________"))
  print(paste0("variable path: ", vpath))
  print(paste0("variable naming scheme: ", vtag))
  
  var_list <- raster::stack(list.files(vpath, pattern = ".tif",
                                       full.names = TRUE))
  crs(var_list) <- '+proj=longlat +datum=WGS84 +no_defs'
  rvars <- terra::rast(var_list)
  
  # standardize names
  print(paste0("before renaming: "))
  print(names(rvars))
  names(rvars) <- gsub(vtag, "wc2.1_2.5m_bio_", names(rvars))
  print(paste0("after renaming: "))
  print(names(rvars))
  
  # crop
  buff_cmrvars <- terra::crop(rvars, buff_shape, mask = TRUE)
  buff_brcmrvars <- brick(buff_cmrvars)
  names(buff_brcmrvars)
  plot(buff_brcmrvars[[1:2]])
  
  # write
  dir.create(paste0('Restricted_accessible/', gcm), recursive = TRUE)
  names <- paste0("Restricted_accessible/", gcm, "/", names(buff_brcmrvars), ".asc")
  # save files
  sav <- lapply(1:nlayers(buff_brcmrvars), function(x) {
    raster::writeRaster(buff_brcmrvars[[x]], filename = names[x], format = "ascii", overwrite = TRUE)})
  
  #'[PCA]
  variables <- paste0("Restricted_accessible/", gcm)
  in.format <- "ascii"
  var.scale <- TRUE
  write.result <- TRUE
  out.format <- "GTiff"
  dir.create(outpca, recursive = T)
  out.dir <- outpca
  
  pcs <- kuenm_rpca(variables = variables, in.format = in.format,
                    var.scale = var.scale, write.result = write.result,
                    out.format = out.format, out.dir = out.dir)
  
  # check out the results
  PCA_results <- read.table(pipe(paste0("sed -n '/PCA summary/,+4p' ", outpca, "/Initial/pca_results.txt")), 
                            header = FALSE, sep = "\t")
  print(paste0(gcm, " PCA Results __________"))
  print(head(PCA_results))
  
  if (gcm == "current") { 
  # M_variables
  var_list <- raster::stack(list.files(paste0(outpca, "/Initial"), pattern = ".tif",
                                       full.names = TRUE))
  crs(var_list) <- '+proj=longlat +datum=WGS84 +no_defs'
  names(rvars)
  
  # buffer
  buff_shapeM <- terra::vect("D:/ENM/GamGla/Manu/b_50/b_50.shp")
  # crop
  buff_cmrvars <- terra::crop(rvars, buff_shapeM, mask = TRUE)
  buff_brcmrvars <- brick(buff_cmrvars)
  names(buff_brcmrvars)
  plot(buff_brcmrvars[[1:2]])
  # write
  dir.create('M_variables', recursive = TRUE)
  names <- paste0("M_variables/", names(buff_brcmrvars), ".asc")
  # save files
  sav <- lapply(1:nlayers(buff_brcmrvars), function(x) {
    raster::writeRaster(buff_brcmrvars[[x]], filename = names[x], format = "ascii", overwrite = TRUE)})
  }
}
```

## 🟧 Ellipsenm ####
### 🟨 Occurrences ####
```r
library(ellipsenm)
library(raster)
library(viridis)

species <- "Gampsocleis glabra"
occs <- read.csv('D:/ENM/GamGla/Occs/Manu/Occs_C3.csv')
colnames(occs)

# subset columns
occs$species <- species
occs$longitude <- occs$decimalLongitude
occs$latitude <- occs$decimalLatitude
occs <- subset(occs, select = c("species", "longitude", "latitude"))

# independent
independent <- read.csv("D:/ENM/GamGla/Occs/independent/independent.csv")
head(independent)
independent$species <- species
independent$longitude <- independent$Longitude
independent$latitude <- independent$Latitude
independent <- subset(independent, select = c("species", "longitude", "latitude"))
head(independent)
write.csv(independent, paste0(species, "_ind.csv"), row.names = FALSE)

# split
data_split <- split_data(occs, method = "random",
                         longitude = "longitude",
                         latitude = "latitude", train_proportion = 0.75,
                         save = T, name = species)

# [ alternative ] loading data_split
#all <- read.csv(paste0(species, '_all.csv'))
#colnames(all) <- c('species', 'longitude', 'latitude')
#train <- read.csv(paste0(species, '_training.csv'))
#colnames(train) <- c('species', 'longitude', 'latitude')
#test <- read.csv(paste0(species, '_testing.csv'))
#colnames(test) <- c('species', 'longitude', 'latitude')
#data_split <- list(as.data.frame(all), as.data.frame(train), as.data.frame(test))
#names(data_split) <- c("all", "train", "test")
#View(data_split)

# save the split data
write.csv(data_split["all"], paste0(species, "_all.csv"), row.names = F)
write.csv(data_split["train"], paste0(species, "_training.csv"), row.names = F)
write.csv(data_split["test"], paste0(species, "_testing.csv"), row.names = F)
occs <- read.csv(paste0(species, "_all.csv"))
training <- read.csv(paste0(species, "_training.csv"))
testing <- read.csv(paste0(species, "_testing.csv"))
```

### 🟨 Calibration ####
```r
library(kuenm)
maxent_path = "C:/maxent"

# reading rasters callibration area
vars <- stack(list.files("M_variables", pattern = ".asc", full.names = TRUE))
crs(vars) <- '+proj=longlat +datum=WGS84 +no_defs'
vars <- terra::rast(vars)
plot(vars)
names(vars)

# First selection
s1 <- vars[[c("pc_1", "pc_2", "pc_4", "pc_11")]]
s2 <- vars[[c("pc_6", "pc_8", "pc_1", "pc_7")]]
s3 <- vars[[c("pc_6", "pc_8", "pc_1")]]
s4 <- vars[[c("pc_1", "pc_2", "pc_3")]]
s5 <- vars[[c("pc_4", "pc_5", "pc_6")]]
s6 <- vars[[c("pc_1", "pc_7", "pc_9")]]
s7 <- vars[[c("pc_1", "pc_2", "pc_3", "pc_4")]]
s8 <- vars[[c("pc_1", "pc_2", "pc_3", "pc_4", "pc_5")]]
s9 <- vars[[c("pc_8", "pc_1", "pc_7")]]

# total of 10
{
    sets <- list(s1 <- c("pc_1", "pc_2", "pc_4", "pc_11"),
                 s2 <- c("pc_6", "pc_8", "pc_1", "pc_7"),
                 s3 <- c("pc_6", "pc_8", "pc_1"),
                 s4 <- c("pc_1", "pc_2", "pc_3"),
                 s5 <- c("pc_4", "pc_5", "pc_6"),
                 s6 <- c("pc_1", "pc_7", "pc_9"),
                 s7 <- c("pc_1", "pc_2", "pc_3", "pc_4"),
                 s8 <- c("pc_1", "pc_2", "pc_3", "pc_4", "pc_5"),
                 s9 <- c("pc_8", "pc_1", "pc_7"))
}

# parameters
data <- data_split
variables <- prepare_sets(vars, sets)
methods <- c("covmat", "mve1")
out_dir <- "calibration"
overwrite <- F
parallel <- F

calib <- ellipsoid_calibration(data_split, species = "species", longitude = "longitude",
                               latitude = "latitude", variables = variables, methods = methods, level = 95,
                               selection_criteria = "S_OR_P",
                               error = 5, iterations = 1000, percentage = 50,
                               output_directory = out_dir, overwrite=F, parallel = F)

print(calib@results)
print(calib@selected_parameters)
```

### 🟨 Model ####
```r
setwd("D:/ENM/GamGla/Manu/ellipsenm")

library(raster)
library(terra)
library(ellipsenm)

species <- "Gampsocleis glabra"

# PC G vars
vars <- raster::stack(list.files("G_variables/current/Initial", pattern = ".tif", full.names = TRUE))
raster::crs(vars) <- '+proj=longlat +datum=WGS84 +no_defs'
vars <- terra::rast(vars)
plot(vars[[1]])
names(vars)
# data
occs <- read.csv(paste0(species, '_all.csv'))
colnames(occs)
colnames(occs) <- c('species', 'longitude', 'latitude')
data_split <- c(paste0(species, "_all.csv"), paste0(species, "_training.csv"), paste0(species, "_testing.csv"))
training <- read.csv(paste0(species, '_training.csv'))
colnames(training)
colnames(training) <- c('species', 'longitude', 'latitude')
testing <- read.csv(paste0(species, '_testing.csv'))
colnames(testing)
colnames(testing) <- c('species', 'longitude', 'latitude')

s1 <- vars[[c("pc_1", "pc_2", "pc_3", "pc_4")]]

# creating the model for set1
s1_ell_model_mve_gvars <- ellipsoid_model(data = occs, species = species,
                                       longitude = "longitude", latitude = "latitude",
                                       replicate_type = "bootstrap", percentage = 75,
                                       raster_layers = s1,
                                       method = "mve1",  level = 95,
                                       replicates = 500, prediction = "both",
                                       return_numeric = TRUE,
                                       color_palette = viridis::viridis,
                                       overwrite = FALSE, output_directory = "s1_ell_model_mve1_gvars")

# Fitting model
s1_ell_fit <- ellipsoid_fit(data = occs, longitude = "longitude", latitude = "latitude",
                            method = "mve1",level = 95, raster_layers = s1)
```

### 🟨 Projections ####
```r
#### 🟩 Current ####
current_s1 <- stack(list.files("G_variables/current/Initial", pattern = ".tif", full.names = TRUE))
current_s1 <- terra::rast(current_s1)
plot(current_s1)
#### 🟩 Mid Holocene ####
BCC.CSM1.1.bcmidbi_2.5m_s1 <- stack(list.files("G_variables/BCC-CSM1-1.bcmidbi_2-5m/Initial/", pattern = ".tif", full.names = TRUE))
BCC.CSM1.1.bcmidbi_2.5m_s1 <- terra::rast(BCC.CSM1.1.bcmidbi_2.5m_s1)
plot(BCC.CSM1.1.bcmidbi_2.5m_s1)
MIROC.ESM.mrmidbi_2.5m_s1 <- stack(list.files("G_variables/MIROC-ESM.mrmidbi_2-5m/Initial/", pattern = ".tif", full.names = TRUE))
MIROC.ESM.mrmidbi_2.5m_s1 <- terra::rast(MIROC.ESM.mrmidbi_2.5m_s1)
plot(MIROC.ESM.mrmidbi_2.5m_s1)

# CRS
crs(current_s1) <- '+proj=longlat +datum=WGS84 +no_defs'
crs(BCC.CSM1.1.bcmidbi_2.5m_s1) <- '+proj=longlat +datum=WGS84 +no_defs'
crs(MIROC.ESM.mrmidbi_2.5m_s1) <- '+proj=longlat +datum=WGS84 +no_defs'
# Options
scenarios_s1 <- list(current_s1, BCC.CSM1.1.bcmidbi_2.5m_s1, MIROC.ESM.mrmidbi_2.5m_s1)
options(expressions = 500)

# project
ellipsoid <-s1_ell_fit
projection_variables <-  scenarios_s1
prvariables_format = "GTiff"
sp_name <- species
prediction = "both"
truncate = TRUE
return_numeric = TRUE
tolerance = 1e-60
format = "ascii"
overwrite = FALSE
force_return = FALSE
return_name = NULL
output_directory = "ellipsenm_projections_s1_mve1_gvars"
dir.create("ellipsenm_projections_s1_mve1_gvars")
model_projection(ellipsoid, projection_variables = projection_variables,
                 prvariables_format = prvariables_format, sp_name = sp_name,
                 prediction = prediction, return_numeric = return_numeric,
                 tolerance = tolerance, format = format, overwrite = overwrite,
                 force_return = force_return, return_name = return_name,
                 output_directory = output_directory)

memory.limit(size = 20000)
```

### 🟨 MOP ##### 
```r
setwd('D:/ENM/GamGla/Manu/ellipsenm')
dir.create("MOP_ellipsenm_projections_s1_mve1_gvars")
library(mop)
library(terra)

# load m and g vars
mvars <- terra::rast(raster::stack(list.files("M_variables", pattern = ".asc", full.names = TRUE))) ; terra::rast(mvars) ; plot(mvars)
gvars_current <- terra::rast(raster::stack(list.files("G_variables/current/Initial", pattern = ".tif", full.names = TRUE))) ; terra::rast(gvars_c) ; plot(gvars_c)
gvars_bcc <- terra::rast(raster::stack(list.files("G_variables/BCC-CSM1-1.bcmidbi_2-5m/Initial", pattern = ".tif", full.names = TRUE))) ; terra::rast(gvars_bcc) ; plot(gvars_bcc)
gvars_miroc <- terra::rast(raster::stack(list.files("G_variables/MIROC-ESM.mrmidbi_2-5m/Initial", pattern = ".tif", full.names = TRUE))) ; terra::rast(gvars_miroc) ; plot(gvars_miroc)
gvar_list <- list(gvars_current = gvars_current, gvars_bcc = gvars_bcc, gvars_miroc = gvars_miroc) # projections to mop

# vars for mop
paral <- F
cores <- 5

# loop mop projections
for (gvn in names(gvar_list)) {
    gv <- gvar_list[[gvn]]
    mop_ellipsenm_gv <- mop::mop(m = mvars, g = gv, type = "detailed", calculate_distance = T,
                                  where_distance = "out_range", scale = T, center = T, percentage = .90, 
                                  fix_NA = F, n_cores = cores , parallel = paral, progress_bar = T)
    assign(paste0("mop_ellipsenm_", gvn), mop_ellipsenm_gv)
    mop_ellipsenm_summ_gv <- utils::capture.output(summary(mop_ellipsenm_gv))
    cat(mop_ellipsenm_summ_gv, file= paste("MOP_ellipsenm_projections_s1_mve1_gvars/mop_ellipsenm_summ", gvn,".txt"), sep = "\n")
    
    # plotting following the mop package guide
    library(ggplot2)
    library(tidyterra)
    library(rnaturalearth)
    library(gridExtra)
    library(grid)
    # creating an empty color for NA
    clear <- rgb(0, 0, 255, max = 255, alpha = 0, names = "clear")
    
    buff_shape <- terra::vect("../b_50/b_50.shp")
    # Continent outline
    land <- aggregate(rnaturalearth::ne_countries(scale = "small", returnclass = "sv"))
    
    #### 🟩 Number of variables outside training region
    p1 <- ggplot() +
        geom_spatraster(data = mop_ellipsenm_gv $mop_simple) +
        scale_fill_brewer(type = "seq", name = "", palette = "Purples") +
        geom_sf(data = land, bg = clear, col = "black") +
        geom_sf(data = buff_shape, bg = clear, col = "black", linewidth = 1) +
        coord_sf(xlim = ext(mop_ellipsenm_gv$mop_simple)[1:2], 
                 ylim = ext(mop_ellipsenm_gv$mop_simple)[3:4],
                 expand = FALSE, label_axes = list(bottom = "E", left = "N")) +
        ggtitle("Number of Variables Outside Training Region ") +
        theme_bw() +
        theme(title = element_text(size=19, colour = "black"),
              panel.grid = element_blank(),
              axis.ticks = element_line(linewidth = 0.5, colour="grey"),
              axis.text = element_text(size=19, colour = "#909090"),
              legend.text = element_text(size=15, colour = "black"),
              legend.position = "bottom"
        )
    p1
    
    # Distance from m in g
    p2 <- ggplot() +
        geom_spatraster(data = mop_ellipsenm_gv$mop_distances) +
        scale_fill_continuous(type = "viridis", na.value = clear, name = "") +
        geom_sf(data = land, bg = clear, col = "black") +
        geom_sf(data = buff_shape, bg = clear, col = "black", linewidth = 1) +
        coord_sf(xlim = ext(mop_ellipsenm_gv$mop_distances)[1:2], 
                 ylim = ext(mop_ellipsenm_gv$mop_distances)[3:4],
                 expand = FALSE, label_axes = list(bottom = "E", left = "N")) +
        ggtitle("Distance Between M and G") +
        theme_bw() +
        theme(title = element_text(size=19, colour = "black"),
              panel.grid = element_blank(),
              axis.ticks = element_line(linewidth = 0.5, colour="grey"),
              axis.text = element_text(size=19, colour = "#909090"),
              legend.text = element_text(size=15, colour = "black"),
              legend.position = "bottom"
        )
    p2
    
    # Variables Outside Training Region at Low End
    dl <- droplevels(mop_ellipsenm_gv$mop_detailed$towards_low_combined)
    p3 <- ggplot() +
        geom_spatraster(data = dl) +
        scale_fill_brewer(name = "", palette = "Blues") +
        geom_sf(data = land, bg = clear, col = "black") +
        geom_sf(data = buff_shape, bg = clear, col = "black", linewidth = 1) +
        coord_sf(xlim = ext(mop_ellipsenm_gv$mop_simple)[1:2], 
                 ylim = ext(mop_ellipsenm_gv$mop_simple)[3:4],
                 expand = FALSE, label_axes = list(bottom = "E", left = "N")) +
        ggtitle("Variables Outside Training Region at Low End") +
        theme_bw() +
        theme(title = element_text(size=19, colour = "black"),
              panel.grid = element_blank(),
              axis.ticks = element_line(linewidth = 0.5, colour="grey"),
              axis.text = element_text(size=19, colour = "#909090"),
              legend.text = element_text(size=15, colour = "black"),
              legend.position = "bottom"
        )
    p3
    
    # Variables Outside Training Region at High End
    dl <- droplevels(mop_ellipsenm_gv$mop_detailed$towards_high_combined)
    p4 <- ggplot() +
        geom_spatraster(data = dl) +
        scale_fill_brewer(name = "", palette = "Oranges") +
        geom_sf(data = land, bg = clear, col = "black") +
        geom_sf(data = buff_shape, bg = clear, col = "black", linewidth = 1) +
        coord_sf(xlim = ext(mop_ellipsenm_gv$mop_simple)[1:2], 
                 ylim = ext(mop_ellipsenm_gv$mop_simple)[3:4],
                 expand = FALSE, label_axes = list(bottom = "E", left = "N")) +
        ggtitle("Variables Outside Training Region at High End") +
        theme_bw() +
        theme(title = element_text(size=19, colour = "black"),
              panel.grid = element_blank(),
              axis.ticks = element_line(linewidth = 0.5, colour="grey"),
              axis.text = element_text(size=19, colour = "#909090"),
              legend.text = element_text(size=15, colour = "black"),
              legend.position = "bottom"
        )
    p4
    grobname2 <- grid.arrange(p1,  p2, p3, p4,
                              ncol = 2)
}

# save rasters with distances between g and m  
for (gvn in names(gvar_list)) {
    raster::writeRaster(get(paste0("mop_ellipsenm_", gvn))$mop_distances, 
                        filename = paste0("MOP_ellipsenm_projections_s1_mve1_gvars/MOP10%_distances_", gvn, ".tif"))
}
```

## 🟧 Threshold suitability & CPH ####
```r
setwd("D:/ENM/GamGla/Manu/ellipsenm")
library(ggplot2)
library(terra)
library(sf)
library(raster)

# occs
occs <- read.csv("D:/ENM/GamGla/Occs/Manu/Region.csv")
occs$status[is.na(occs$status)] <- "extant"
occs <- st_as_sf(occs, coords = c('decimalLongitude', 'decimalLatitude'), crs = 4326, remove = F)

# load rasters and add values to occs
cal_mean <- raster::raster("s1_ell_model_mve1_gvars/mean_suitability_calibration_.tif")

# Quantiles for core
# Ellipsenm already thresholds unsuitable/suitable
occs$cal_mean <- extract(cal_mean, occs)
percent_cali <- stats::quantile(occs$cal_mean, c(.05, .10, .15, .50, .75, .85, .95)) # diff quantiles to visualize
percent_cali

# Results table
lb <- capture.output(cat(rep("# ", 20))) # lb var for results file
cat(c(lb, paste("###: Quantiles from Calibration Mean:"), capture.output(print(percent_cali)), lb), 
    sep = "\n", file= "CPH__ellipsenm_projections_s1_mve1_gvars/CPH_Results.txt", append=F)

current_proj <- terra::rast("ellipsenm_projections_s1_mve1_gvars/suitability_scenario1_Gampsocleis glabra.asc")
bcc_proj <- terra::rast("ellipsenm_projections_s1_mve1_gvars/suitability_scenario2_Gampsocleis glabra.asc")
miroc_proj <- terra::rast("ellipsenm_projections_s1_mve1_gvars/suitability_scenario3_Gampsocleis glabra.asc")
projlist <- list(current_proj = current_proj, bcc_proj = bcc_proj, miroc_proj = miroc_proj)

# loop thresh
for (prjn in names(projlist)) {
    prj <- projlist[[prjn]]
    clip.u <- terra::ifel(prj < percent_cali[[2]], 0, prj)
    #plot(clip.u, main = paste0(prjn, " projection w/o unsuitable"))
    clip.p <- terra::ifel(clip.u > 0 & clip.u < percent_cali[[5]], 1, clip.u)
    #plot(clip.p, main = paste0(prjn, " projection w/o unsuitable, periph marked"))
    clip.c <- terra::ifel(clip.p < 1 & clip.p > 0, 2, clip.p)
    #plot(clip.c, main = paste0(prjn, " projection w/o unsuitable, periph marked, core marked"))
    raster::writeRaster(clip.c, filename = paste0("Threshold_ellipsenm_projections_s1_mve1_gvars/CPH__ellipsenm_projections_s1_mve1_gvars_binned_", prjn, ".tif"))
    clip.c <- raster::raster(clip.c)
    occs[[paste0("quantile_", prjn)]] <- terra::extract(clip.c, occs)
}
# save csv with all binned projections
occs <- sf::st_drop_geometry(occs)
write.csv(occs, "CPH__ellipsenm_projections_s1_mve1_gvars/CPH__ellipsenm_projections_s1_mve1_gvars_binned.csv", row.names = F)
```

### 🟨 Plot CPH ####
```r
setwd("D:/ENM/GamGla/Manu/ellipsenm")
library(ggplot2)
library(terra)
library(sf)
library(raster)

cph <- read.csv("CPH__ellipsenm_projections_s1_mve1_gvars/CPH__ellipsenm_projections_s1_mve1_gvars_binned.csv")
cph$quantile_current_proj[cph$quantile_current_proj == 0] <- "Unsuitable"
cph$quantile_current_proj[cph$quantile_current_proj == 1] <- "Periphery"
cph$quantile_current_proj[cph$quantile_current_proj == 2] <- "Core"

# line break for results table
lb <- capture.output(cat(rep("# ", 20)))
# results table
all <- subset(cph, select=c("status", "quantile_current_proj"))
all <- as.data.frame(table(all)) ; print(all)
cat(c(lb, paste("###: CPH table between groups:"), capture.output(print(all)), lb), 
    sep = "\n", file= "CPH__ellipsenm_projections_s1_mve1_gvars/CPH_Results.txt", append=T)

temp <- unlist(table(cph$quantile_current_proj)) ; print(temp)
cat(c(lb, paste("###: Suitability between all Occurrences:"), capture.output(print(temp)), lb), 
    sep = "\n", file= "CPH__ellipsenm_projections_s1_mve1_gvars/CPH_Results.txt", append=T)

temp <- unlist(table(cph$Region[cph$quantile_current_proj == "Unsuitable"])) ; print(temp)
cat(c(lb, paste("###: Unsuitable between Regions:"), capture.output(print(temp)), lb), 
    sep = "\n", file= "CPH__ellipsenm_projections_s1_mve1_gvars/CPH_Results.txt", append=T)

# bar plot with totals
ordervalue <- c("Unsuitable", "Periphery", "Core")
p1 <- ggplot(cph, aes(x = quantile_current_proj, fill = status)) +
    geom_bar(aes(x=factor(quantile_current_proj, c("Unsuitable", "Periphery", "Core"))), position = "dodge") +
    labs(x = "Suitability", y = "Populations") +
    scale_fill_manual(values = c("extant"="#ffb000", "extinct"="#648fff"), 
                      name = "Status", labels = c("Extant", "Extinct")) +
    guides(fill="none") +
    geom_text(data=all,aes(x= quantile_current_proj,y=Freq,label=Freq),vjust=-1, fontface= 2, size=7, position = position_dodge(width=0.9)) +
    ylim(0,230) +
    theme_minimal() +
    theme(panel.grid = element_blank(),
          axis.ticks = element_line(linewidth = 0.5, colour="grey"),
          axis.text = element_text(size=19, colour = "#909090"),
          axis.title = element_text(size=21, face="bold", colour = "black"))
p1
# save
library(devtools)
jpeg("plots/CPH_extinct_extant_core-periph-unsuit_current.jpeg", width = 400, height = 600, units='px', quality = 100)
p1
dev.off()

# plot of points on current cph map
cph <- st_as_sf(cph, coords = c('decimalLongitude', 'decimalLatitude'), crs = 4326) 
r <- terra::rast("Threshold_ellipsenm_projections_s1_mve1_gvars/CPH__ellipsenm_projections_s1_mve1_gvars_binned_current_proj.tif")
countries <- rnaturalearth::ne_countries(scale = "small", returnclass = "sv")
# plot and save
library(devtools)
jpeg("CPH__ellipsenm_projections_s1_mve1_gvars/CPH_extinct_extant_core-periph-unsuit_current_w_occs.jpeg", width = 1800, height = 800, units='px', quality = 100)
terra::plot(r, col=c("lightgrey", "#606060", "black"), legend=T) ; terra::plot(countries, add=T)
plot(cph, col = "blue", add=T, pch = 20, cex = .7)
plot(cph[cph$quantile_current_proj == "Unsuitable", ], col = "red", add=T, pch = 20, cex = 1)
dev.off()

# Histogram of suitability between groups
r <- raster::raster("ellipsenm_projections_s1_mve1_gvars/suitability_scenario1_Gampsocleis glabra.asc") # current proj continuous
cph$current <- terra::extract(r, cph)

# plot
p1 <- ggplot(cph, aes(x = current, fill = status)) +
    geom_rect(aes(xmin = 0.4467963, xmax = 1, ymin = 0, ymax = Inf), fill = "#dbbfda") +
    geom_histogram(position = "dodge", bins = 10) +
    labs(title = "Extinct vs. Extant Suitability \n(Current)", x = "Suitability", y = "Populations") +
    scale_fill_manual(values = c("extant"="#ffb000", "extinct"="#648fff"), 
                      name = "Status", labels = c("Extant", "Extinct")) +
    geom_vline(xintercept = 0.4467963, color = "#810f7c", linewidth = 1.2, linetype = "dashed") +
    theme_minimal()
# save
jpeg("CPH__ellipsenm_projections_s1_mve1_gvars/CPH_extinct_extant_current_suitability_hist.jpeg", width = 500, height = 400, units='px', quality = 100)
p1
dev.off()
```

### 🟨 Fisher exact test ####
```r
cph <- read.csv("CPH__ellipsenm_projections_s1_mve1_gvars/CPH__ellipsenm_projections_s1_mve1_gvars_binned.csv")
cph$quantile_current_proj[cph$quantile_current_proj == 0] <- "Unsuitable"
cph$quantile_current_proj[cph$quantile_current_proj == 1] <- "Periphery"
cph$quantile_current_proj[cph$quantile_current_proj == 2] <- "Core"

cph$status <- factor(cph$status, levels = c("extant", "extinct"))
cph$quantile_current_proj <- factor(cph$quantile_current_proj, levels = c("Core", "Periphery", "Unsuitable"))
table(cph$quantile_current_proj)

tab <- table(cph$quantile_current_proj, cph$status)
table(cph$quantile_current_proj, cph$status)
table(cph$status)

# Test if chisq or fisher test is more suitable
chisq.test(tab)$expected # expected frequency below 5. Fisher test selected

# Fisher
test <- fisher.test(tab)
test$p.value

#### 🟩 Fisher Effect Size ####
cph <- read.csv("CPH__ellipsenm_projections_s1_mve1_gvars/CPH__ellipsenm_projections_s1_mve1_gvars_binned.csv")

cph$quantile_current_proj <- as.numeric(cph$quantile_current_proj)
cph$status <- factor(cph$status, levels = c("extant", "extinct"))

ef <- effectsize::rank_biserial(x=cph$quantile_current_proj, y = cph$status)
effectsize::interpret_rank_biserial(ef, rules = "funder2019")
```

### 🟨 Mann-whitney U ####
```r
cph <- read.csv("CPH__ellipsenm_projections_s1_mve1_gvars/CPH__ellipsenm_projections_s1_mve1_gvars_binned.csv")
current_proj <- terra::rast("ellipsenm_projections_s1_mve1_gvars/suitability_scenario1_Gampsocleis glabra.asc")
cph <- st_as_sf(cph, coords = c('decimalLongitude', 'decimalLatitude'), crs = 4326, remove = F)
cph$current <- terra::extract(current_proj, cph, ID=F)[[1]]

# Mann-Whitney U test
test <- wilcox.test(current ~ status, alternative = "greater", data=cph) ; test
z <- qnorm(0.0002062) ; z

#### 🟩 Mann-whitney U Effect Size ####
ef <- effectsize::rank_biserial(x=cph$current, y = cph$status, alternative = "greater")
effectsize::interpret_rank_biserial(ef, rules = "funder2019")
```

## 🟧 Final model ####
### 🟨 Suitability MOP binary & cleanup ####
```r
setwd("D:/ENM/GamGla/Manu/ellipsenm")

library(raster)
library(terra)
library(rnaturalearth)
library(sf)

# country outline for plots
countries <- rnaturalearth::ne_countries(scale = "small", returnclass = "sv")

gcms <- c("current", "bcc", "miroc")

for(gcm in gcms) {
  print(paste0("Starting ", gcm, "  __________"))
  
  # Mops
  MOP <- terra::rast(paste0("MOP_ellipsenm_projections_s1_mve1_gvars/MOP10%_distances_gvars_", gcm, ".tif"))
  # Binary
  MOP <- terra::ifel(MOP > 0, 1, 0)
  # Load thresholded model
  Thresh_Mod <- terra::rast(paste0("Threshold_ellipsenm_projections_s1_mve1_gvars/CPH__ellipsenm_projections_s1_mve1_gvars_binned_", gcm, "_proj.tif"))
  # Binary
  Thresh_Mod <- terra::ifel(Thresh_Mod > 0, 1, 0)
  
  # Mask final thresholded model with the binary
  fmb <- ifel(MOP > 0, 0, Thresh_Mod) # any disimilarity will be masked from the thresholded final model
  terra::plot(MOP, main = paste0(gcm, " MOP"), sub = "Dissimilarity = 1, Similarity = 0") ; terra::plot(countries, add=T)
  terra::plot(Thresh_Mod, main = paste0(gcm, " MOPthresholded model"), sub = "Suitable = 1, Unsuitable = 0") ; terra::plot(countries, add=T)
  terra::plot(fmb, main  = paste0(gcm, " MOPfinal mod without extrapolation"), sub = "Suitable & Non Extrapolative = 1, Unsuitable / Extrapolative = 0") ; terra::plot(countries, add=T)
  # save raster
  raster::writeRaster(fmb, filename = paste0("MOP_ellipsenm_projections_s1_mve1_gvars/MOP10%_Thresh10%_Set1_", gcm, "_FinalBinary.tif"))
  
  # save plots
  library(devtools)
  # mop
  jpeg(paste0("plots/MOP10_s1_", gcm, "_projection.jpeg"), width = 1800, height = 1500, units='px', quality = 100)
  terra::plot(MOP, main = paste0(gcm, " MOP"), sub = "Dissimilarity = 1, Similarity = 0") ; terra::plot(countries, add=T)
  dev.off()
  # thresholded model
  jpeg(paste0("plots/Thresh5_Set1_", gcm, "_Thresholded.Model.jpeg"), width = 1800, height = 1500, units='px', quality = 100)
  terra::plot(Thresh_Mod, main = paste0(gcm, " Thresholded model"), sub = "Suitable = 1, Unsuitable = 0") ; terra::plot(countries, add=T)
  dev.off()
  # final model
  jpeg(paste0("plots/Final_MOP10_Thresh5_Set1_", gcm, "_FinalModel.jpeg"), width = 1800, height = 1500, units='px', quality = 100)
  terra::plot(fmb, main  = paste0(gcm, " MOPfinal mod without extrapolation"), sub = "Suitable & Non Extrapolative = 1, Unsuitable / Extrapolative = 0") ; terra::plot(countries, add=T)
  dev.off()
}
```

### 🟨 Final model ####
```r
setwd("D:/ENM/GamGla/Manu/ellipsenm")
library(rnaturalearth)
library(terra)
library(raster)

dir.create("Final_Model_Clipped")

# country outline for plots
countries <- rnaturalearth::ne_countries(scale = "small", returnclass = "sv")
# gcms
gcms <- c("current", "bcc", "miroc")
projs <- list("current" = "ellipsenm_projections_s1_mve1_gvars/suitability_scenario1_Gampsocleis glabra.asc",
              "bcc" = "ellipsenm_projections_s1_mve1_gvars/suitability_scenario2_Gampsocleis glabra.asc",
              "miroc" = "ellipsenm_projections_s1_mve1_gvars/suitability_scenario3_Gampsocleis glabra.asc")

for(gcm in gcms) {
  proj <- projs[[gcm]]
  print(paste0("Starting ", gcm, "  __________"))
  print(paste0("Path to Ellipsenm projection for ", gcm, ": ", proj))
  
  fmb <- terra::rast(paste0("MOP_ellipsenm_projections_s1_mve1_gvars/MOP10%_Thresh10%_Set1_", gcm, "_FinalBinary.tif")) # final model binary
  proj <- terra::rast(proj) # final model proj
  fm <- ifel(fmb == 0, NA, proj) # final model, masked with unsuitable & extrapolative areas
  countries <- rnaturalearth::ne_countries(scale = "small", returnclass = "sv") # country outline
  plot(fmb, main = paste0(gcm, " Mop10%_Thresh10%")) ; plot(countries, add=T)
  plot(proj, main = paste0(gcm, " Thresh10%_Model")) ; plot(countries, add=T)
  plot(fm, main = paste0(gcm, " Final_Mop10%_Thresh10%"))  ; plot(countries, add=T)
  # save raster
  raster::writeRaster(fm, filename = paste0("Final_Model_Clipped/Final_MOP10%_Thresh10%_Set1_", gcm, "_FinalModel.tif"))
}
```

## 🟧 Mahalanobis dist ####
### 🟨 Threshold ####
```r
setwd("D:/ENM/GamGla/Manu/ellipsenm")
library(ggplot2)
library(terra)
library(sf)
library(raster)

dir.create("Maha_ellipsenm_projections_s1_mve1_gvars")

# occs
occs <- read.csv("D:/ENM/GamGla/Occs/Manu/Region.csv")
occs$status[is.na(occs$status)] <- "extant"
occs <- st_as_sf(occs, coords = c('decimalLongitude', 'decimalLatitude'), crs = 4326, remove = F)

# mahalanobis raster
maha_cal <- raster::raster("s1_ell_model_mve1_gvars/mean_mahalanobis_calibration_.tif")
maha_current <- raster::raster("ellipsenm_projections_s1_mve1_gvars/mahalanobis_scenario1_Gampsocleis glabra.asc") # current proj continuous
maha_BCC.CSM1.1.bcmidbi_2.5m_s1 <- raster::raster("ellipsenm_projections_s1_mve1_gvars/mahalanobis_scenario2_Gampsocleis glabra.asc")
maha_MIROC.ESM.mrmidbi_2.5m_s1 <- raster::raster("ellipsenm_projections_s1_mve1_gvars/mahalanobis_scenario3_Gampsocleis glabra.asc")

# percentiles for current
occs$maha_cal <- terra::extract(maha_cal, occs)
percent_cali <- stats::quantile(occs$maha_cal, c(.05, .25, .50, .75, .95)) # diff percentiles
percent_cali 

# For the results table
lb <- capture.output(cat(rep("# ", 20)))
cat(c(lb, paste("###: Quantiles from Mahalanobis Calibration Mean:"), capture.output(print(percent_cali)), lb), 
    sep = "\n", file= "Maha_ellipsenm_projections_s1_mve1_gvars/Maha_Results.txt", append=F)

# add values to occs
occs$maha_cal <- terra::extract(maha_cal, occs)
occs$maha_current <- terra::extract(maha_current, occs)
occs$maha_BCC.CSM1.1.bcmidbi_2.5m_s1 <- terra::extract(maha_BCC.CSM1.1.bcmidbi_2.5m_s1, occs)
occs$maha_MIROC.ESM.mrmidbi_2.5m_s1 <- terra::extract(maha_MIROC.ESM.mrmidbi_2.5m_s1, occs)

maha_current <- terra::rast(maha_current)

# percentiles
clip.4 <- terra::ifel(maha_current > percent_cali[[5]], 95, maha_current)
plot(clip.4, main = paste0("Mahalanobis", " projection 95%"))
clip.3 <- terra::ifel(clip.4 > percent_cali[[4]] & clip.4 < 95, 75, clip.4)
plot(clip.3, main = paste0("Mahalanobis", " projection 75%"))
clip.2 <- terra::ifel(clip.3 > percent_cali[[3]] & clip.3 < 75, 50, clip.3)
plot(clip.2, main = paste0("Mahalanobis", " projection 50%"))
clip.1 <- terra::ifel(clip.2 > percent_cali[[2]] & clip.2 < 50, 25, clip.2)
plot(clip.1, main = paste0("Mahalanobis", " projection 25%"))
clip.0 <- terra::ifel(clip.1 > percent_cali[[1]] & clip.1 < 25, 5, clip.1)
plot(clip.0, main = paste0("Mahalanobis", " projection 5%"))
clip.x <- terra::ifel(clip.0 < 5, 0, clip.0)
plot(clip.x, main = paste0("Mahalanobis", " projection 0%"))
names(clip.x) <- "quantile"

writeRaster(clip.x, filename = "Maha_ellipsenm_projections_s1_mve1_gvars/Maha_ellipsenm_projections_s1_mve1_gvars_binned.tif")

# save csv with all binned percentiles and projections
clip.x <- terra::rast("Maha_ellipsenm_projections_s1_mve1_gvars/Maha_ellipsenm_projections_s1_mve1_gvars_binned.tif")
occs$quantile <- terra::extract(clip.x, occs, ID=F)[[1]]
occs <- sf::st_drop_geometry(occs)
write.csv(occs, "Maha_ellipsenm_projections_s1_mve1_gvars/Maha_ellipsenm_projections_s1_mve1_gvars_binned.csv", row.names = F)

# write extinct and extant csvs
extinct<- occs[ which(occs$status=='extinct'), ]
head(extinct)
extant<- occs[ which(occs$status=='extant'), ]
head(extant)
write.csv(extinct, 'MahalanobisDistance_extinct.csv', row.names = F)
write.csv(extant, 'MahalanobisDistance_extant.csv', row.names = F)
```

### 🟨 Histogram ####
```r
maha <- read.csv("Maha_ellipsenm_projections_s1_mve1_gvars/Maha_ellipsenm_projections_s1_mve1_gvars_binned.csv")
maha$quantile[maha$quantile == 95] <- "Unsuitable"
maha$quantile[maha$quantile == 75] <- "Quantile75"
maha$quantile[maha$quantile == 50] <- "Quantile50"
maha$quantile[maha$quantile == 25] <- "Quantile25"
maha$quantile[maha$quantile == 5] <- "Quantile5"
maha$quantile[maha$quantile == 0] <- "Core"

# line break for results table
lb <- capture.output(cat(rep("# ", 20)))
# results table
allmaha <- subset(maha, select=c("status", "quantile"))
allmaha <- as.data.frame(table(allmaha)) ; print(allmaha)
cat(c(lb, paste("###: Mahalanobis table between groups:"), capture.output(print(allmaha)), lb), 
    sep = "\n", file= "Maha_ellipsenm_projections_s1_mve1_gvars/Maha_Results.txt", append=T)

temp <- unlist(table(maha$quantile)) ; print(temp)
cat(c(lb, paste("###: Mahalanobis between all Occurrences:"), capture.output(print(temp)), lb), 
    sep = "\n", file= "Maha_ellipsenm_projections_s1_mve1_gvars/Maha_Results.txt", append=T)

temp <- unlist(table(maha$Region[maha$quantile == "Unsuitable"])) ; print(temp)
cat(c(lb, paste("###: Mahalanabis between Regions:"), capture.output(print(temp)), lb), 
    sep = "\n", file= "Maha_ellipsenm_projections_s1_mve1_gvars/Maha_Results.txt", append=T)

# bar plot with totals
ordervalue <- c("Unsuitable", "Quantile75", "Quantile50", "Quantile25", "Quantile5", "Core")
p1 <- ggplot(maha, aes(x = quantile, fill = status)) +
    geom_bar(aes(x=factor(quantile, c("Unsuitable", "Quantile75", "Quantile50", "Quantile25", "Quantile5", "Core"))), position = position_dodge2(preserve = "single")) +
    labs(title = "Extinct vs. Extant \nCore x Periphery", x = "Mahalanobis", y = "Populations") +
    scale_fill_manual(values = c("extant"="#ffb000", "extinct"="#648fff"), 
                      name = "Status", labels = c("Extant", "Extinct")) +
    geom_text(data=allmaha,aes(x= quantile,y=Freq,label=Freq),vjust=-1, fontface= 2, position = position_dodge2(width=0.9, preserve = "single")) +
    ylim(0,230) +
    theme_minimal()
p1
# save
library(devtools)
jpeg("plots/Maha_extinct_extant_core-periph-unsuit_current.jpeg", width = 500, height = 400, units='px', quality = 100)
p1
dev.off()
jpeg("Maha_ellipsenm_projections_s1_mve1_gvars/Maha_extinct_extant_core-periph-unsuit_current.jpeg", width = 500, height = 400, units='px', quality = 100)
p1
dev.off()

# plot of points on current maha map
maha <- st_as_sf(maha, coords = c('decimalLongitude', 'decimalLatitude'), crs = 4326, remove = F) 
r <- terra::rast("Maha_ellipsenm_projections_s1_mve1_gvars/Maha_ellipsenm_projections_s1_mve1_gvars_binned.tif")
countries <- rnaturalearth::ne_countries(scale = "small", returnclass = "sv")
# plot and save
library(devtools)
jpeg("Maha_ellipsenm_projections_s1_mve1_gvars/Maha_extinct_extant_core-periph-unsuit_current_w_occs.jpeg", width = 1800, height = 800, units='px', quality = 100)
terra::plot(r, col=c("white", "#b0b0b0", "#999999", "#505050", "#202020", "black"), legend=T) ; terra::plot(countries, add=T)
plot(maha, col = "blue", add=T, pch = 20, cex = .7)
plot(maha[maha$quantile == "Unsuitable", ], col = "red", add=T, pch = 20, cex = 1)
dev.off()

# Histogram of suitability between groups
# plot
p1 <- ggplot(maha, aes(x = maha_current, fill = status)) +
    geom_rect(aes(xmin = 14.7101469, xmax = Inf, ymin = 0, ymax = Inf), fill = "#969696") +
    geom_rect(aes(xmin = 5.5902185, xmax = 14.7101469, ymin = 0, ymax = Inf), fill = "#bdd4e7") +
    geom_rect(aes(xmin = 2.8521247, xmax = 5.5902185, ymin = 0, ymax = Inf), fill = "#9babd1") +
    geom_rect(aes(xmin = 1.6113051, xmax = 2.8521247, ymin = 0, ymax = Inf), fill = "#8a7bb9") +
    geom_rect(aes(xmin = 0.5889364, xmax = 1.6113051, ymin = 0, ymax = Inf), fill = "#87479e") +
    geom_rect(aes(xmin = -Inf, xmax = 0.5889364, ymin = 0, ymax = Inf), fill = "#810f7c") +
    geom_histogram(position = "identity" , bins = 10, colour = "white") +
    labs(x = "Mahalanobis Distance (D2)", y = "Populations") +
    scale_x_continuous(expand = c(0, 0)) +
    ylim(0, 150) +
    scale_fill_manual(values = c("extant"="#ffb000", "extinct"="#648fff"), 
                      name = "Status", labels = c("Extant", "Extinct")) +
    geom_vline(xintercept = 0.5889364, color = "#810f7c", linewidth = 1.2, linetype = "dashed") +
    guides(fill=F) +
    theme_minimal() +
    theme(panel.grid = element_blank(),
          axis.ticks = element_line(linewidth = 0.5, colour="grey"),
          axis.text = element_text(size=19, colour = "#909090"),
          axis.title = element_text(size=21, face="bold", colour = "black"))
p1
# save
jpeg("Maha_ellipsenm_projections_s1_mve1_gvars/Maha_extinct_extant_current_suitability_hist.jpeg", width = 500, height = 600, units='px', quality = 100)
p1
dev.off()
```
### 🟨 Violin plot ####
```r
setwd("D:/ENM/GamGla/Manu/ellipsenm")
library(ggplot2)
library(gridExtra)
library(dplyr)
library(tidyr)
library(tidyverse)
library(grid)

# occs
region <- read.csv("Maha_ellipsenm_projections_s1_mve1_gvars/Maha_ellipsenm_projections_s1_mve1_gvars_binned.csv")
head(region)

p1 <- ggplot(region,aes(status,maha_current)) +
    geom_violin(fill="lightgray") +
    scale_color_manual(values = mypal) +
    geom_jitter(aes(colour = factor(Region))) +
    guides(size="none") +
    stat_sum(alpha=0.7) +
    scale_size(breaks=1:2, range=c(3,6)) +
    scale_x_discrete(labels = c("Extant", "Extinct")) +
    labs(x = "Status", y = "Mahalanobis Distance", colour = "Region") +
    theme_minimal() +
    theme(panel.grid = element_blank(),
          axis.ticks = element_line(linewidth = 0.5, colour="grey"),
          axis.text = element_text(size=19, colour = "#909090"),
          axis.title = element_text(size=21, face="bold", colour = "black"),
          legend.title = element_text(size=21),
          legend.position = c(1, .8),
          legend.text = element_text(size=18),
          plot.margin = margin(5.5,80,5.5,5.5)) +
    guides(colour = guide_legend(override.aes = list(size = 5)))
p1

# save
jpeg("Maha_ellipsenm_projections_s1_mve1_gvars/Maha_extinct_extant_current_suitability_violin.jpeg", width = 700, height = 400, units='px', quality = 100)
p1
dev.off()
```
### 🟨 Normality test ####
```r
setwd('D:/ENM/GamGla/Manu/ellipsenm')
region <- read.csv("Maha_ellipsenm_projections_s1_mve1_gvars/Maha_ellipsenm_projections_s1_mve1_gvars_binned.csv")

# shapiro-wilk test
p1_st1 <- capture.output(shapiro.test(region[region$status == "extinct", ]$maha_current))
p1_st2 <- capture.output(shapiro.test(region[region$status == "extant", ]$maha_current))
p2_st1 <- capture.output(shapiro.test(region[region$status == "extinct", ]$maha_BCC.CSM1.1.bcmidbi_2.5m_s1))
p2_st2 <- capture.output(shapiro.test(region[region$status == "extant", ]$maha_BCC.CSM1.1.bcmidbi_2.5m_s1))
p3_st1 <- capture.output(shapiro.test(region[region$status == "extinct", ]$maha_MIROC.ESM.mrmidbi_2.5m_s1))
p3_st2 <- capture.output(shapiro.test(region[region$status == "extant", ]$maha_MIROC.ESM.mrmidbi_2.5m_s1))

# qq-plot
p1 <- ggplot(region, aes(sample = maha_current)) +
    geom_qq() +
    geom_qq_line() +
    ylab("Sample Quantiles") +
    xlab("") +
    labs(title = "Current", 
         subtitle = paste0("     Extinct Shapiro-Wilk:\n          ", grep("^W" , p1_st1, value = T),
                           "\n     Extant Shapiro-Wilk:\n          ", grep("^W" , p1_st2, value = T))) +
    theme_minimal() +
    theme(title = element_text(size=15, face="bold", colour = "black"),
          plot.subtitle = element_text(size = 12, hjust = 0, face = 1),
          axis.text = element_text(size=15, colour = "#909090"),
          axis.title = element_text(size=15, face="bold", colour = "black"))
p2 <- ggplot(region, aes(sample = maha_BCC.CSM1.1.bcmidbi_2.5m_s1)) +
    geom_qq() +
    geom_qq_line() +
    xlab("Theoretical Quantiles") +
    ylab("") +
    labs(title = "BCC",
         subtitle = paste0("     Extinct Shapiro-Wilk:\n          ", grep("^W" , p2_st1, value = T),
                           "\n     Extant Shapiro-Wilk:\n          ", grep("^W" , p2_st2, value = T))) +
    theme_minimal() +
    theme(title = element_text(size=15, face="bold", colour = "black"),
          plot.subtitle = element_text(size = 12, hjust = 0, face = 1),
          axis.text = element_text(size=15, colour = "#909090"),
          axis.title = element_text(size=15, face="bold", colour = "black"))
p3 <- ggplot(region, aes(sample = maha_MIROC.ESM.mrmidbi_2.5m_s1)) +
    geom_qq() +
    geom_qq_line() +
    ylab("")  +
    xlab("") +
    labs(title = "MIROC",
         subtitle = paste0("     Extinct Shapiro-Wilk:\n          ", grep("^W" , p3_st1, value = T),
                           "\n     Extant Shapiro-Wilk:\n          ", grep("^W" , p3_st2, value = T))) +
    theme_minimal() +
    theme(title = element_text(size=15, face="bold", colour = "black"),
          plot.subtitle = element_text(size = 12, hjust = 0, face = 1),
          axis.text = element_text(size=15, colour = "#909090"),
          axis.title = element_text(size=15, face="bold", colour = "black"))
qq.together <- grid.arrange(p1,  p2,  p3, nrow = 1)
ggsave(filename = "qq.together.jpeg", plot = qq.together, width = 300, height = 150, units='mm', dpi = 600)
```

### 🟨 Mann-Whitney u ####
```r
region <- read.csv("Maha_ellipsenm_projections_s1_mve1_gvars/Maha_ellipsenm_projections_s1_mve1_gvars_binned.csv")

# Mann-Whitney U test
wilcox.test(maha_current ~ status, alternative = "less", data=region)
Zstat <- qnorm(0.0002433) #-3.488026

# Effect size ####
library(effectsize)
rb <- rank_biserial(
    maha_current ~ status,
    y = NULL,
    data = region,
    mu = 0,
    paired = FALSE,
    reference = NULL,
    ci = 0.95,
    alternative = "less",
    verbose = TRUE)

interpret_rank_biserial(rb, rules = "funder2019")
```

# 🟥 Land Use ####
```r
dir.create("landuse")
setwd("landuse")
```

## 🟧 Buffer groups and crop land use variables
```r
library(sf)
library(raster)

region <- read.csv("../Maha_ellipsenm_projections_s1_mve1_gvars/Maha_ellipsenm_projections_s1_mve1_gvars_binned.csv")
table(unlist(region$status))

extinct <- region[region$status == "extinct", ]
extant <- region[region$status == "extant", ]

# extant 50 km buffer
dir.create("extant50")
extant50 <- sf::st_as_sf(extant, coords = c("decimalLongitude", "decimalLatitude"), crs="EPSG:4326")
extant50 <- sf::as_Spatial(extant50)
extant50 <- raster::buffer(extant50, width=50000, dissolve=TRUE)
plot(extant50)
# write 
shapefile(x= extant50, file = "extant50/extant50.shp")

# extinct 50 km buffer
dir.create("extinct50")
extinct50 <- sf::st_as_sf(extinct, coords = c("decimalLongitude", "decimalLatitude"), crs="EPSG:4326")
extinct50 <- sf::as_Spatial(extinct50)
extinct50 <- raster::buffer(extinct50, width=50000, dissolve=TRUE)
plot(extinct50)
# write 
shapefile(x= extinct50, file = "extinct50/extinct50.shp")

# shapefile
buff_shape <- terra::vect("extinct50/extinct50.shp")

# reading in the vars
temp <- list.files("D:/ENM/Variables/HILDA_LandUse/hildap_vGLOB-1.0_geotiff_wgs84/hildap_GLOB-v1.0_lulc-states", pattern = ".tif",
                   full.names = TRUE)
var_list <- raster::stack(temp[1:4])
rvars <- terra::rast(var_list)

# crs for terra::crop
terra::same.crs(buff_shape, rvars)

buff_cmrvars <- terra::crop(rvars, buff_shape, mask = TRUE)
r <- raster::brick(buff_cmrvars)
plot(r[[1]])

# aggregate
a <- lapply(1:nlayers(r), function(x) {
  ar <- raster::aggregate(raster::getValues(raster::area(r[[x]], weights=FALSE)), by=list(raster::getValues(r[[x]])), sum)
  names(ar) <- c("LU", names(r)[x])
  ar 
  })

# combine years
all <- a[[1]]
for (year in a[-1]) {
  all <- merge(all, year, by = "LU", all=T)
  }

extinct_hilda <- all
extinct_hilda <- extinct_hilda[which(extinct_hilda$LU!=c(0, 77)), ]

extinct_hilda$LU[extinct_hilda$LU==11] <- "Urban"
extinct_hilda$LU[extinct_hilda$LU==22] <- "Cropland"
extinct_hilda$LU[extinct_hilda$LU==33] <- "Pasture / Rangeland"
extinct_hilda$LU[extinct_hilda$LU==44] <- "Forest"
extinct_hilda$LU[extinct_hilda$LU==55] <- "Unmanaged Grass / Shrubland"
extinct_hilda$LU[extinct_hilda$LU==66] <- "Sparse / No Vegetation"
colnames(extinct_hilda) = gsub("_states_GLOB.v1.0_wgs84.nn", "", colnames(extinct_hilda))
colnames(extinct_hilda) = gsub("_states_GLOB.v1.0_base.map_wgs84.nn", "", colnames(extinct_hilda))

# ratios
extinct_hilda_r <- extinct_hilda
total <- colSums(extinct_hilda[, -1])
# Loop each year
for (year in colnames(extinct_hilda_r)[-1]) {
  extinct_hilda_r[[paste0("ratio_", year)]] <- extinct_hilda[[year]] / total[year]
  }
# save
write.csv(extinct_hilda_r, file = "extinct_hilda_r.csv", quote = F, row.names = F)

#'[ extant ]
# reading in the shapefile
buff_shape <- terra::vect("extant50/extant50.shp")

# reading in the vars
var_list <- raster::stack(list.files("D:/ENM/Variables/HILDA_LandUse/hildap_vGLOB-1.0_geotiff_wgs84/hildap_GLOB-v1.0_lulc-states", pattern = ".tif",
                                     full.names = TRUE))
rvars <- terra::rast(var_list)
# crs for terra::crop
terra::same.crs(buff_shape, rvars)

buff_cmrvars <- terra::crop(rvars, buff_shape, mask = TRUE)
r <- raster::brick(buff_cmrvars)
plot(r[[1]])

# aggregate
a <- lapply(1:nlayers(r), function(x) {
  ar <- raster::aggregate(raster::getValues(raster::area(r[[x]], weights=FALSE)), by=list(raster::getValues(r[[x]])), sum)
  names(ar) <- c("LU", names(r)[x])
  ar 
  })
# combine years
all <- a[[1]]
for (year in a[-1]) {
  all <- merge(all, year, by = "LU", all=T)
  }

extant_hilda <- all
extant_hilda <- extant_hilda[which(extant_hilda$LU!=c(0, 77, 99)), ] 

extant_hilda$LU[extant_hilda$LU==11] <- "Urban"
extant_hilda$LU[extant_hilda$LU==22] <- "Cropland"
extant_hilda$LU[extant_hilda$LU==33] <- "Pasture / Rangeland"
extant_hilda$LU[extant_hilda$LU==44] <- "Forest"
extant_hilda$LU[extant_hilda$LU==55] <- "Unmanaged Grass / Shrubland"
extant_hilda$LU[extant_hilda$LU==66] <- "Sparse / No Vegetation"

colnames(extant_hilda) = gsub("_states_GLOB.v1.0_wgs84.nn", "", colnames(extant_hilda))
colnames(extant_hilda) = gsub("_states_GLOB.v1.0_base.map_wgs84.nn", "", colnames(extant_hilda))

# ratios
extant_hilda_r <- extant_hilda
total <- colSums(extant_hilda[, -1])
# Loop each year
for (year in colnames(extant_hilda_r)[-1]) {
  extant_hilda_r[[paste0("ratio_", year)]] <- extant_hilda[[year]] / total[year]
}
# save
write.csv(extant_hilda_r, file = "extant_hilda_r.csv", quote = F, row.names = F)


# extinct
extinct_hilda_r <- read.csv("extinct_hilda_r.csv", row.names = NULL)
extinct_hilda_r <- extinct_hilda_r[, -grep("^hilda_plus", colnames(extinct_hilda_r))]
extinct <- tidyr::pivot_longer(extinct_hilda_r, cols = starts_with("ratio_hilda_plus"), names_to = "year", values_to = "value")
extinct$status <- "extinct"
# View(extinct)
# extant
extant_hilda_r <- read.csv("extant_hilda_r.csv", row.names = NULL)
extant_hilda_r <- extant_hilda_r[, -grep("^hilda_plus", colnames(extant_hilda_r))]
extant <- tidyr::pivot_longer(extant_hilda_r, cols = starts_with("ratio_hilda_plus"), names_to = "year", values_to = "value")
extant$status <- "extant"
# View(extant)
# combine extant and extinct 
alllu <- rbind(extinct, extant)
alllu$year <- gsub("^ratio_hilda_plus_", "", alllu$year) ; head(alllu)
alllu$year <- as.numeric(alllu$year) ; head(alllu)
# list of lu classes
hilda_class <- as.list(unique(alllu$LU))
```

## 🟧 Linearity ####
```r
library(lmtest)

for (class in hilda_class) {
    temp <- alllu[which(alllu$LU == class), ]
    model_fit <- lm(data=temp, value ~ year + status)
    message(cat(c(class, paste(raintest(model_fit))), sep = "\n", 
                file= "linearity_Results.txt", append=T))
}
```

## 🟧 Normality w/ qqplot ####
```r
library(ggplot2)
library(grid)
hilda_class <- as.list(unique(alllu$LU))
plot_list <- list()

for (class in hilda_class) {
    lu <- class
    # shapiro-wilk test
    extinct <- alllu[alllu$status == "extinct" & alllu$LU == lu, ]
    extant <- alllu[alllu$status == "extant" & alllu$LU == lu, ]
    
    shapiro.test(extinct$value)
    shapiro.test(extant$value)
    
    p1_st1 <- capture.output(shapiro.test(extinct$value))
    p1_st2 <- capture.output(shapiro.test(extant$value))
    
    library(gridExtra)
    # qq-plot
    p1 <- ggplot(extinct, aes(sample = value)) +
        geom_qq() +
        geom_qq_line() +
        ylab("")  +
        xlab("") +
        labs(title = paste(lu),
             subtitle = paste0("Extinct Shapiro-Wilk: ", grep("^W" , p1_st1, value = T),
                               "\nExtant Shapiro-Wilk: ", grep("^W" , p1_st2, value = T))) +
        theme(plot.subtitle = element_text(size = 8, hjust = 1, face = 3))
    p2 <- ggplot(extant, aes(sample = value)) +
        geom_qq() +
        geom_qq_line() +
        ylab("")  +
        xlab("") +
        labs(title = paste(lu),
             subtitle = paste0("Extinct Shapiro-Wilk: ", grep("^W" , p1_st1, value = T),
                               "\nExtant Shapiro-Wilk: ", grep("^W" , p1_st2, value = T))) +
        theme(plot.subtitle = element_text(size = 8, hjust = 1, face = 3))
    
    grobname <- paste(lu, "_grob")
    grobname2 <- grid.arrange(p1,  p2,
                              nrow = 1,
                              top = textGrob("Test of Normality (Quantile-Quantile)", gp = gpar(fontface = 2, fontsize = 12), hjust = 2, x = 1)
    )
    plot_list <- append(plot_list, list(c = grobname2))
}

grid.arrange(grobs = plot_list, ncol = 2)
#
```

## 🟧 Non parametric McSweeny and Porter ANCOVA ####
```r
# looping
for (class in hilda_class) {
    lu2 <- alllu[which(alllu$LU == class), ]
    res <- npANCOVA::McSweeny_Porter(data = lu2, 
                                     formula = value ~ year + status) # npANCOVA
    lb <- capture.output(cat(rep("# ", 20)))
    ftbl <- capture.output(print(res)) # full table
    ge <- capture.output(print(res$group_effect)) # group effect
    ie <- capture.output(print(res$interaction_effect)) # interaction effect
    cat(c(lb, paste("###:", class), lb, paste("FULL RESULTS:"), ftbl, lb, 
          paste("GROUP EFFECT:"), ge, lb, paste("INTERACTION EFFECT:"), ie), 
        sep = "\n", file= "npANCOVA_Results.txt", append=T)
    assign(paste0("res_", class), res)
    year_breaks <- seq(1900, 2020, by = 10)
    x_breaks <- seq(1, 250, length.out = length(year_breaks))
    plot <- ggplot(res[["data"]], aes(x = ranked_year, y = ranked_response, colour = status)) +
        geom_point(alpha = 0.5) +
        geom_smooth(method = "lm", se = TRUE) +
        labs(x = "Year", y = "Ranked L.U. Ratio", title = class) +
        scale_color_manual(values =c(extant="#ffb000", extinct="#648fff")) +
        guides(size="none") +
        scale_x_continuous(limits = c(1, 250), breaks = x_breaks, labels = ifelse(year_breaks %in% c(1900, 1950, 2000), year_breaks, "")) +
        theme_minimal() +
        theme(panel.grid = element_blank(),
              axis.ticks = element_line(linewidth = 0.5, colour="grey"),
              axis.text = element_text(size=19, colour = "#909090"),
              axis.title = element_text(size=21, face="bold", colour = "black"),
              legend.title = element_text(size=21),
              legend.position = "none",
              legend.text = element_text(size=18),
              title = element_text(size = 20, face="bold"))
    guides(colour = guide_legend(override.aes = list(size = 5)))
    assign(paste0("plot_", class), plot)
}

together <- grid.arrange(plot_Urban,  plot_Cropland, `plot_Pasture / Rangeland`, plot_Forest, `plot_Unmanaged Grass / Shrubland`, `plot_Sparse / No Vegetation`, nrow = 2)
ggsave(filename = "lm_plots_allvars.jpeg", plot = together, width = 5200, height = 3500, units='px', dpi = 300)
```

# 🟥 Maxent x Ellipsoid Envelop model ####
```r
setwd("D:/ENM/GamGla/Manu/ellipsenm")

library(terra)
library(sf)

# Final MaxEnt KUENM SDM
sdm <- terra::rast("D:/ENM/GamGla/Manu/MOP_res/MOP10%_Thresh5%_Set5_current_Binary.tif") ; plot(sdm)
# Final ellipsoid model
enm <- terra::rast("D:/ENM/GamGla/Manu/ellipsenm/Threshold_ellipsenm_projections_s1_mve1_gvars/CPH__ellipsenm_projections_s1_mve1_gvars_binned_current_proj.tif")  ; plot(enm)

# occs
cph <- read.csv("CPH__ellipsenm_projections_s1_mve1_gvars/CPH__ellipsenm_projections_s1_mve1_gvars_binned.csv")
combo <- st_as_sf(cph, coords = c('decimalLongitude', 'decimalLatitude'), crs = 4326, remove=F)
combo$sdm <- terra::extract(sdm, combo, ID=F)[[1]]
combo$enm <- terra::extract(enm, combo, ID=F)[[1]]
names(combo)

# vars
countries <- rnaturalearth::ne_countries(scale = "small", returnclass = "sv")
clear <- rgb(0, 0, 255, max = 255, alpha = 0, names = "clear")

# Binary the best from both
sdm <- terra::ifel(sdm >= 1, 1, 0) ; plot(sdm)
enm <- terra::ifel(enm >= 2, 1, 0) ; plot(enm)
# overlap
enm.sdm <- sdm*enm ; plot(enm.sdm, col = c(clear, "red")) ; terra::plot(countries, add=T)
# save raster
raster::writeRaster(enm.sdm, filename = "../SDM.ENM.overlap.tif")
```



