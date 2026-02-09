# Mapping Agricultural Plastics in California
This repo contains the processing scripts related to the research article _Fields of plastic: Tracking the growing footprint of agricultural plastics using remote sensing in California._ One script in particular was used to run all principal analyses including training and testing the model and analyzing the results. This repo is published with a doi on zenodo: ___________. To cite this work, please use the zenodo doi. 

### Project Description
Globally, plastic use in agriculture has become ubiquitous. While plastics offer many benefits such as increased crop productivity and water-use efficiency, degraded or discarded plastic products can pose a serious threat to soil productivity, as well as ecosystem and human health. California is the largest agricultural producer in the United States, yet the use of agricultural plastics is not monitored in the state. We used Sentinel-2 multispectral satellite imagery and satellite-derived evapotranspiration data to model and map field plastics (plastic mulch and hoop houses) across six years in three key agricultural counties in California. The scripts in this repository outline our multi-temporal approach. This work spun off from Yuanyuan Tian's Grad Innovation Intern Project with The Nature Conservancy from the summer of 2023. 

### Results
Our multi-temporal approach demonstrated high model test accuracy (99% for hoop houses, 88% for plastic mulch). We found widespread and increasing field plastic use over time, with nearly a third of agricultural areas covered by field plastic during the 2024 water year. Plastic mulch comprised 69% of all field plastics detected over the study period, and was often detected on cropland adjacent to major waterways. We also identified fields with high cumulative plastic exposure, which can inform in-situ research on soil health impacts. In California, this research provides valuable insights for agencies to better track and manage the rapid growth of plastic use in agriculture, with the ultimate goal of improving end-of-life management, reducing plastic pollution, and improving ecosystem and soil health in agricultural regions. Finally, these methods are designed to be applicable across a variety of cropping practices and therefore advance field plastic detection capabilities for many regions globally. 

### Objectives & Approach
This project aimed to achieve the following objectives:
- Identify the extent of field plastics (plastic mulch and hoop houses) across six water years within three counties;
- Establish a baseline against which to assess the implications of policies directed at field plastics; and
- Demonstrate an effective method to monitor field plastics across a large area with dynamic cropping practices.

Our approach leverages open source datasets and cloud-computing through the Google Earth Engine Python API. 
Figure 2 from 


### Running the code
The script ______________ contains all core analysis and processing. 

#### Dependencies
User must enter their own Earth Engine account and associated cloud project to run the script (authentification to the author's EE account will fail). 

* ee
* geemap
* pandas
* geopandas
* numpy
* matplotlib
* sklearn
* altair
* seaborn

#### Data
* Download training data from this repository or the associated zenodo repository here:
* All other data are accessible through the EE catalog
  
#### Outputs
* Export resulting rasters as tifs to Drive or EE assets

### Authors
All authors are listed in the associated publication.
* Main code author: [Annie Taylor](mailto:annie.taylor@tnc.org)
* Previous code contributor: [@Yuanyuan Tian](https://www.linkedin.com/in/yuanyuan-tian-geo/): yuanyuantian@asu.edu

### Version History
* Version 1.0 released February 2026.

### License
This project is licensed under the [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/legalcode) - re-distribution and re-use of this licensed work is permitted on the condition that the creator is appropriately credited.

### Acknowledgments
The authors would like to thank the members of the Ocean Plastics Strategy team at the Nature Conservancy California, including Patrick Jurney, Amy Zimmer-Faust, and Alexis Jackson. We are very grateful for the help of Brandee Wagner in annotating training data and advising on that process. Finally, we would like to thank April Chang, Amelia Li, Rebecca Qiu, Peter Wu, and Evan Arnold for their interest and engagement with an early phase of this work through their Harvard John A. Paulson School of Engineering and Applied Sciences Capstone project. 
