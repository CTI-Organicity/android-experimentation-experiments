# Smartphone Experimentation Experiment Plugins

In order to create a mobile Experiment for Organicity experimenters need to implement a Context Plugins that will be executed in the experimenter's smartphones, process the collected data periodically and generate experiment data that are forwarded to the Organicity Platform.

Each experiment plugin generates a list of measurements and uses a predefined set of Sensor Plugins in order to receive measurements.

Experiments are built using the Ambient Dynamix framework and the Open Context Plugin Sdk as the Sensor Plugins.

## Creating a new Experiment Plugin

The simplest way is to copy the experiment plugin template available and simply change the name, context type and the 'Runtime' class of the plugin. 
For example in the 'ExperimentPluginTemperatureGpsScan' you need to modify the 'AndroidManifest.xml', 'MANIFEST.MF' and source files under the 'org.ambientdynamix.contextplugins.ExperimentPlugin' package.

The 'PluginFactory' class is used to instantiate the plugin during runtime and actually launches the specified 'ExperimentPluginRuntime' class provided by calling its 'init' method.

The 'start', 'stop' methods are used internally by the osgi manager to enable or disable the plugin.

The 'handleContextRequest' and 'handleConfiguredContextRequest' methods are used by the osgi framework when an updated sensor value is needed. Typically this is done every 30 - 60 seconds, but times may vary based on the experiment. 

## Accessing Plugin Sensor Data

For security reasons access to the Plugin instances is restricted. Instead plugin measurements are forwarded to the Experiment Plugins when the 'handleConfiguredContextRequest' and 'handleContextRequest' are called. To access the plugin data you need to access the data from the 'Bundle config' parameter passed to the plugin instance.

      String jsonReadingTemperatureScan= b.getString("org.ambientdynamix.contextplugins.TemperaturePlugin");
      Reading temperatureReading= Reading.fromJson(jsonReadingTemperatureScan);


## Merging Sensor Measurements

To merge measurements from Sensor Plugins we use the'Reading' class. The class contains a 'context' that device the source of the measuremnt, a 'timestamp' that refers to the time instant of the measurement and a 'value' which is the string representation of a map of all the received sensor readings. A measurement can contain multiple sensor measurements like temperature, humidity and atmospheric pressure. All measurements are passed to a PluginInfo that is parsed from the osgi manager.

    String jsonReading= b.getString("org.ambientdynamix.contextplugins.GpsPlugin");
    String jsonReadingTemperatureScan= b.getString("org.ambientdynamix.contextplugins.TemperaturePlugin");
    this.gpsReading= Reading.fromJson(jsonReading);
    this.temperatureReading= Reading.fromJson(jsonReadingTemperatureScan);
    List<Reading> readings=new ArrayList<Reading>();
    PluginInfo info = new PluginInfo();
    info.setState("ACTIVE");
    
    if (this.gpsReading!=null){
        Log.w("Experiment Message:", gpsReading.toJson());						
        readings.add(new Reading(Reading.Datatype.String, this.gpsReading.toJson(),PluginInfo.CONTEXT_TYPE));
    }else{
        readings.add(new Reading(Reading.Datatype.String, "",PluginInfo.CONTEXT_TYPE));						
    }
    if (this.temperatureReading!=null){
      	readings.add(new Reading(Reading.Datatype.String, temperatureReading.toJson(),PluginInfo.CONTEXT_TYPE));
  	}else{
        readings.add(new Reading(Reading.Datatype.String, "",PluginInfo.CONTEXT_TYPE));						
    }
    info.setPayload(readings);
    sendContextEvent(requestId, new SecuredContextInfo(info,	PrivacyRiskLevel.LOW), 60000);
