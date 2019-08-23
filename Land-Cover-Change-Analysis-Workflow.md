## Step 1: Obtain Remote Sensing Data
Detailed instructions are in Lab Five in the course manual. Summary:
* Save a kml file of the area of interest and load it in Earthdata to look for images (spatial filter).
* To find appropriate tiles in Earthdata, apply collection filter (landsat) and temporal filter (years of interest). To further refine the area of interest, apply a spatial filter by grid code derived from a relevant Landsat tile.
* Download granules and geotiff products.
* Uncompress .tar.gz files.
* Note: Remember that most Landsat images will require patching, so download more than one granule.

## Step 2: Write up Metadata for Granules
* Use tips from Lab 4 and end of Lab 5 for metadata format.

## Step 3: Correct the Images

### Landsat 8
These steps are outlined in Lab 7.

In order to configure the processing framework of Orfeo in QGIS- 
• Go to Processing  Options. 
• Expand the Providers menu to GRASS commands 

• Under settings, expand the providers menu and select the GRASS commands. Once there: delete the value for the Msys folder setting. 

• Then expand the Orfeo Toolbox (image analysis) menu:
◦set the OTB application folder to C:\Orfeo\lib\otb\applications
◦set the OTB command line tools folder to C:\Orfeo\bin. 

• Select OK. Then close and re-open QGIS to apply the changes 
(If the Processing Toolbox is not open yet, go to Processing -> Toolbox and the Orfeo Toolbox should now appear). 

Add the reflective Landsat bands to your QGIS project:
• Exclude the thermal energy band from analysis, by moving bands 10 and 11 into a folder saved as: Thermal folder. 
• Then add band 2 (blue) to the QGIS project.

Remove the background 0’s with the Manage NoData module:
 • In the processing Toolbox, open the Orfeo Toolbox tree.
Go to Image Manipulation -> No Data Management
◦ Set the Input image to B2
◦ Set the No-data handling mode to changevalue
◦ Inside value: 1
◦ Outside value: 0
◦ New no-data value: 0
◦ Nodata value used: 0
◦ Output image: [save to temporary file]
(This should make the output image free of it's background).

Radiometric Correction for Landsat 8:
Use band math to convert to top-of-atmosphere reflectance. 
This formula can be implemented in Orfeo's Band Math module (Miscellaneous  Band
Math) where TOA (Top of Atmosphere) equals:
◦ (REFLECTANCE_MULT_BAND * im1b1 + REFLECTANCE_ADD_BAND) / sin(SUN_ELEVATION)
Preempt the fact that Band Math dos not regard the nodata value with: evaluation ? true formula : false formula

Run a model as a batch script: with your model for calculating top of atmosphere reflectance,correct all of your Landsat 8 reflectance bands at the same time.
• First, create a new folder, TOAR
• Right-click your model and Execute as Batch Process
◦ Select each of your Landsat bands for InputBand from 1 to 9
◦ Set the outputTOAR to B in your TOAR folder. (Autofill with numbers).


### Before Landsat 8
Requires the following: concatenate band images, optical calibration, and manage noData. These steps can be carried out in Orfeo and are outlined in Lab 8 and in the videos L8a_Concatenate, L8b_OpticalCalibration, L8h_RecapOpticalCalibrationMaskingPatching. Summary:

* Image concatenation: Launch OTB applications browser and use Image Manipulation -> ConcatenateImages. Set output to uint 8. Set different RGB composites to check. (L8a_Concatenate)

* Optical calibration: Open the Calibration -> Optical Calibration tool. Save output as float, calibration level is Image to Top of Atmosphere, fill out the rest of the parameters using the scrapeMetaData sheet (may need to alter the formulae in the sheet depending on landsat year). (L8b_OpticalCalibration)

* NoData management: (L8h_RecapOpticalCalibrationMaskingPatching, 6:45 onwards)
  * Resample to match grid system. Use Geometry -> Superimpose tool. 
  * Create data mask (=1 if value in both images, 0 if no data in either image). Use Miscellaneous -> Band Math tool and expression <im1b1 && im1b2 && im1b3 && im1b4 && im1b5 && im2b1 && im2b2 && im2b3 && im2b4 && im2b5>.
  * Apply this mask to trim images. Use Conversion -> ManageNoData tool -> Apply a mask as no data.

