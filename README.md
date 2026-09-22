# ENM Gampsocleis
|             |                                                                                                                                                                                                   |
|:-----------:|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Author**: | Nina Casillas                                                                                                                                                                                     |
|  **Info**:  | Scripts used to model the ecological niche of *Gampsocleis glabra* in the manuscript "Climatic marginality and anthropogenic habitat change jointly shape extinction patterns in the threatened steppic species *Gampsocleis glabra* (Orthoptera: Ensifera)" |

# Table of Contents
<details>
<summary>🟥 <a href="https://github.com/NACasillas/ENM_Gampsocleis/blob/main/GamGla.ENM.md#-clean-occurrences">Clean Occurrences</a> </summary>  
&nbsp;&nbsp;&nbsp;🟧 C1 - Dedup, remove uncertain<br>
&nbsp;&nbsp;&nbsp;🟧 C2 - Move occs outside of soil rasters to nearest pixel<br>
&nbsp;&nbsp;&nbsp;🟧 C3 - Thinning<br>
</details>

<details>
<summary>🟥 <a href="https://github.com/NACasillas/ENM_Gampsocleis/blob/main/GamGla.ENM.md#-variables">Variables</a> </summary>
&nbsp;&nbsp;&nbsp;🟧 Cropping Vars  
</details>

<details>
<summary>🟥 <a href="https://github.com/NACasillas/ENM_Gampsocleis/blob/main/GamGla.ENM.md#-kuenm">KUENM</a> </summary>
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
<summary>🟥 <a href="https://github.com/NACasillas/ENM_Gampsocleis/blob/main/GamGla.ENM.md#-ellipsoid-envelope-model-with-ellipsenm">Ellipsoid envelope model with ellipsenm</a> </summary>
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
<summary>🟥 <a href="https://github.com/NACasillas/ENM_Gampsocleis/blob/main/GamGla.ENM.md#-land-use">Land Use</a> 
</summary>
&nbsp;&nbsp;&nbsp;🟧 Buffer groups and crop land use variables<br>
&nbsp;&nbsp;&nbsp;🟧 Linearity<br>
&nbsp;&nbsp;&nbsp;🟧 Normality w/ qqplot<br>
&nbsp;&nbsp;&nbsp;🟧 Non parametric McSweeny and Porter ANCOVA<br>
</details>
🟥 <a href="https://github.com/NACasillas/ENM_Gampsocleis/blob/main/GamGla.ENM.md#-maxent-x-ellipsoid-envelop-model">Maxent x Ellipsoid Envelop model</a> 


***

