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

  ***To Do:***


