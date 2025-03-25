# GLC_FCS30D: global 30m LULC Landsat based map from 1985-2022
**Paper** : https://essd.copernicus.org/articles/16/1353/2024/essd-16-1353-2024.pdf


```javascript
// Example script showing how to pre-process the GLC_FCS30D
// landcover dataset to create an ImageCollection with yearly
// landcover images and recoded class values

// Yearly data from 2000-2022
var annual = ee.ImageCollection(
  'projects/sat-io/open-datasets/GLC-FCS30D/annual');
// Five-Yearly data for 1985-90, 1990-95 and 1995-2000
var fiveyear = ee.ImageCollection(
  'projects/sat-io/open-datasets/GLC-FCS30D/five-years-map');

// The classification scheme has 36 classes 
// (35 landcover class and 1 fill value)
var classValues = [
  10, 11, 12, 20, 51, 52, 61, 62, 71, 72, 81, 82, 91, 92, 120, 121, 122, 
  130, 140, 150, 152, 153, 181, 182, 183, 184, 185, 186, 187, 190, 200, 
  201, 202, 210, 220, 0
];
// Landcover class names
// We removes spaces and special characters from the names
var classNames = [
  'Rainfed_cropland',
  'Herbaceous_cover_cropland',
  'Tree_or_shrub_cover_cropland',
  'Irrigated_cropland',
  'Open_evergreen_broadleaved_forest',
  'Closed_evergreen_broadleaved_forest',
  'Open_deciduous_broadleaved_forest',
  'Closed_deciduous_broadleaved_forest',
  'Open_evergreen_needle_leaved_forest',
  'Closed_evergreen_needle_leaved_forest',
  'Open_deciduous_needle_leaved_forest',
  'Closed_deciduous_needle_leaved_forest',
  'Open_mixed_leaf_forest',
  'Closed_mixed_leaf_forest',
  'Shrubland',
  'Evergreen_shrubland',
  'Deciduous_shrubland',
  'Grassland',
  'Lichens_and_mosses',
  'Sparse_vegetation',
  'Sparse_shrubland',
  'Sparse_herbaceous',
  'Swamp',
  'Marsh',
  'Flooded_flat',
  'Saline',
  'Mangrove',
  'Salt_marsh',
  'Tidal_flat',
  'Impervious_surfaces',
  'Bare_areas',
  'Consolidated_bare_areas',
  'Unconsolidated_bare_areas',
  'Water_body',
  'Permanent_ice_and_snow',
  'Filled_value'
];

var classColors = [
  '#ffff64', '#ffff64', '#ffff00', '#aaf0f0', '#4c7300',
  '#006400', '#a8c800', '#00a000', '#005000', '#003c00',
  '#286400', '#285000', '#a0b432', '#788200', '#966400', 
  '#964b00', '#966400', '#ffb432', '#ffdcd2', '#ffebaf', 
  '#ffd278', '#ffebaf', '#00a884', '#73ffdf', '#9ebb3b', 
  '#828282', '#f57ab6', '#66cdab', '#444f89', '#c31400', 
  '#fff5d7', '#dcdcdc', '#fff5d7', '#0046c8', '#ffffff',
  '#ffffff'
];
  
// The data is split into tiles
// Mosaic them into a single image
var annualMosaic = annual.mosaic();
var fiveYearMosaic = fiveyear.mosaic();

// Each image in five year image 3 bands,
// one for each year from 1985-90, 1990-95 and 1995-2000

// Create a list of year strings
var fiveYearsList = ee.List.sequence(1985, 1995, 5).map(function(year) {
  return ee.Number(year).format('%04d');
});
var fiveyearMosaicRenamed = fiveYearMosaic.rename(fiveYearsList);

// Each image in annual image 23 bands, one for each year from 2000-2022
// Rename bands from b1,b2.. to 2000,2001... etc.
var yearsList = ee.List.sequence(2000, 2022).map(function(year) {
  return ee.Number(year).format('%04d')
})
var annualMosaicRenamed = annualMosaic.rename(yearsList);

var years = fiveYearsList.cat(yearsList);

// Turn the multiband image to a ImageCollection
var fiveYearlyMosaics = fiveYearsList.map(function(year) {
  var date = ee.Date.fromYMD(ee.Number.parse(year), 1, 1);
  return fiveyearMosaicRenamed.select([year]).set({
    'system:time_start': date.millis(),
    'system:index': year,
    'year': ee.Number.parse(year)
  });
});

var yearlyMosaics = yearsList.map(function(year) {
  var date = ee.Date.fromYMD(ee.Number.parse(year), 1, 1);
  return annualMosaicRenamed.select([year]).set({
    'system:time_start': date.millis(),
    'system:index': year,
    'year': ee.Number.parse(year)
  });
});

var allMosaics = fiveYearlyMosaics.cat(yearlyMosaics);
var mosaicsCol = ee.ImageCollection.fromImages(allMosaics);

// For ease of visualization and analysis, it is recommended
// to recode the class values into sequential values
// We use remap() to convert the original values into new values
// from 1 to 36
var newClassValues = ee.List.sequence(1, ee.List(classValues).length());

var renameClasses = function(image) {
  var reclassified = image.remap(classValues, newClassValues)
    .rename('classification');
  return reclassified;
};

var landcoverCol = mosaicsCol.map(renameClasses);

print('Pre-processed Collection', landcoverCol);

// Visualize the data
var year = 2022;
var selectedLandcover = landcoverCol
  .filter(ee.Filter.eq('year', year)).first();

var palette = [
  '#ffff64', '#ffff64', '#ffff00', '#aaf0f0', '#4c7300', '#006400', '#a8c800', '#00a000', 
  '#005000', '#003c00', '#286400', '#285000', '#a0b432', '#788200', '#966400', '#964b00', 
  '#966400', '#ffb432', '#ffdcd2', '#ffebaf', '#ffd278', '#ffebaf', '#00a884', '#73ffdf', 
  '#9ebb3b', '#828282', '#f57ab6', '#66cdab', '#444f89', '#c31400', '#fff5d7', '#dcdcdc', 
  '#fff5d7', '#0046c8', '#ffffff', '#ffffff'
];
var classVisParams = {min:1, max:36, palette: palette};
Map.addLayer(selectedLandcover, classVisParams, 'Landcover ' + year);
print(legend);
```
