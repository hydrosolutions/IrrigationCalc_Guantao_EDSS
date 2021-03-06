# Guantao Irrigation Calculator

## Overview

This webapplication has been designed on the basis of the R library Shiny. See here for a quick-start turorial: http://rstudio.github.io/shiny/tutorial/. The App has several features:

* Displaying and comparing meteorological data (Tab: Climate)
* Calculating Irrigation Water Demand under a certain climate (Tab: Irrigation Calculator)
* Planning Irrigation Water Demand of a region with multiple plots (Tab: Irrigation Planning Table)
* Monitoring NDVI with MODIS Data (Tab: liveNDVI)

The Irrigation Calculator is based on the [FAO Irrigation and drainage paper 56](http://www.fao.org/docrep/X0490E/X0490E00.htm) and a Single crop coefficient.

The App has no global user login. However, the Irrigation Planning Table can be saved&loaded to an username and password combination. This data is stored in a SQLite Database. Passwords are securely encrpyted here. It is recommended to set-up a server such that request to the Shiny Server are sent via an SSL connection.

The data for the Tab liveNDVI are not directly produced in the App. (This is not possible due to the fact that a webapplication only runs on request of an user and should not run processes that are highly computationally expensive. The data are thus produced by the download script of the [MODISdata-server](https://github.com/hydrosolutions/MODISdata-server). The Webapplication accesses the data via an SQLite database, that the MODISdata-server is regularly updating.

In the following, each part of the Irrigation Calculator is described in more detail and technical particularities are given.

## Shiny - Concept

Shiyn offers an easy and quick approach for creating interactive web application. A typical Shiny application consists of three parts: 

* global.R: The code here is run at the beginning of starting a Shiyn application and only once. Here global variables and data can be loaded and prepared.
* ui.R: The ui.R file contains the structure of the user interface. All static UI elements can be constructed here. However, every UI element that might change during the use of the application must be constructed in server.R. In the Guantao Irrigation Calculator, this concerns almost every UI element, because their titles are dynamically changed depending on the selected language.
* server.R: This is the core of the application. Here, all dynamic content is processed. UI elements can have values e.g. an checkbox has a value of TRUE/FALSE. They are called reactive values. This value can be used in any function in server.R. The reactivity concept of Shiny is the most important thing to understand here: As soon as a reactive value does change, all functions depending on it will be re-evaluated. Please see the extensive documentation on R Shiny for further details and explanations.

Hint: Debugging does not work as in a usual R script. Instead, the function browser() must be called on the line were the execution of the application should be paused.

## Climate Data

Climate Data are monthly averages and are loaded as simple .csv files in global.R. [Example](data/meteodata/CLIMWAT_FAO.csv). Dummy data may be modified by the user in the climate tab. Climate data is stored as a reactive variable in the app.  


## Single crop coefficient model

The model is loaded in globa.R from the folder application folder bin/. The folder contains two R files:

* ETmodel.R: Here, Penman Monteith reference ET is computed based on the climate data.
* waterbalancemodel.R: The function monthly.soil_water_balance does compute the output variables that are finally shown on the Tab Irrigation Calculator. Beside the climateb data, it takes the arguments "crop" and "soil", which contain the crop resp. soil parameters as specified in the FAO 56 paper. A selected list of parameters is contained in [data/croptypes.csv](data/croptypes.csv) and [data/soiltypes.csv](data/soiltypes.csv). In the Webapplication, the user chooses the crop- resp. soiltype from a dropdown menu. The corresponding row of the parameter table is then given to the monthly.soil_water_balance function as argument.

## Irrigation Planning Table

The content of the Irrigation Planning Table is stored as a data.frame during runtime fo the application. Additional Rows are added via the Irrigation Calculator. When a user wants to store the current table, the variables username and password are fetched via the UI, and the Data.Frame is stored in the database as BLOB where the username matches an existing user with the given password. A new user can be created as well. For this, an mail is sent to verify the users e-mail adress.

## liveNDVI

As metioned above, the liveNDVI data are not produced within the webapplication itself. The webapplication accesses a SQLite database, that is maintained by another application. The database stores the filepathes to the Geotiff files as well as to csv files containing timeseries data. The files itself are - due to the size - stored seperatly on the file system. The path to the database and the root path of the data storage location must be defined in global.R.

An example database and example data are located in [data/exampledb.sql] and [data/exampledata/]

## E-mail

An e-mail smtp server is required for this application in order for the send.mail function ofn the mailR library to work. At the moment (28.09.2018) my user account on [smtp2go.com](https://www.smtp2go.com) is used. Therefore, all emails sent from the application come from the e-mail adress ic_help@hydrosolutions.ch. This can easily be changed.

## Language Switch

Translation is based on two dictionary files. The normal file is located in [data/dictionary.csv](data/dictionary.csv) and is structure by: key\English\Chinese. Each row contains another term. Within server.R, the function tr("key") returns the string in that dictionary file for the key in the currently selected language. Remark: tr() can not be used outside of server.R. That means all UI elements requiring translation must be constructed in server.R

Another dictionary file is found here [data/dictionary_UI.csv](data/dictionary_UI.csv). This table is reserved for simple textOutput Elements with no other functionality e.g. the Tab titles. A short function does loop over the table to produce these elements automatically in order to avoid writing down the construction of each of them. (Remark: Somehow, probably due to a bug in Shiny, the amount of such constructed UI elements must be kept low. At a certain pointed other elements randomly started to dissappear when extending this list further)

## IP logger

When an user open the webapplication, its IP and other information is logged in the file [log/access.log](log/access.log). Unfortunately this does only work if the Shiny Server is accessed via HTTP. However, this is not recommed because passwords and emails adresses are sent w encryption and SSL access is therefore recommened.

## The folder www/

... is expected by a Shiny application to locally store the elements that will be rendered in the web browser and are not the output of the scripts. 
Here, it stores the titlebar images and some javascript functions, that are loaded in ui.R to enable some functionality, that is not natively supported by Shiny.


## Authors

* **henze** - @hydrosolutions.ch




