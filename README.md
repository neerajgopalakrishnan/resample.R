# resample.R
# To resample raster files in proportionate to its corresponding climate raster, employed using "Mode" method of resampling.
# Mask Non-forested areas and resample only forested areas.


# Install terra if not already installed: install.packages("terra")
library(terra)

# 1. Load your datasets
# fsi_forest: 30m categorical data 
# climate_data: 4km continuous data (Temp, Precip, etc.)
forest_30m <- rast(" Path file.tif")
climate_4km <- rast("Path file.tif")

# 2. Check CRS and Projections
cat("Forest CRS:", crs(forest_30m, describe=T)$name, "\n")
cat("Climate CRS:", crs(climate_4km, describe=T)$name, "\n")

# 3. Resample and Reproject
# We use 'project' to align forest_30m to the grid of climate_1km.
# method = "mode" ensures we pick the most frequent forest class in the 1km cell.
# This avoids creating "average" decimals like 1.5.
cat("Starting resampling (this may take a moment for 30m data)...\n")

forest_4km <- project(
  forest_30m, 
  climate_4km, 
  method = "mode"
)

# 4. Masking
# Since you already removed non-forest areas, your forest_4km will have 
# NA (NoData) in those spots. The mask() function sets climate pixels 
# to NA wherever the forest_4km is NA.
climate_masked <- mask(climate_4km, forest_4km)

# 5. Save the outputs
writeRaster(forest_4km, "D:files\\resample\\forest_1km.tif", overwrite=TRUE)   # set directory 
writeRaster(climate_masked, "D:files\\resample\\climate_mask4km.tif", overwrite=TRUE)

# 6. Visual Check
# Plotting the result to ensure the mask worked correctly
plot(climate_masked, main="Climate Data (Forest Areas Only)")
