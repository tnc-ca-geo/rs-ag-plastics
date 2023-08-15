# Project 2: RS + ML

## GEE code
Copy and paste the code into GEE code editor to run.

```
//-----------------------------------
// Project 2: monitor the ag plastic in Oxnard, CA
// should be validated with data in 2019
//-----------------------------------

// Define the date range
var START = '2019-02-01';
var END = '2019-06-01';
var Boundary_Oxnard = ee.FeatureCollection('projects/ee-tianyuanyuan/assets/tnc/Oxnard');


//-----------------------------------
//-----------load shp data
//-----------------------------------
var Strawberries_CA = ee.FeatureCollection('projects/ee-tianyuanyuan/assets/tnc/Strawberries_CA');
var vis_params_Strawberries = {
    'color': 'red', 
    'pointSize': 3,
    'pointShape': 'circle',
    'width': 3,
    'lineType': 'solid',
    'fillColor': '00000000',
};

var Bushberries_CA = ee.FeatureCollection('projects/ee-tianyuanyuan/assets/tnc/Bushberries_T19');
var vis_params_Bushberries = {
    'color': 'orange', 
    'pointSize': 3,
    'pointShape': 'circle',
    'width': 3,
    'lineType': 'solid',
    'fillColor': '00000000',
};

var Flowers_CA = ee.FeatureCollection('projects/ee-tianyuanyuan/assets/tnc/Flowers_T16');
var vis_params_Flowers = {
    'color': 'purple', 
    'pointSize': 3,
    'pointShape': 'circle',
    'width': 3,
    'lineType': 'solid',
    'fillColor': '00000000',
};

var Pepers_CA = ee.FeatureCollection('projects/ee-tianyuanyuan/assets/tnc/Pepers_T21');
var vis_params_Pepers = {
    'color': 'red', 
    'pointSize': 3,
    'pointShape': 'circle',
    'width': 3,
    'lineType': 'solid',
    'fillColor': '00000000',
};

var Avacados_CA = ee.FeatureCollection('projects/ee-tianyuanyuan/assets/tnc/Avocados');
var vis_params_Avacados = {
    'color': 'Grey', 
    'pointSize': 3,
    'pointShape': 'circle',
    'width': 3,
    'lineType': 'solid',
    'fillColor': '00000000',
};

// Load labeled feature collections from the hand-made geometries.
var polygons_label_mulch_hoop = ee.FeatureCollection('projects/ee-tianyuanyuan/assets/tnc/tnc-training-data/label_mulch_hoop');
var polygons_label_nonplastic = ee.FeatureCollection('projects/ee-tianyuanyuan/assets/tnc/tnc-training-data/label_nonplastic');
var polygons = polygons_label_mulch_hoop.merge(polygons_label_nonplastic);


// var combined_aoi = Strawberries_CA.merge(Melon_CA);
// var combined_aoi = Strawberries_CA.merge(Melon_CA).merge(Pepers_CA).merge(Flowers_CA);
var aoi_truck_crop = ee.FeatureCollection('projects/ee-tianyuanyuan/assets/tnc/TruckCrops');
// var Boundary_CA = ee.FeatureCollection('projects/ee-tianyuanyuan/assets/tnc/CA_Counties_TIGER2016') // clip is very slow
var Boundary_Oxnard = ee.FeatureCollection('projects/ee-tianyuanyuan/assets/tnc/Oxnard');
var aoi = aoi_truck_crop;

//-----------------------------------//
//-----------load RS data
//-----------------------------------//

//////////////sentinel 
/**
* Function to mask clouds using the Sentinel-2 QA band
* @param {ee.Image} image Sentinel-2 image
* @return {ee.Image} cloud masked Sentinel-2 image
*/
function maskS2clouds(image) {
  var qa = image.select('QA60');

  // Bits 10 and 11 are clouds and cirrus, respectively.
  var cloudBitMask = 1 << 10;
  var cirrusBitMask = 1 << 11;

  // Both flags should be set to zero, indicating clear conditions.
  var mask = qa.bitwiseAnd(cloudBitMask).eq(0)
      .and(qa.bitwiseAnd(cirrusBitMask).eq(0));

  return image.updateMask(mask).divide(10000);
}


// Define a function to calculate indices and create a processed image collection
var processImageCollection = function(imageCollection, aoi, startDate, endDate) {
  // Define cloud masking function
  var maskS2clouds = function(image) {
    // Cloud masking code
    return image;
  };
  
  // Add NDVI band
  var addNDVI = function(image) {
    var ndvi = image.normalizedDifference(['B8', 'B4']).rename('NDVI');
    return image.addBands(ndvi);
  };
  
  // Add NDTI band
  var addNDTI = function(image) {
    var ndti = image.normalizedDifference(['B11', 'B12']).rename('NDTI');
    return image.addBands(ndti);
  };
  
  // Add PGI band
  var addPGI = function(image) {
    var NIR = image.select('B8');
    var R = image.select('B4');
    var G = image.select('B3');
    var B = image.select('B2');
    var pgi = NIR.subtract(R).multiply(100).multiply(R)
      .divide(NIR.add(B).add(G).divide(3).subtract(1))
      .rename('PGI');
    return image.addBands(pgi);
  };
  
  // Add PMLI band
  var addPMLI = function(image) {
    var SWIR1 = image.select('B11');
    var R = image.select('B4');
    var pmli = SWIR1.subtract(R)
      .divide(SWIR1.add(R))
      .rename('PMLI');
    return image.addBands(pmli);
  };
  
  // Add RPGI band
  var addRPGI = function(image) {
    var NIR = image.select('B8');
    var R = image.select('B4');
    var G = image.select('B3');
    var B = image.select('B2');
    var rpgi = B.multiply(100)
      .divide(NIR.add(B).add(G).divide(3).subtract(1))
      .rename('RPGI');
    return image.addBands(rpgi);
  };

  // Apply all processing steps to the image collection
  var processedCollection = imageCollection
    .filterDate(startDate, endDate)
    .filterBounds(aoi)
    .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 20))
    .map(maskS2clouds)
    .map(addNDVI)
    .map(addNDTI)
    .map(addPGI)
    .map(addPMLI)
    .map(addRPGI);
  
  return processedCollection;
};


// Call the processImageCollection function
var processedCollection = processImageCollection(ee.ImageCollection('COPERNICUS/S2_SR'), Boundary_Oxnard, START, END);

// Get the median image from the processed collection and clip it to the AOI
var image = processedCollection.median().clip(aoi).clip(Boundary_Oxnard);


// Use these bands and festures for prediction.
// refer: Combining Multi-Source Data and Feature Optimization for Plastic-Covered Greenhouse Extraction and Mapping Using the Google Earth Engine : A Case in Central Yunnan
var bands = [
  // Spectrum Features
  'B4', 'B3', 'B2', 'B6', 'B8', 'B11', 'B12', 
  // Index Features
  'NDVI', 'NDTI', 'PGI', 'PMLI',
  ];



//-----------------------------------//
//-----------prepare training data
//-----------------------------------//

// Get the values for all pixels in each polygon.
var data = image.sampleRegions({
  // Get the sample from the polygons FeatureCollection.
  collection: polygons,
  // Keep this list of properties from the polygons.
  properties: ['class'],
  // Set the scale to get Landsat pixels in the polygons.
  scale: 30,
  geometries: true, // If true, the results will include a point geometry per sampled pixel. Otherwise, geometries will be omitted (saving memory).
});
print('total number of labeled data: ', data.size());

//-------------------- process data (1): split---------------------
// The randomColumn() method will add a column of uniform random
// numbers in a column named 'random' by default.
data = data.randomColumn({
  seed: 0 // 
  // seed: 1 
  // seed: 2 
  });

var split = 0.7;  // Roughly 70% training, 30% testing.
var training = data.filter(ee.Filter.lt('random', split));
var validation = data.filter(ee.Filter.gte('random', split));


// polygons.evaluate(function(result){
//     console.log(result); // print the entire FeatureCollection properties
// });
print('number of training: ', training.size());
print('number of validation: ', validation.size());

//-------------------- process data (2): address autocorrelation correction---------------------
// Spatial join.
var distFilter = ee.Filter.withinDistance({
  distance: 30, // unit: meter. 
  leftField: '.geo',
  rightField: '.geo',
  maxError: 10
});

var join = ee.Join.inverted();

// Apply the join.
training = join.apply(training, validation, distFilter);
print('training after remove autocorrelation: ', training);


//-----------------------------------//
//-----------prepare model
//-----------------------------------//

//-----------model 1: SVM-----------//
// Create an SVM classifier with custom parameters.
var classifier_SVM = ee.Classifier.libsvm({
  kernelType: 'RBF',
  gamma: 0.5,
  cost: 10
});

// Train the classifier.
var trained_SVM = classifier_SVM.train(training, 'class', bands);

// Classify the image.
var classified_SVM = image.classify(trained_SVM);


//-----------model 2: CART-----------//
// Train a CART classifier with default parameters.
var classifier_CART = ee.Classifier.smileCart({
});

// Train the classifier.
var trained_CART = classifier_CART.train(training, 'class', bands);

// Classify the image with the same bands used for training.
var classified_CART = image.select(bands).classify(trained_CART);

//-----------model 3: RF-----------//
// Make a Random Forest classifier and train it.
var classifier_RF = ee.Classifier.smileRandomForest(50) 

// Train the classifier.
var trained_RF = classifier_RF.train(training, 'class', bands);

// Classify the input imagery.
var classified_RF = image.select(bands).classify(trained_RF);


// resubstitution error matrix is also called confusion matrix)

//-----------accuracy for model 1: SVM-----------//
// Get a confusion matrix representing resubstitution accuracy.
var trainAccuracy_SVM = trained_SVM.confusionMatrix();
print('SVM Resubstitution error matrix: ', trainAccuracy_SVM);
print('SVM Training overall accuracy: ', trainAccuracy_SVM.accuracy());
// Validation
var validated_SVM = validation.classify(trained_SVM);
var testAccuracy_SVM = validated_SVM.errorMatrix('class', 'classification');
print('SVM Validation error matrix: ', testAccuracy_SVM);
print('SVM Validation overall accuracy: ', testAccuracy_SVM.accuracy());

//-----------accuracy for 2: CART-----------//
// Get a confusion matrix representing resubstitution accuracy.
var trainAccuracy_CART = trained_CART.confusionMatrix();
print('CART Resubstitution error matrix: ', trainAccuracy_CART);
print('CART Training overall accuracy: ', trainAccuracy_CART.accuracy());
// Validation
var validated_CART = validation.classify(trained_CART);
var testAccuracy_CART = validated_CART.errorMatrix('class', 'classification');
print('CART Validation error matrix: ', testAccuracy_CART);
print('CART Validation overall accuracy: ', testAccuracy_CART.accuracy());


//-----------accuracy for model 3: RF-----------//
// Get a confusion matrix representing resubstitution accuracy.
var trainAccuracy_RF = trained_RF.confusionMatrix();
print('RF Resubstitution error matrix: ', trainAccuracy_RF);
print('RF Training overall accuracy: ', trainAccuracy_RF.accuracy());
// Validation
var validated_RF = validation.classify(trained_RF);
var testAccuracy_RF = validated_RF.errorMatrix('class', 'classification');
print('RF Validation error matrix: ', testAccuracy_RF);
print('RF Validation overall accuracy: ', testAccuracy_RF.accuracy());


// //-----------------------------------//
// //-----------aggregation statistics: one polygon as boundary
// // take ~10 seconds
// //-----------------------------------//


// ref: https://courses.spatialthoughts.com/end-to-end-gee-supplement.html#calculating-area-by-class
var classified = classified_RF;
var geometry = Boundary_Oxnard.geometry();
Map.addLayer(geometry, {}, "Oxnard");

var palette = ['yellow', 'pink', 'green'];


// We can calculate the areas of all classes in a single pass
// using a Grouped Reducer. Learn more at 
// https://spatialthoughts.com/2020/06/19/calculating-area-gee/

// First create a 2 band image with the area image and the classified image
// // Divide the area image by 1e6 so area results are in Sq Km
// var areaImage = ee.Image.pixelArea().divide(1e6).addBands(classified);
// // Divide the area image by 4046.86 so area results are in acre
var areaImage = ee.Image.pixelArea().divide(4046.86).addBands(classified);

// Calculate areas
var areas = areaImage.reduceRegion({
      reducer: ee.Reducer.sum().group({
      groupField: 1,
      groupName: 'classification',
    }),
    geometry: geometry,
    scale: 10,
    maxPixels: 1e10
    }); 
 
var classAreas = ee.List(areas.get('groups'))

// Process results to extract the areas and
// create a FeatureCollection

// We can define a dictionary with class names
var classNames = ee.Dictionary({
  '0': 'hoop',
  '1': 'mulch',
  '2': 'non-plastic truck crop',
})

//--------!! the most important function
var classAreas = classAreas.map(function(item) {
  var areaDict = ee.Dictionary(item)
  var classNumber = ee.Number(areaDict.get('classification')).format();
  var className = classNames.get(classNumber);
  var area = ee.Number(
    areaDict.get('sum'))
  return ee.Feature(null, {'class': classNumber, 'class_name': className, 'area': area})
})

var classAreaFc = ee.FeatureCollection(classAreas);
// print('classAreaFc',classAreaFc)

// We can now chart the resulting FeatureCollection
// If your area is large, it is advisable to first Export
// the FeatureCollection as an Asset and import it once
// the export is finished.
// Let's create a Bar Chart
var areaChart = ui.Chart.feature.byProperty({
  features: classAreaFc,
  xProperties: ['area'],
  seriesProperty: 'class_name',
}).setChartType('ColumnChart')
  .setOptions({
    hAxis: {title: 'Classes'},
    vAxis: {title: 'Area (acre)'},
    title: 'Area by class',
    series: {
      0: { color: 'yellow' },
      1: { color: 'pink' },
      2: { color: 'green' },
    }
  });
print(areaChart); 

// We can also create a Pie-Chart
var areaChart = ui.Chart.feature.byFeature({
  features: classAreaFc,
  xProperty: 'class_name',
  yProperties: ['area']
}).setChartType('PieChart')
  .setOptions({
    hAxis: {title: 'Classes'},
    vAxis: {title: 'Area (acre)'},
    title: 'Area by class',
    colors: palette
  });
print(areaChart); 






//-----------------------------------//
//-----------visualization
//-----------------------------------//
Map.setCenter(-119.11901884, 34.1661622, 11); //overview: 11, detail: 16

//sentinel
Map.addLayer(image,
            {bands: ['B4', 'B3', 'B2'], min: 0, max: 0.25},
            'sentinel image', 
            false);
           

// Display the classification result
Map.addLayer(classified_RF,
            {min: 0, max: 2, palette: ['yellow', 'pink', 'green']},
            'classification result (RF)');   

            
Map.addLayer(polygons, {color: 'blue'}, 'labeled polygons', false);            
Map.addLayer(training, {color: 'blue'}, 'training points', false);
Map.addLayer(validation, {color: 'green'}, 'validation points', false);  

Map.addLayer(Flowers_CA.style(vis_params_Flowers), {}, "Flowers", false);
Map.addLayer(Pepers_CA.style(vis_params_Pepers), {}, "Pepers", false);
Map.addLayer(Avacados_CA.style(vis_params_Avacados), {}, "Avacados", false);
Map.addLayer(Bushberries_CA.style(vis_params_Bushberries), {}, "Bushberries", false);
Map.addLayer(Strawberries_CA.style(vis_params_Strawberries), {}, "Strawberries", false);

```

## Expected result:

RF accuracy: 90%+, plastic covering ~50% of truck crop farming area in Oxnard. 
<img width="1372" alt="image" src="https://github.com/Yuanyuan-T/TNC-Ag-Plastic/assets/43053656/d5d0b521-045d-4d0c-8725-178f5a2533ef">
