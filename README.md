# Drought-analyisis-data-and-results
In this repository, you are able to see the data and results regarding the drought analysis in Iran. More instructions are already provided in the article
///This code is meant for the classification of the country. To apply this in your own country, add the shape file in .shp format. 
Map.centerObject(country);

var lst_day = ee.ImageCollection("MODIS/006/MOD11A2")
.filterBounds(country)
.filterDate('2000-01-01','2011-01-01')
.select('LST_Day_1km')
.mean()
.multiply(0.02)
.subtract(273.15)
.clip(country);

var lst_night = ee.ImageCollection("MODIS/006/MOD11A2")
.filterBounds(country)
.filterDate('2000-01-01','2011-01-01')
.select('LST_Night_1km')
.mean()
.multiply(0.02)
.subtract(273.15)
.clip(country);

var LST = lst_day.add(lst_night).divide(2.0).rename('LST');

//Map.addLayer(LST,{},'LST');

var TRMM =ee.ImageCollection("UCSB-CHG/CHIRPS/DAILY")
.filterBounds(country)
.filterDate('2000-01-01','2011-01-01')
.select('precipitation');

var years = ee.List.sequence(2010,2020);
var image_year = ee.ImageCollection.fromImages(
  years.map(function(y){
    
    return TRMM.filter(ee.Filter.calendarRange(y,y,'year'))
    .sum()
    .set('year',y)
  }));
  
print(image_year,'image_year');  
var mean_TRMM =image_year.mean().clip(country);

//Map.addLayer(mean_TRMM,{},'mean_TRMM');

//Dormaton

var dataset = ee.Image.cat(LST,mean_TRMM);
print(dataset,'dataset');

var domarton = dataset.expression(
  'p/(t+10)',{
    'p':dataset.select('precipitation'),
    't':dataset.select('LST')
  });
  

//Map.addLayer(domarton,{},'domarton');

var thr = domarton.where(domarton.lt(10),1);
var thr1 = thr.where(domarton.gte(10).and(domarton.lt(20)),2);
var thr2 = thr1.where(domarton.gte(20).and(domarton.lt(24)),3);
var thr3 = thr2.where(domarton.gte(24).and(domarton.lt(28)),4);
var thr4 = thr3.where(domarton.gte(28).and(domarton.lt(35)),5);
var thr5 = thr4.where(domarton.gte(35),6);

Map.addLayer(thr5.randomVisualizer(),{},'De Martonne');
Export.image.toDrive({
  image:thr5,
  description:'thr5_domarton',
  scale:6000,
  region:country,
  maxPixels:1e9
});



