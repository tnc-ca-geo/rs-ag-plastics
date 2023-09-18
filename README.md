# Mapping Agricultural Plastics in California

The Nature Conservancy, CA Oceans

Grad Innovation Intern Project 2023 Summer


## Description

Plastic use in agriculture is ubiquitous. Plastics are cheap and versatile, and various plastic products have been developed to effectively increase productivity and efficiency in food systems around the globe. However, plastics are also a major pollutant and degraded or discarded plastic products can pose a serious threat to human and ecosystem health. Carbon emissions from plastics are set to outpace coal by 2030, with agricultural plastic use specifically forecast to increase by 50% over the coming decade. In response, the FAO has declared “an urgent need to better monitor the quantities of plastic products used and that leak into the environment from agriculture” (FAO 2021).

California is home to 69,600 farms that support 1.2 million jobs and contribute roughly $50 billion to the state’s economy each year (CDFA 2021). With over 400 agricultural commodities produced in the state, California accounts for 13.7% of all US agricultural production (including 36% of all organic production), creates more than 60% of the national value of fruit and nut crops, and is the fifth largest supplier of food and cotton in the world (CDFA 2021). Yet, plastic use in the state’s agricultural sector is not comprehensively monitored. We do not know the total quantity of plastic used, nor do we know where plastics are applied. The application, degradation, and accumulation of agricultural plastics in soil and groundwater in rural areas across the state poses an environmental threat and raises equity and environmental justice concerns. 


### Objectives & Approach
To address this environmental challenge, we propose creating the first proof of concept map of agricultural plastic use in the state. Pairing satellite imagery with sophisticated image processing algorithms has revolutionized the ability to map and monitor plastic use across vast agricultural areas at relatively low cost. Though applying remote sensing tools and techniques to quantify and map agricultural plastics has been done extensively across Europe and Asia and in select locations in North America, these methods have not been applied to California despite its outsized contribution to global agricultural production. 

We have four objectives: 
- To identify where agricultural plastics are located in the state;
- To demonstrate a cost-effective method to monitor agricultural plastic use in CA; 
- To establish a baseline against which to assess the implications of policies directed at agricultural plastics; and
- To understand equity impacts resulting from agricultural plastic pollution in CA communities.

To achieve these objectives, we will leverage multiple open-source datasets and information types to map and monitor plastic-covered greenhouse farming areas and plastic-mulched farmlands across California. Specifically, we will conduct a phased mapping exercise using available data from published studies, gray literature, and commercial satellite sensors: 

- Statewide / county level / relative use: use published information from the California Department of Food and Agriculture and elsewhere to map crop type by county (Department of Water Resources; crop map); leverage knowledge of relative plastic use by crop type to create a first cut map of agricultural plastic hotspots;
- Statewide / county level / waste by compound type: use published information on plastic waste by compound type at the county level to identify plastic compounds prevalent in agriculture and create a second relative use map of agricultural plastic use;
- Statewide/ hyper-local / detailed use map: use SENTINEL-1 & 2 synthetic aperture radar (SAR) and multispectral sensor imagery via Google Earth Engine and imaging processing algorithms to create a detailed map of plastic-covered greenhouse farming areas and plastic-mulched farmlands.


## Project #1: Crop Type-Based

### Description
Calculate ag plastic use in California based on the spatial distribution of crop types in 2019. 

### Dependencies

* pandas
* geopandas
* matplotlib
* glob
* os

### Installing

* Recommend to configure the environment using [Anaconda](https://www.anaconda.com/).


### Input
* Download spatial dataset [i15_Crop_Mapping_2019](https://gis.data.ca.gov/datasets/363c00277ad74c4ba4f64238edc5430c_0/about).
* Extract crop of interest shp files into a new folder called "i15_Crop_Mapping_2019_extracted". E.g., to extract Strawberries, use this filter: 'MAIN_CROP' = 'T20' (crop type code reference: https://gis.water.ca.gov/arcgis/rest/services/Planning/i15_Crop_Mapping_2019/MapServer/0?f=pjson)
* Prepare "input_from_ERG.csv" to adopt calculation method from the ERG project.
  
### Executing Program
* Run code: [plastic_mapping_code.ipynb](https://github.com/Yuanyuan-T/TNC-Ag-Plastic/blob/main/plastic_mapping_code.ipynb)

### Output
* "map": jpg file of maps;
* "shp": spatial data to visualize each map

Other files:
- "Ag-knowledge-ChatGPT-mulching-greenhouse-tunnel.md": knowledge extracted from GPT 4;
- "plastic_mapping_output.ipynb": maps in the code;
- "output/Agricultural Plastic Mapping in California.pdf": pdf file for general public 


## Project #2: RS + ML

### Description
Use remote sensing and machine learning to build a model for ag plastic assessment in Oxnard, CA in 2019. 

### Input
(1) Create labeling dataset: visual interpretation (GEE, Earth Engine, GoogleMaps POI and historical street view). Use GEE to draw multipolygons such as

<img width="300" alt="image" src="https://github.com/Yuanyuan-T/TNC-Ag-Plastic/assets/43053656/a9527320-dc8f-4997-931e-b913da8e9d7d">

then assign the class, e.g. 
```
// Make a FeatureCollection from the hand-made geometries.
var polygons = ee.FeatureCollection([
  ee.Feature(hoop_1, {'class': 0}),
  ee.Feature(hoop_2, {'class': 0}),
  ee.Feature(hoop_3, {'class': 0}),
  ee.Feature(mulch_1, {'class': 1}),
  ee.Feature(mulch_2, {'class': 1}),
  ee.Feature(mulch_3, {'class': 1}),
```
then export the feature collection as shp,
```
Export.table.toDrive({
  folder: "GEE data/TNC",
  collection:polygons,
  description:'label_mulch_hoop',
  fileFormat:'SHP',
  fileNamePrefix:'label_mulch_hoop'
});
```

Finally, download the shp from Google Drive, and upload it to GEE asset. See instruction: https://developers.google.com/earth-engine/guides/table_upload

In asset (must-have):
- label_mulch_hoop, 
- label_nonplastic,

(2) Prepare AOI

In asset (must-have):
- Oxnard: boundary of Oxnard city.  
- TruckCrops: extract from crop dataset. Filter: 'Class2' = 'T'.

(3) Crop type shp as validation

In asset (optional):
- Strawberries_CA, 
- Bushberries_T19, 
- Flowers_T16, 
- Pepers_T21, 
- Avocados, 

### Executing Program
* Run code: [Project2-GEE-code.md](https://github.com/Yuanyuan-T/TNC-Ag-Plastic/blob/main/Project2-GEE-code.md)
(Please adjust the file dir path and name if needed.)

## Authors

Contributors names and contact info
  
[@Yuanyuan Tian](https://www.linkedin.com/in/yuanyuan-tian-geo/): yuanyuantian@asu.edu

[@Darcy Bradley](darcy.bradley@tnc.org): darcy.bradley@tnc.org

## Version History

* 0.1
    * Initial share (2023/08/14)

## License

This project is private now.

This project is licensed under the [NAME HERE] License - see the LICENSE.md file for details

## Acknowledgments
Special thanks to Oceans Plastic team and Kirk R. Klausmeyer <kklausmeyer@TNC.ORG>.
