# Border forensics GIS session: service accessibility from assylum facilities in Switzerland

In this session, we will evaluate the accessibility to services from asylum facilities in Switzerland and compare it to the [service accessibility of the general population according to the Swiss Federal Statistical Office (FSO)](https://www.bfs.admin.ch/bfs/fr/home/statistiques/themes-transversaux/analyses-spatiales/services-population/accessibilite.html). To that end, we provide the following data:

- `bassins.gpkg`: a vector layer with the labour market areas (bassins d'emploi) of Switzerland [as defined by the FSO](https://www.bfs.admin.ch/bfs/en/home/statistics/catalogues-databases.assetdetail.8966775.html)
- `asylum-facilities.gpkg`: locations of asylum facilities (cantonal and federal) in Switzerland
- **amenities**: locations of amenities of three categories, each in a dedicated file: `amenities-pharmacies.gpkg`, `amenities-restaurants-cafes.gpkg` and `amenities-supermarkets.gpkg`.

The data can be downloaded in a zip file from [this link](https://drive.switch.ch/index.php/s/lDv37pouN1fIXmU).

The end goal is to create three maps that show the distance from asylum facilities in Switzerland to the closest pharmacy, restaurant/cafe and supermarket respectively. Such distances will be averaged over the labour market areas (bassins d'emploi) of Switzerland to obtain maps such as:

![Accessibility to restaurants/cafes ([link to the source](https://www.atlas.bfs.admin.ch/maps/13.442/map/mapIdOnly/25092_fr.html))](images/restaurant-accessibility-fso.png)

## Prerequisites

The analysis will be performed in QGIS 3.28.4 'Firenze'.

## Loading data in QGIS

Load the `assylum-facilities.gpkg` file to QGIS as a new layer - you can drag and drop it to the QGIS window or use the `Layer` > `Add Layer` > `Add Vector Layer...` menu. To add more context to the visualization, you can add the `OpenStreetMap` layer at the `Browser` panel, under the `XYZ Tiles` group. Similarly, load the `bassins.gpkg` file to QGIS as a new layer and place it below the "assylum-facilities" layer but over the "OpenStreetMap" layer. Finally, add the three amenity layers (`amenities-pharmacies.gpkg`, `amenities-restaurants-cafes.gpkg` and `amenities-supermarkets.gpkg`) to QGIS and place them anywhere over the "bassins" layer. You can optionally group the amenity layers under a new group called "amenities" to keep the layer panel tidy by selecting the layers and right-clicking on them and clicking `Group Layers...`. Feel free to change the symbology of the layers to make them more visible by right-clicking on the layer and selecting `Properties...` > `Symbology` > `Single symbol` and selecting a color and a style for the layer.

## Computing distances to the closest amenities

We will now compute the distance from each assylum facillity to the closest amenity of each category. To do so, we will use the `Processing Toolbox` and select the `Distance matrix` algorithm. To open the `Processing Toolbox`, click on the `Processing` menu and select `Toolbox`. In the `Processing Toolbox`, expand the `Vector analysis` group and double-click on the `Distance matrix` algorithm. The `Distance matrix` algorithm will compute the distance from each feature of the input layer to the closest feature of the reference layer. In our case, the input layer will be the `assylum-facilities` layer and the reference layer will be the an amenity layer. We can proceed with the default options except for the `Use only the nearest (k) target points` option, which we will set to `1` to compute the distance to the closest amenity. Finally, click on the `Run` button to start the computation. You can rename the output layer to `distance-pharmacies` by right-clicking on the layer and selecting `Rename...`. Repeat the same process for the other two amenity layers, renaming the output layers to `distance-restaurants-cafes` and `distance-supermarkets` respectively. Feel free to group the output layers under a new group called "distances".

We can improve the visualization of the distance layers by right-clicking on one of the layers and selecting `Properties...` > `Symbology` > `Graduated` and selecting the `Distance` field as the "Value". You can optionally change the color ramp to make the results more visible.

## Analysis at the labour market area level

### Filtering labour market areas with no assylum facilities

Before going any further, we will first filter out the labour market areas that do not contain any assylum facilities. To do so, we will use the `Extract by location` algorithm, which can be found in the `Vector selection` group of the `Processing toolbox`. The `Extract by location` algorithm will extract the features of the input layer that intersect with the features of the reference layer. We will set the `Extract features from` field to the `bassins` layer and the `By comparing to the features from` to the `assylum-facilities` layer. The remaining options can be left to the defaults.

It may be convenient to rename the resulting layer (e.g., to "bassins-with-facilities") and even deleting the original "bassins" layer.

### Aggregating distances to the labour market areas

We will now aggregate the distances to the closest amenities to the labour market areas (bassins d'emploi) of Switzerland. To do so, we will use the `Processing Toolbox` and select the `Join attributes by location (summary)` algorithm, which is under the `Vector general` group. In our case, we will set `Join to features in` to the *filtered* `bassins` layer and `By comparing to` to one of the distance layers. The rest of the options can be left to the defaults. Again, feel free to rename the resulting layer to something more meaningful (e.g., "bassins-pharmacies") and optionally group it under a new group.

The resulting layer will contain many statistics, which we can see by right-clicking to the layer and selecting `Open Attribute Table`. Since we are looking for the *average* distance to the nearest amenity for each labour market area, we only need the `Distance_mean` column, but you can keep the other columns if you want to perform further analysis.

### Plotting the results in the map composer

In order to plot the results, we can create a `Print Layout` by going to the `Project` menu and selecting `New Print Layout...`. You can optionally rename the layout to something more meaningful (e.g., "Accessibility to amenities"). To add a map to the layout, click on the `Add new map` button and place it on the page layout.

To edit the composed map, you can zoom and pan the map in the main QGIS window, as well as select and order the layers. In the context of this assignment, we need to color the labour market areas according to the average distance to the nearest amenity. This can be achieved by right-clicking on the layer and selecting `Properties...` > `Symbology` > `Graduated` and selecting the `Distance_mean` field as the `Value`. You can try different modes of classification (e.g., `Equal interval`, `Quantile`, `Natural breaks`) and color ramps to find the best visualization. For instance, you can choose the `Fixed interval` mode and set the interval size to 250 (meters) to mimic the map from the FSO.

Once you are happy with the rendered map, you can add a legend in the Layout window by clicking on the `Add Legend` button and placing it on the page layout. You can edit the legend in the `Item Properties` panel on the right side of the window. You can also add a title to the map by clicking on the `Add Label` button and placing it on the page layout and edit the text in the `Item Properties` panel on the right side of the window, or also add a scale bar by clicking on the `Add Scale Bar` button and placing it on the page layout. See the [QGIS documentation](https://docs.qgis.org/3.22/en/docs/user_manual/print_composer/index.html) for more information on how to use the Layout window.

Finally, you can export the layout to a PDF file by going to the `File` menu and selecting `Export as PDF...`. You can optionally change the resolution to 300 dpi to obtain a better quality PDF file.
