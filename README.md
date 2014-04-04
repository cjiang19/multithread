multithread
===========
#Author: Chen Jiang
#Date: 04/04/2014
#Outline: The main function of this python script is to generate routes between Building 379 to the others in UMD campus.

#Define a function call Routes379
def Routes379(BldgList):

#import arcpy, os, traceback module
#check out extension and set up workspace
#create a Geo-database in my local drive
  import arcpy     
  import os     
  import traceback     
  arcpy.CheckOutExtension("Network")     
  arcpy.env.workspace = r"C:\GIS_DATA\USERS\JIANG_C\PedestrianNavigation.gdb"
  arcpy.CreateFileGDB_management("C:/GIS_DATA/USERS/JIANG_C", "379.gdb")     
  env.overwriteOutput = True
  
#Set up variable x to count
#Set a for loop to create combinations between Building 379 and Building i 
#Since we use "ArcGIS" software to calculate routes, we use built-in function in "arcpy" module to process data.
#For example "arcpy.MakeFeatureLayer" is to select data and make that as a temporary layer, "arcpy.Delete_management" is #to delete data
#Work flow of route calculation is: 
# 1. Create a "Closest Facility Layer", which is a developed "Network Dataset"
# 2. Select Building i as a "incident layer"
# 3. Select Building 397 as a "facilities layer"
# 4. Use "arcpy.solve" function to calculate possible routes between Building i and Building 397
# 5. Save Routes to the Geo-database in my local drive
# 6. select the shortest Route and delete the rest of them
# 7. "Use try: except:" to handle errors in major data processing steps
  x = 0     
  for i in BldgList:
    if i<>379:
      arcpy.MakeClosestFacilityLayer_na("C:/GIS_DATA/USERS/JIANG_C/PedestrianNavigation.gdb/Pedestrian_Navigation/Pedestrian_Navigation_ND", "Closest_Facilities"+str(x), "Length", "TRAVEL_TO", "26400")
      arcpy.MakeFeatureLayer_management("Entrance", "Entrance"+str(x)+".lyr","Bldg_No = '" + i + "'")
      arcpy.AddLocations_na("Closest_Facilities"+str(x), "Incidents", "Entrance"+str(x)+".lyr", "Name Bldg_No #")             arcpy.MakeFeatureLayer_management("Entrance", "Entrance"+str(x)+"1.lyr","Bldg_No = '379'")
      arcpy.AddLocations_na("Closest_Facilities"+str(x), "Facilities", "Entrance"+str(x)+"1.lyr", "Name Bldg_No #")
      try:
        arcpy.Solve_na("Closest_Facilities"+str(x))
        Routes = arcpy.SelectData_management("Closest_Facilities"+str(x), "Routes")
        arcpy.FeatureClassToFeatureClass_conversion(Routes, "C:/GIS_DATA/USERS/JIANG_C/379.gdb", "Routes"+str(i)+"379")
        print "All the routes from Building " + str(i) + " to Building 379 have been generated."
        del Routes
        arcpy.Delete_management("Entrance"+str(x)+".lyr")
        arcpy.Delete_management("Entrance"+str(x)+"1.lyr")
        arcpy.Delete_management("Closest_Facilities"+str(x))
        try:
          arcpy.Sort_management("C:/GIS_DATA/USERS/JIANG_C/379.gdb/Routes"+str(i)+"379","C:/GIS_DATA/USERS/JIANG_C/379.gdb/Routes"+str(i)+"379_Sort", [["Total_Length","ASCENDING"]])
          arcpy.MakeFeatureLayer_management("C:/GIS_DATA/USERS/JIANG_C/379.gdb/Routes"+str(i)+"379_Sort","Routes"+str(i)+"379_Sort.lyr","OBJECTID &lt;> 1")
          arcpy.DeleteFeatures_management("Routes"+str(i)+"379_Sort.lyr")
          arcpy.Delete_management("Routes"+str(i)+"379_Sort.lyr")
          print "The shortest routes from Building " + str(i)+ " to Building 379 has been generated." 
        except Exception as e:
          print "Failed in sorting route for building " + str(i) + " and Building 379"
          print e.message
          del Routes
          arcpy.Delete_management("Routes"+str(i)+"379_Sort.lyr")
          pass             
      except Exception as e:
        print "Failed in calcuting routes between Building "+str(i)+" and Building 379. Please check network dataset sources."                 
        print e.message
        arcpy.Delete_management("Entrance"+str(x)+".lyr")
        arcpy.Delete_management("Entrance"+str(x)+"1.lyr")
        arcpy.Delete_management("Closest_Facilities"+str(x))
        pass
      x = x+1
      print str(x)
  return

# Make a list to store all buildings
import arcpy
BldgList = []
cursor = arcpy.SearchCursor("C:/GIS_DATA/USERS/JIANG_C/PedestrianNavigation.gdb/Entrance_Summary")
for row in cursor:
  a = row.getValue("Bldg_No")
  BldgList.append(str(a))
del row,cursor

# Use multiple cores to execute Routes379()
from multiprocessing.dummy import Pool as ThreadPool
pool = ThreadPool(8) 
results = pool.map(Routes379(BldgList),(BldgList))
pool.close()
pool.join() 
