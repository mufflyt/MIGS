# MIGS

***points_to_polygons.R***

Gathered data for women of reproductive age using US Census Bureau data.  I found the age and sex variables here:  (https://api.census.gov/data/2010/dec/sf1/variables.html) and chose "P012039","P012038","P012037", "P012036", "P012035", "P012034", "P012032","P012031".  I was able to get data from 2010 but not from 2020 decennial census.  Updated all packages.  

***Variables***
Find the variables at https://www.socialexplorer.com/data/ACS2014_5yr/metadata/?ds=ACS14_5yr&var=B01001026.  We used 2019 data because 2020 was all estimated per tidycensus.  https://www.socialexplorer.com/data/ACS2019/documentation/53fe1152-1446-4d2a-9ec5-5db795d48eb6

Get 2020 data from the decennial census.  Getting this error: "Getting data from the 2020 decennial Census
Using the PL 94-171 Redistricting Data summary file
Error in UseMethod("gather") : 
  no applicable method for 'gather' applied to an object of class "character"

The PL 94-171 Redistricting Data summary file is not available for the 2020 decennial census.  

***Materials & Methods***
This is a good link to the accuracy of the ACS data:  https://www.socialexplorer.com/data/ACS2019/documentation/6f6b45bf-6046-4b82-9c17-053f8f4c8da1.  Good documentation for the ACS data: https://www.socialexplorer.com/data/ACS2019/documentation/ca1173fe-3362-4468-86f6-a2fedfe34955.  
```{r}
# Set a dataset of median home values from the 1-year ACS  #shift Hawaii and AK
county_female_pop <- get_acs(geography = "county", 
                       variables = c("B01001_033", 
                                     "B01001_034", 
                                     "B01001_035", 
                                     "B01001_036", 
                                     "B01001_037", 
                                     "B01001_038", 
                                     "B01001_039"),
                       survey = "acs1", #Screws everything up if this is included.  
                       year = 2020,
                       geometry = TRUE,
                       cache = TRUE) %>%
                        dplyr::select(GEOID, NAME, estimate, geometry)
```
***Limitations***
Limitations are limited population data 'Getting data from the 2019 1-year ACS.  The 1-year ACS provides data for geographies with populations of 65,000 and greater.'

***Isochrones Tutorial***
http://www.hrecht.com/r-drive-time-analysis-tutorial/tutorial.html
Make sure you are using the API key and not the Oath bearer ID.  
https://stackoverflow.com/questions/42424544/here-api-these-credentials-do-not-authorize-access-some-requests

***Shapefiles***
You may download TIGER/Line Shapefiles using a Web interface, directly from our FTP site, or via an FTP client. To download through the Web interface or FTP site, go to <www.census.gov/programs -surveys/geography. html>. Choose Geographies, then Mapping Files, then TIGER/Line Shapefiles.

Which files are national?  https://www.census.gov/programs-surveys/geography/technical-documentation/complete-technical-documentation/file-availability.html.  I found some national block group files at Block group data source: IPUMS NHGIS 2019 national shapefile and https://data2.nhgis.org/downloads when searching 2020.  

Where is the water?  Downloaded the National Hydrography file from USGS.  https://www.sciencebase.gov/catalog/item/5ea068ae82cefae35a12a120, http://prd-tnm.s3-website-us-west-2.amazonaws.com/?prefix=StagedProducts/Hydrography/NHD/State/Shape/

**clip_to_water.R** - Able to scale to the entire USA.  This file downloads all the water in each state and then makes one US layer.  The file is outputed as an sf object called 'nhd-water.shp'.  These files are real THICC bois so I downloaded them to the external hard drive by changing the download settings in Chrome.  Sweet code block from the www.hrecht.com code to unzip a ton of files at once.  hrecht is a baller and a shot caller:
```r
# Turn fips into names for file downloading
nhd_directory <- "/Volumes/Video Projects Muffly 1/Workforce/Hydrology/"

# Downloaded manually, then run code to unzip in nice subfolders
for (i in stroke_states$state_name) {
  #i = "Alabama" #for testing
	formatted <- str_replace_all(i, " ", "_")
	print(formatted)
	save_path <- paste0(nhd_directory, "NHD_H_", formatted, "_State_Shape.zip")
	dir.create(paste0(nhd_directory, formatted, "/"))
	unzip(save_path, exdir = paste0(nhd_directory, formatted, "/"))
}
```
I keep getting a corrupt zip file for Oregon and Texas so the zip files were downloaded again and manually unzipped instead of using code.  This is some more amazing code where all the state shapefiles are brought together.  WOW!
```r
###########################################################################
# Read in Waterbody shapefiles and join non-tiny features
###########################################################################
water <- NULL
for (i in stroke_states$state_name) {
  #i = "Washington" #for testing
	formatted <- str_replace_all(i, " ", "_")
	print(formatted)
	
	# If more than 200k features it's split into multiple files
	waterbody_files <- fs::dir_info(paste0(nhd_directory, formatted, "/Shape/"), recurse = FALSE, glob = "*.shp")%>% #Creates: /Volumes/Video Projects Muffly 1/Workforce/Hydrology/Alabama/Shape/NHDWaterbody.shp
		filter(type == "file" & str_detect(path, "NHDWaterbody")) %>%
		select(path)
	
	for (f in 1:nrow(waterbody_files)) {
	  #f=1 #for testing
		water_full <- st_read(waterbody_files[f,])
		water_min <- water_full %>% filter(areasqkm > 1)
		water <- bind_rows(water, water_min)
	}
	
}
rm(water_full, water_min)
```

**Make sure that API for the US Census Bureau is loaded up**
```r
#https://www.hrecht.com/censusapi/articles/getting-started.html
# Add key to .Renviron
Sys.setenv(CENSUS_KEY="xyz")
# Reload .Renviron
readRenviron("~/.Renviron")
# Check to see that the expected key is output in your R console
Sys.getenv("CENSUS_KEY")
```
***get_census_data.R***
Instead of the Southeast US with 10 fips codes only I want to get data for the entire USA.  I used this code:
```r
us_fips <- tigris::fips_codes %>%
  select(state_code, state_name) %>%
  dplyr::distinct(state_code, .keep_all = TRUE) %>%
  filter(state_code <56) %>% select(state_code) %>% pull()
```

**get_isochrones.R** - This file calls the API for here and finds the isochrones.  Impressive AF.  I also like how she named the files get-??? so you had an idea of what goes first.  

I kept getting this error and it was solved by SO.  
`In CPL_write_ogr(obj, dsn, layer, driver, as.character(dataset_options),  :
  GDAL Error 1: data/shp/isochrones/isochrones.shp does not appear to be a file or directory.`
https://stackoverflow.com/questions/60454067/cpl-write-ogr-error-when-writing-out-shape-file-in-r

***make_delta-appalchian-region.R***
This was a treasure trove of SHP files:  https://www2.census.gov/geo/tiger/GENZ2019/shp/ and where I found this beaut: cb_2019_us_county_5m.shp.  

***Calculate polygon demographics.R****
Runs great.  

***Calculate-polygon-overlap.R***
I could not find "block-groups-water-removed.shp" created anywhere.  I think she created ""block-groups-water-removed.shp"" in QGIS.  Downloaded QGIS and this thing is nice.  I wonder if it has programmable code because I am going to forget a step here or there.  The grouped block areas had some invalid geometry so I started with the county level data.  Will need to use this link to figure out how to make the gemoetry valid: https://www.qgistutorials.com/en/docs/3/handling_invalid_geometries.html

qgis steps: Bring in data/shp/water/water-processed/nhd-water.shp and /workforce/Code/2021-delta-appalachia-stroke-access/data/shp/block-groups.  
Open processing toolbox and chose "Check Validity".  Run "Fix Geometries" on the Error Layer.  

***To Do:***
Figure out which data we want to get for get_census_data.R.  US female population counts.  

***Urban vs. Rural***
`tigris::urban_areas()`:  Urban areas include both "urbanized areas," which are densely developed areas with a population of at least 50,000, and "urban clusters," which have a population of greater than 2,500 but less than 50,000.  This is for the methods:  https://www2.census.gov/geo/pdfs/reference/ua/2020_Urban_Areas_FAQs.pdf about the definitions of urban vs. rural.  I got a shapefile from TIGER: https://catalog.data.gov/dataset/tiger-line-shapefile-2017-2010-nation-u-s-2010-census-urban-area-national.  


