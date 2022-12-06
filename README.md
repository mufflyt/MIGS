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

**clip_to_water.R** - This file downloads all the water in each state and then makes one US layer.  The file is outputed as an sf object called 'nhd-water.shp'.  These files are real THICC so I downloaded them to the external hard drive by changing the download settings in Chrome.  

Sweet code block from the www.hrecht.com code to unzip a ton of files at once.  hrecht is a baller and a shot caller:
```r
for (i in stroke_states$state_name) {
  #i = "Alabama" #for testing
	formatted <- str_replace_all(i, " ", "_")
	print(formatted)
	save_path <- paste0(nhd_directory, "NHD_H_", formatted, "_State_Shape.zip")
	dir.create(paste0(nhd_directory, formatted, "/"))
	unzip(save_path, exdir = paste0(nhd_directory, formatted, "/"))
}
```

***To Do:***
Figure out how to get miles driven to nearest MIGS from county centroid.  