## Step 4: Unsupervised Classification 
* Use SAGA and trimmed images from previous step.
* Tools -> Imagery -> Classification -> K-Means Clustering for Grids. (L8i_KMeans)
* Can add NDVI layer to grids, parameters: combined method, 18 clusters, 20 iterations.
* Simplified classification when you want to create a mask for clouds:
  * Mark clusters as clouds or no clouds.
  * Reclassify the cluster layer to get rid of duplicates. Clean and make two excel sheets, assign 0 to clouds, then Grid -> Tools -> Reclassify Grid Values. ( L9a_ReclassifyFilter)
  * Reduce noise from classification filter by using majority filter. Grid -> Filter -> Majority/Minority Filter. (L9b_MajorityFilter)
  * [Only do this if more than 2 classes] Reclassify again to make boolean mask. Grid -> Tools -> Reclassify Grid Values tool to make everything greater than or equal to 1 into 1 (Thus clouds become 0). (L9c_MaskInRasterAndVector)
  * Convert SAGA grid to polygon to use in QGIS. Shapes -> Grid Tools -> Vectorizing Grid Classes. 

## Step 5: Create Training Areas for Supervised Classification
* Can do this in QGIS.
* Load mask into Q and export biggest polygon as kml to view in Google Earth.
* Make polygons for training classes. ( L9d_DigitizeTrainingAreas)
* Load training areas and make more polygons in QGIS. (L9e_DigitizeInQGIS)
* Toggle edit the layer and add polygons for important or left out classes.
* Can load this training areas layer in SAGA and refine or add polygons based on kmeans classes. ( L9f_SupervisedClassification)

## Step 6: Supervised Classification
* Use SAGA for this step.
* Load the latest image in SAGA and use the training layer as a parameter in Imagery -> Classification -> Supervised Classification for Grids. (L9f_SupervisedClassification, 2:45 onwards) 
* Combine any classes if needed with the excel sheet based reclassification method outlined above. (L9i_ReclassifyMajorityFilter)
* Run majority filter to remove noise. (L9i_ReclassifyMajorityFilter, 6:45 onwards)
* Save all files.

## Step 7: Assess Accuracy of Land Cover Classification
* Can do this through cross-tabulation by selecting random points and checking if they match with the actual imagery.
* In SAGA, use Grid -> Tools -> Resampling to simplify the grid so you can create polygons for ground truthing. (L10a_Resampling)
* To reduce processing time, run this grid through a majority filter. (L10c_RandomPoints)
* Create polygons of classes. Shapes -> Grid Tools -> Vectorizing Grid Classes. (L10b_VectorizePolygons)
* Open this shape file in QGIS, specify coordinate system, then use Vector -> Research Tools -> Random points inside polygons (fixed). (L10c_RandomPoints, 1:20 onwards)
* Convert the points to kml layer and open in Google Earth for ground truthing. (L10d_GroundTruth)
* Classify the points using satellite data, then load into QGIS and open the google satellite image to fill in any missing values.
* Convert this shape file into a grid in order to compare. For this, open shape file in SAGA, convert the Name column to integer, then Grid -> Gridding -> Shapes to Grid. (L10e_ShapeToGrid)
* Crosstabulate with Imagery -> Classification -> Confusion Matrix (two grids). (L10f_Crosstabulate)
* Look at the confusion table: y axis is map classification and x axis is ground truth, AccUser is producer's reliability.
* Summary table provides the kappa statistic and overall accuracy.
* Use these tables to refine classification.

## Step 8: Detect Land Change
* Continue in SAGA and use the same confusion table tool as before: Imagery -> Classification -> Confusion Matrix (two grids). Classification 1 should be the older image and classification 2 should be the newer image, output as area (most probably output is in sq metres). (L10g_ChangeDetection)
* Load combination layer and look at legend to qualitatively identify at the change in land use.
* In the confusion table, rows represent the earlier year and columns represent the newer year.
* Suggested: Save change table and reclassify using excel method to reduce number of fields.

## Step 9: Summarize Change by Polygons
* Export polygon from QGIS as a shapefile in the projection system of the raster. Load it in SAGA so that it overlaps with the t1 to t2 combination layer.
* Convert this shapefile to a grid. Grid -> Gridding -> Shapes to grid tool.
* Now cross-tabulate roaster districts with land cover change.
* L10i mentions how to visualize land cover change, i.e. reclassifying the raster. Reclassify the raster such that you have a layer containing only the land cover change categories of interest. For example: deforestation, increased agriculture, etc.





## Notes: 
* Can look at GIMMS to check land cover change. (L8j_FinishedKmeans)
* To get a nice visualization in QGIS, set Red band to 4, Green band to 3, Blue band to 2, and select contrast enhancement -> Stretch to MinMax -> Mean +- sd. (L9e_DigitizeInQGIS)
* Additional topics mentioned in L9g_ClassificationResults: Looking at spectral signatures of classes, texture analysis.
* When exporting saga tif after supervised classification, the coordinate system is often undefined. Be careful to define the coordinate system when loading into a GIS like Q.

