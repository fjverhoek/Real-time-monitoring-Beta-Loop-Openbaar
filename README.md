# Beta-Loop-Pipeline-Health
## General
- In order for the FEWS-Wanda integration to work, you must have Wanda 4.7 installed.
- All commands are in _italics_. 
- If you already have certain software installed you can skip that step, but be aware that version differences can cause issues.
- Make sure to save the changes you make to any files with Ctrl+S.
- If you change something in the FEWS configuration while the FEWS interface is still open, you can update the configuration without restarting. You can do so by clicking somewhere in the log window within the FEWS interface and pressing F5. 

## 1 Install Git
1. Download it from [Git](https://www.git-scm.com).
2. Run the installer (the default settings are usually fine).
3. Open a command prompt (by typing cmd in the search bar) and type<br>
_git --version_ <br>
If it returns a number, you're ready.

## 2 Clone the repository
1. Open a command prompt and navigate to the desired folder to place your FEWS installation. 
You can do so by using the command:<br>
_cd your/desired/path_ <br>
Make sure that it's a local folder and not on the cloud or a networkdisk.

3. Clone the repository into your desired folder with the command:<br>
_git clone https://github.com/FlorisJanVerhoek/Beta-Loop-Pipeline-Health_ <br>
Close cmd<br>
If you are encountering an error while cloning, see

[Troubleshooting](#troubleshooting)

## 3 Copy the files from the network disk
1. Navigate to: P:\11211477-pipeline-healthmonitoring<br>
Copy the folders FEWS and Software into the Beta-Loop-Pipeline-Health folder that you've just cloned from GitHub. If it asks if you want to overwrite existing files press yes to all.

![drag_files](https://i.imgur.com/2bh8Rsn.gif)

## 4 Setup FEWS
1. Navigate to the folder Beta-Loop-Pipeline-Health\FEWS\bin\windows and run createShortcuts.exe
2. Choose "Path to Stand Alone or Operator Client *clientConfig.xml" and browse to the location of clientConfig.xml <br>
You can find it under Beta-Loop-Pipeline-Health\FEWS\FEWS
3. Choose to create shortcuts at Region Home and optionally also at Desktop.
4. Press "Create Shortcuts".

![createShortcuts](https://i.imgur.com/G64o024.png)

## 5 Setup of the Web Services
1. Open FEWS. You can find it under Beta-Loop-Pipeline-Health\FEWS\FEWS
2. In the top left corner under Help, open the About page. Check which version of Java is being used. If this is 17.0.9, you can use the installer in the Software folder. If not, get the correct version of Java from [Java Archive Downloads](https://www.oracle.com/java/technologies/javase/jdk17-archive-downloads.html) or [Java Downloads](https://www.oracle.com/java/technologies/downloads/).

![About_FEWS](https://i.imgur.com/2PzPFxv.png)

3. Install Java (if 17.0.9,)
4. Install Apache Tomcat 9, you can find it in the Software folder (apache-tomcat-9.0.10).
5. Leave all settings in the installer on default, but make sure to select the correct path to the installed Java (for example: C:\Program Files\Java\jdk-17).
6. Restart the FEWS interface.<br>
In the log window at the bottom of the FEWS interface, check which port the webservices are being hosted on. If this is NOT 8080, you will have to adjust this value later down the line.
7. Go to the local web address in your browser. In this example: http://localhost:8080/FewsWebServices/
8. Under PI REST Web Service, press Test page.
9. Select PI_JSON as the documentFormat, and press the link shown in the middle of the screen. It should show something similar to the image below. If you got the output, then you have succeeded in setting up the local webservice.

![Webservice_response](https://i.imgur.com/Z0ffqCc.png)

## 6 Installing the other software
1. Install Python 3.11.9, you can find it in the Software folder (python-3.11.9-amd64). 
2. Install Visual Studio Code 1.106.0, you can find it in the Software folder (VSCodeUserSetup-x64-1.106.0).
3. Install the following extensions for VS Code:
  - Pylance
  - Python
  - Rainbow CSV
  - XML Tools<br>
<br>
Usually VS Code will suggest these by itself when opening the relevant files. If not, you can search for them under the Extensions tab (Ctrl+Shift+X).
4. Setup the Python virtual environment (venv) by opening create-env.bat in the Scripts folder.<br>
If Python modules are still missing you may have to manually install them using the command prompt. The command for this is:<br>
_py -m pip install_<br>

## 7 Configuring the master script
1. Navigate to the Scripts folder and open master_script.py in VS Code. 
2. Here you can configure some settings for your project. For now, make sure that the value for PORT matches the one from step 5.4, adjust the location of the measurement directory and change SETUP_ID to an abbreviation of your facilities name (BL = Beta Loop).

![Script_configuration](https://i.imgur.com/s7hGeVs.png)

## 8 Configuring the sensors
1. Set up your measurement system and allocate a channel to each sensor in Delft Measure <br>
Each parameter has been assigned a unit. This can be changed in the FEWS config, but it is much easier to make Delft-Measure use the correct units by applying the necessary multiplications there. For example, if a pressure sensor measures in bar(g), increase factor C2 in Delft Measure by 1.<br>
<br>
Flow rate (Q) is in [m3/h].<br>
Head (H) is in [m].<br>
Pressure (p) is in [bar(a)].<br>
Differential pressure (p_dp) is in [bar(a)].<br>
Pump motor frequency (f) is in [Hz].<br>
Relative deviation (percent) is in [%].<br>
Absolute deviation (abs_dev) has no unit [-].<br>

2. Open VS Code and open the explorer (Ctrl+Shift+E). Click on Open Folder and navigate to <br>
Beta-Loop-Pipeline-Health\FEWS\FEWS and choose the folder "Config".

![open_folder](https://i.imgur.com/oXDTxV6.gif)

4. Open locations.cs under MaplayerFiles.
5. Adjust the ID, PARENT_ID, NAME. Maintain the format shown in the image below and remember to use the abbreviation you chose in step 7.2
6. Look at the top row of the file in VS Code. Each row has a corresponding color. For each parameter that is true for a given sensor, write True in the corresponding row. <br>
For example, in the image below sensor p0 measures pressure. So Col 9: p should be True. <br>
<br>
It can be helpful to use the .set file generated by Delft-Measure when you start a measurement as reference. <br>
<br>
For X and Y fill in 86045, 444592 and leave Z empty. <br>
<br>
Relative and absolute deviation are used as performance indicators. They compare the measured values of your facility with the simulated values of the digital twin. This means that you should only make this True for sensors that you are both measuring and simulating.

![locations_csv](https://i.imgur.com/gOliUYz.png)

## 9 Adding parameters
If you need additional parameters or want to change the units, we need to configure some files in the Config folder.
1. Start by adding an abbreviation or symbol for the parameter to the header of locations.csv. You must also add a comma or "True" to each row, otherwise the FEWS will see the schema as invalid.

You have to change the following files:

![parameters_files](https://i.imgur.com/MSmzV9d.png)

**Parameters.xml**
2. Add a parametergroup. You can leave <parameterType> as is.

![parameters_xml](https://i.imgur.com/8LLJjb0.png)

**LocationSets.xml**
3. Add an attribute…

![locationSet_xml_attribute](https://i.imgur.com/6OmX8N0.png)

4. …and a locationSet. You can leave <locationsetId> as is.

![locationSet_xml_locationSet](https://i.imgur.com/UaYYoSM.png)

**Filters.xml**
5. For now, under <filter id="Data" name="Measurement Data"> add a timeSeriesSet. You only need to adjust <parameterId> and <locationSetId>.

![filters_xml](https://i.imgur.com/yaoRoPJ.png)

**Import.xml (under ModuleConfigFiles)**
6. Add a timeSeriesSet. You only need to adjust <parameterId> and <locationSetId>.

![Import_xml](https://i.imgur.com/FAKKHAh.png)

## 10 Configuring the ModuleDataSetFiles: the dataConfig
1. Navigate to the ModuleDataSetFiles in Windows explorer.<br>
(Beta-Loop-Pipeline-Health\FEWS\FEWS\Config\ModuleDataSetFiles)<br>
The ModuleDataSetFiles contain two folders, one for the full Wanda model and one that merely determines the theoretical flow rate of the pump.

![ModuleDataSetFiles](https://i.imgur.com/kEvN3Uo.png)

2. These folders contain a folder with the code, and one that contains the Wanda model. FEWS only accepts these folders as .zip files. The .zip files you have copied from the network disk may be outdated, so we need to zip Wanda_Flow and Wanda_Full again.

![zip_files](https://i.imgur.com/ZsNSRdy.gif)

## 11 Configuring Wanda_Flow
Wanda_Flow uses the pump curve to determine the theoretical flow rate of the pump based the on the amount of head. Required inputs are the pressure before, and the pressure after the pump (in the example case, p0 and p1).

![Wanda_Flow](https://i.imgur.com/chCEPJv.png)

1. Open dataConfig.xml in VS Code.

![dataConfig_location](https://i.imgur.com/au7pGLX.png)

In the dataConfig you can configure which parameters are imported and exported into a Wanda model. There are 3 types of imports/exports: <br>
<br>
Type 1: Used for exporting variables to FEWS<br>
Type 2: Used for importing an initial value to Wanda<br>
Type 3: Used for importing a timeseries to Wanda<br>
<br>
To import a timeseries into a diagram, the action table and the initial value must be imported.<br>
2. Change p0 (and p0_InitialQ) to the pressure sensor that is right before the pump in your configuration.
3. Change p1 (and p1_InitialQ) to the pressure sensor that is right after the pump in your configuration.<br>
For both of them you must change the <timeSeries id> and the <locationId>.

![Wanda_Prep](https://i.imgur.com/sT2EeTZ.png)

4. Open Wanda_Flow.wdi (Beta-Loop-Pipeline-Health\FEWS\FEWS\Config\ModuleDataSetFiles\Wanda_Flow\wanda)
5. Click on Pump-1. Here you must configure the pump so it uses the correct specifications.

![Wanda_pump_settings](https://i.imgur.com/r91iTcw.png)
<br>
A given pump has different pump curves for different Speeds (rpm) or Frequencies (Hz). <br>

![pump_curve](https://i.imgur.com/FS4EN0R.png)

6. The current configuration imports a timeseries of the motorfrequency that drives the pump. If your setup instead measures the Speed (rpm) of the pump, you must change the dataConfig.xml accordingly.

![Frequency_to_speed](https://i.imgur.com/lsp97By.gif)

7. Save the Wanda model and zip it, as shown in step 10.2

## 12 Configuring Wanda_Full
Wanda_Full is the full model of your facility, or at least the branch you intend to simulate.<br>
<br>
1. Open your Wanda model, and Save As "Wanda_Full", in the correct folder<br>
(Beta-Loop-Pipeline-Health\FEWS\FEWS\Config\ModuleDataSetFiles\Wanda_Full\wanda)

![Wanda_Save_As](https://i.imgur.com/Oq7wXYC.gif)

2. In the Wanda interface, replace your pump diagram with a BoundQ diagram. If possible, name this BoundQ "Tank1-2" to simplify things. 
3. Remove all components placed before the pump but after the reservoir. Alternatively you can make them disused.

![BoundQ](https://i.imgur.com/xqxwjBJ.png)

4. Set up measurements points.<br>
The timeseries we want to export from Wanda to FEWS need to have a defined place. This can be done by separating a pipe at the location of a sensor.<br>
<br>
For example: if you want to export the pressure at the halfway point of pipe P1, you must split it up into two segments.

![split_pipe](https://i.imgur.com/YaSleBo.png)

5. Now it is time to set up the dataConfig.xml for Wanda_Full. Open VS Code and navigate to dataConfig.xml under Wanda_Full.

![dataConfig_Full_location](https://i.imgur.com/AYV6o6T.png)

6. To import or export a timeseries, we need to define which component it has to go/come from, and which component property we wish to import/export. This works the same as in step 11.2 and 11.3.<br>
<br>
In this example, we will import the flow rate to the BoundQ diagram, and export the pressure timeseries. You can import or export any Wanda parameter you'd like however.<br>
<br>
Here is an example of configuring a pressure sensor.

![configuring_pressure_sensor](https://i.imgur.com/O7dM2mx.mp4)

## 13 Setting up the Wanda workflow
In order for the Wanda workflow to work, the modules must be configured so they handle your timeseries correctly. All the timeseries you are importing into Wanda (all the <importedVars>) must be resampled so Wanda can use them. All imported and exported timeseries must also be referred to correctly in the Wanda_Flow and Wanda_Full module.<br>
<br>
1. In VS Code, navigate to the Wanda folder, under ModuleConfigFiles. Open Wanda_Prep.xml first.

![dataConfig_Prep_location](https://i.imgur.com/c7ZGMXc.png)

To perform a transformation on a timeseries, we must define the input, the output and the transformation we wish to do.
2. For every timeseries you are importing make sure there is an input, an output and the correct transformation.

The <moduleInstanceId> is the module where the timeseries originates from. In this case, that would be the Import module, where all measurement data is stored. ModuleInstanceIds are defined under WorkflowFiles. In the example configuration however, all moduleInstanceIds are the same as the file name of the xmls.<br>
<br>
The <timeSeriesType> is a value we have assigned to a timeseries. In this example, we only use three types: "external historical", "simulated historical" and "temporary". In general, you can put down all timeseries as "simulated historical", except those originating from the Import module. "temporary" timeseries are deleted after a workflow finishes, and will not show up in the FEWS GUI. You can read more about timeSeriesTypes on the FEWS wiki. 
[FEWS Wiki - timeSeriesTypes](https://publicwiki.deltares.nl/spaces/FEWSDOC/pages/8683850/02+Data+Handling+in+Delft-FEWS#id-02DataHandlinginDelftFEWS-Typesoftimeseries)<br>
<br>
Below is an example showing how to add a pressure timeseries.

![Wanda_Prep](https://i.imgur.com/CDcGezh.gif)

3. Navigate to Wanda_Full.xml.

![Wanda_Full_location](https://i.imgur.com/LMO1mnS.png)

4. Under <exportActivities> add a <timeSeriesSet> for all the timeseries you are exporting from FEWS into Wanda. These are the timeseries you have defined as <importedVars> in the dataConfig for Wanda_Full in step 12.6.

![Wanda_Prep_xml](https://i.imgur.com/AClNWsj.png)

5. Under <importActivities> (scroll down a bit) add a <timeSeriesSet> for all the timeseries you are importing from Wanda into FEWS. These are the timeseries you have defined as <exportedVars> in the dataConfig for Wanda_Full in step 12.6.

![Wanda_Flow_xml](https://i.imgur.com/BsCvuLz.png)

6. Navigate to Wanda_Flow.xml, and repeat steps 4 and 5 for Wanda_Flow.

## 14 Setting up the Berekeningen workflow
The Berekeningen workflow is used to perform calculations on all the timeseries we have gathered. In the example case it's primary use is to calculate absolute and relative deviation of the pressure sensors.<br>
<br>
1. Navigate to Berekeningen_Prep.xml. Here we prepare the timeseries for further calculations. All nonequidistant timeseries (all timeseries from the import module) must be resampled into equidistant if we wish to compare them to our simulated data, which is equidistant.

![Berekeningen_Prep_xml](https://i.imgur.com/7A9djAq.png)

2. Here you must add an input, output and transformation. Just as in step 13.2.
3. Repeat step 2 and 3 for Berekeningen.xml, but set up the transformation as you wish. The <user>, <simple> transformation can perform basic calculus (additions, multiplication etc). If you wish to do more, see the FEWS wiki [FEWS Wiki - Transformations](https://publicwiki.deltares.nl/spaces/FEWSDOC/pages/8684059/20+Transformation+Module+-+Improved+schema)

![example_simple_transform](https://i.imgur.com/4x7ezNR.png)

## 15 Setting up the filters
In order for our timeseries to show up in FEWS, we must add them to the filter.
1. Navigate to Filters.xml.

![Filters_xml_location](https://i.imgur.com/B8zhjVZ.png)

2. Add your timeseries to the corresponding filter (in the example, these are <filter id="Data" name="Measurement Data">, <filter id="SimData" name="Simulated Data"> and <filter id="CalcData" name="Calculated Data">.
<br>
You can choose to add a <locationId> or a <locationSetId>. A <locationId> will add a single locationId, while a <locationSetId> will add all locations under that set (as defined in locationSets.xml, under RegionConfigFiles).<br>
For example, <locationSetId>locations_p</locationSetId> will add locations p0_BL, p1_BL, p2_BL, p3_BL and p4_BL.

![locationSetId](https://i.imgur.com/1B5MRGG.png)

## 16 Setting up the thresholds

## 17 Starting the real-time monitoring
1. Navigate to the Scripts folder (Beta-Loop-Pipeline-Health\Scripts) and open master_script.py with VS Code.
2. Run the script by pressing the run button in the upper right corner. If the button is not there, run from the Run tab in the upper left corner (before running, make sure that FEWS is opened and connected to the right PORT, and that Wanda is closed).

![run_button](https://i.imgur.com/Br1sSTN.png)

3. In the FEWS interface, look at the log window. If the log is showing the message in the image below, the measurement data is being imported into FEWS. For testing purposes, there is a script to generate fake data called generate_fake_data. Make sure to "Run Python File in Dedicated Terminal". This option is given in the drop down menu next to the run button.

![import_in_log](https://i.imgur.com/DTq8XF7.png)

If you have configured the Filters.xml correctly, your FEWS interface should look something like the gif below. If it does, you have set up the FEWS-Wanda integration succesfully.

![data_shown_in_FEWS](https://i.imgur.com/FYOW6ur.gif)

4. You can make the interface scroll along with the data by setting the Relative View Period.

![FEWS_interface_autoscroll](https://i.imgur.com/XKr7tIi.gif)

##  Troubleshooting
testing  
testing  
testing  
testing
