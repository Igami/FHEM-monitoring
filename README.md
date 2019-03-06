<span id="monitoring"></span>
# monitoring
Each monitoring has a warning and an error list, which are stored as readings.  
When a defined add-event occurs, the device is set to the warning list after a predefined time.  
After a further predefined time, the device is deleted from the warning list and set to the error list.  
If a defined remove-event occurs, the device is deleted from both lists and still running timers are canceled.  
This makes it easy to create group messages and send them formatted by two attributes.  

The following applications are possible and are described below:
- opened windows
- battery warnings
- activity monitor
- regular maintenance (for example changing the table water filter or cleaning rooms)
* operating hours dependent maintenance (for example clean the Beamer filter)

The monitor does not send a message by itself, a notify or DOIF is necessary, which responds to the event "<monitoring-name> error add: <name>" and then sends the return value of "get <monitoring-name> default".

<span id="monitoringdefine"></span>
## Define
`define <name> monitoring <add-event> [<remove-event>]`  
  The syntax for <add-event> and <remove-event> is the same as the pattern for notify (device-name or device-name:event).  
  If only an <add-event> is defined, the device is deleted from both lists as it occurs and the timers for warning and error are started.

<span id="monitoringset"></span>
## Set
- `active`  
  Two things will happen:
  1. Restores pending timers, or sets the devices immediately to the corresponding list if the time is in the past.
  2. Executes the commands specified under the "setActiveFunc" attribute.

- `clear (warning|error|all)`  
  Removes all devices from the specified list and aborts timers for this list. With "all", all devices are removed from both lists and all running timers are aborted.

- `errorAdd <name>`  
  Add <name> to the error list.

- `errorRemove <name>`  
  Removes <name> from the error list.

- `inactive`  
  Inactivates the current device. Note the slight difference to the disable attribute: using set inactive the state is automatically saved to the statefile on shutdown, there is no explicit save necesary.  

- `warningAdd <name>`  
  Add <name> to the warning list.

- `warningRemove <name>`  
  Removes <name> from the warning list.

<span id="monitoringget"></span>
## Get
- `all`  
  Returns the error and warning list, separated by a blank line.  
  The formatting can be set with the attributes "errorReturn" and "warningReturn".

- `default`  
  The "default" value can be set in the attribute "getDefault" and is intended to leave the configuration for the return value in the monitoring device. If nothing is specified "all" is used.

- `error`  
  Returns the error list.  
  The formatting can be set with the attribute "errorReturn".

- `warning`
  Returns the warning list.  
  The formatting can be set with the attribute "warningReturn".

<span id="monitoringreadings"></span>
## Readings
- `allCount`  
  Displays the amount of devices on the warning and error list..

- `error`  
  Comma-separated list of devices.

- `errorAdd_<name>`
  Displays the time when the device will be set to the error list.

- `errorCount`
  Displays the amount of devices on the error list.

- `state`
  Displays the status (active, inactive, or disabled). In "active" it displays which device added to which list or was removed from which list.

- `warning`
  Comma-separated list of devices.

- `warningAdd_<name>`
  Displays the time when the device will be set to the warning list.

- `warningCount`
  Displays the amount of devices on the warning list.

<span id="monitoringattr"></span>
## Attribute
- [`addStateEvent`](#addStateEvent)

- `blacklist`  
  Space-separated list of devspecs which will be ignored.  
  If the attribute is set all devices which are specified by the devspecs are removed from both lists.

- `disable (1|0)`  
  1: Disables the monitoring.  
  0: see "set active"

- [`disabledForIntervals HH:MM-HH:MM HH:MM-HH-MM ...`](#disabledForIntervals)

- `errorFuncAdd {<perl code>}`  
  The following variables are available in this function:
    - $name  
      Name of the event triggering device
    - $event  
      Includes the complete event, e.g. measured-temp: 21.7 (Celsius)
    - $addMatch  
      Has the value 1 if the add-event is true
    - $removeMatch  
      Has the value 1 if the remove-event is true
    - $SELF  
      Name of the monitoring
      
  If the function returns a 1, the device is set to the error list after the wait time.  
  If the attribute is not set, it will be checked for $addMatch.
  
- `errorFuncRemove {<perl code>}`  
  This function provides the same variables as for "errorFuncAdd".  
  If the function returns a 1, the device is removed from the error list and still running timers are canceled.  
  If the attribute is not set, it will be checked for $removeMatch if there is a <remove-event> in the DEF, otherwise it will be checked for errorFuncAdd.

- `errorWait <perl code>`  
  Wait until the device is set to the error list.

- `errorReturn {<perl code>}`  
  The following variables are available in this attribute:
  - @errors  
    Array with all devices on the error list.
  - @warnings  
    Array with all devices on the warning list.
  - $SELF  
    Name of the monitoring

  With this attribute the output created with "get <name> error" can be formatted.

- `getDefault (all|error|warning)`  
  This attribute can be used to specify which list(s) are / are returned by "get <name> default". If the attribute is not set, "all" will be used.

- `setActiveFunc <Anweisung>`  
  The statement is one of the FHEM command types and is executed when you define the monitoring or "set active".  
  For a battery message "trigger battery=low battery: low" can be useful.

- `warningFuncAdd {<perl code>}`  
  Like errorFuncAdd, just for the warning list.

- `warningFuncRemove {<perl code>}`  
  Like errorFuncRemove, just for the warning list.

- `warningWait <perl code>`  
  Like errorWait, just for the warning list.

- `warningReturn {<perl code>}`  
  Like errorReturn, just for the warning list.

- `whitelist {<perl code>}`  
  Space-separated list of devspecs which are allowed.  
  If the attribute is set all devices which are not specified by the devspecs are removed from both lists.

- [`readingFnAttributes`](#readingFnAttributes)

<span id="monitoringexamples"></span>
## Examples
The following sample codes can be imported via ["Raw definition"](https://wiki.fhem.de/wiki/Import_von_Code_Snippets).

- Global, flexible opened windows/doors message (similar to those described in the [forum](https://forum.fhem.de/index.php/topic,36504))
  >defmod Fenster_monitoring monitoring .*:(open|tilted) .*:closed  
  >attr Fenster_monitoring errorReturn {return unless(@errors);;\  
  > $_ = AttrVal($_, "alias", $_) foreach(@errors);;\  
  > return("Das Fenster \"$errors[0]\" ist schon länger geöffnet.") if(int(@errors) == 1);;\  
  > @errors = sort {lc($a) cmp lc($b)} @errors;;\  
  > return(join("\n - ", "Die folgenden ".@errors." Fenster sind schon länger geöffnet:", @errors))\  
  >}  
  >attr Fenster_monitoring errorWait {AttrVal($name, "winOpenTimer", 60*10)}  
  >attr Fenster_monitoring warningReturn {return unless(@warnings);;\  
  > $_ = AttrVal($_, "alias", $_) foreach(@warnings);;\  
  > return("Das Fenster \"$warnings[0]\" ist seit kurzem geöffnet.") if(int(@warnings) == 1);;\  
  > @warnings = sort {lc($a) cmp lc($b)} @warnings;;\  
  > return(join("\n - ", "Die folgenden ".@warnings." Fenster sind seit kurzem geöffnet:", @warnings))\  
  >}`

  As soon as a device triggers an "open" or "tilded" event, the device is set to the warning list and a timer is started after which the device is moved from the warning to the error list. The waiting time can be set for each device via userattr "winOpenTimer". The default value is 10 minutes.  
  As soon as a device triggers a "closed" event, the device is deleted from both lists and still running timers are stopped.

- Battery monitoring
  >defmod Batterie_monitoring monitoring .*:battery:.low .*:battery:.ok  
  >attr Batterie_monitoring errorReturn {return unless(@errors);;\  
  > $_ = AttrVal($_, "alias", $_) foreach(@errors);;\  
  > return("Bei dem Gerät \"$errors[0]\" muss die Batterie gewechselt werden.") if(int(@errors) == 1);;\  
  > @errors = sort {lc($a) cmp lc($b)} @errors;;\  
  > return(join("\n - ", "Die folgenden ".@errors." Geräten muss die Batterie gewechselt werden:", @errors))\  
  >}  
  >attr Batterie_monitoring errorWait 60*60*24*14  
  >attr Batterie_monitoring warningReturn {return unless(@warnings);;\  
  > $_ = AttrVal($_, "alias", $_) foreach(@warnings);;\  
  > return("Bei dem Gerät \"$warnings[0]\" muss die Batterie demnächst gewechselt werden.") if(int(@warnings) == 1);;\  
  > @warnings = sort {lc($a) cmp lc($b)} @warnings;;\  
  > return(join("\n - ", "Die folgenden ".@warnings." Geräten muss die Batterie demnächst gewechselt werden:", @warnings))\  
  >}

  As soon as a device triggers a "battery: low" event, the device is set to the warning list and a timer is started after which the device is moved from the warning to the error list. The waiting time is set to 14 days.  
  As soon as a device triggers a "battery: ok" event, the device is deleted from both lists and still running timers are stopped.

- Activity Monitor
  >defmod Activity_monitoring monitoring .*:.*  
  >attr Activity_monitoring errorReturn {return unless(@errors);;\  
  > $_ = AttrVal($_, "alias", $_) foreach(@errors);;\  
  > return("Das Gerät \"$errors[0]\" hat sich seit mehr als 24 Stunden nicht mehr gemeldet.") if(int(@errors) == 1);;\  
  > @errors = sort {lc($a) cmp lc($b)} @errors;;\  
  > return(join("\n - ", "Die folgenden ".@errors." Geräten haben sich seit mehr als 24 Stunden nicht mehr gemeldet:", @errors))\  
  >}  
  >attr Activity_monitoring errorWait 60*60*24  
  >attr Activity_monitoring warningReturn {return unless(@warnings);;\  
  > $_ = AttrVal($_, "alias", $_) foreach(@warnings);;\  
  > return("Das Gerät \"$warnings[0]\" hat sich seit mehr als 12 Stunden nicht mehr gemeldet.") if(int(@warnings) == 1);;\  
  > @warnings = sort {lc($a) cmp lc($b)} @warnings;;\  
  > return(join("\n - ", "Die folgenden ".@warnings." Geräten haben sich seit mehr als 12 Stunden nicht mehr gemeldet:", @warnings))\  
  >}  
  >attr Activity_monitoring warningWait 60*60*12  

  Devices are not monitored until they have triggered at least one event. If the device does not trigger another event in 12 hours, it will be set to the warning list. If the device does not trigger another event within 24 hours, it will be moved from the warning list to the error list.

  Note: It is recommended to use the whitelist attribute.

- Regular maintenance (for example changing the table water filter)
  >defmod Wasserfilter_monitoring monitoring Wasserfilter_DashButton:.*:.short  
  >attr Wasserfilter_monitoring errorReturn {return unless(@errors);;\  
  > return "Der Wasserfilter muss gewechselt werden.";;\  
  >}  
  >attr Wasserfilter_monitoring errorWait 60*60*24*30  
  >attr Wasserfilter_monitoring warningReturn {return unless(@warnings);;\  
  > return "Der Wasserfilter muss demnächst gewechselt werden.";;\  
  >}  
  >attr Wasserfilter_monitoring warningWait 60*60*24*25  

  A DashButton is used to tell FHEM that the water filter has been changed.
  After 30 days, the DashButton is set to the error list.

- Regular maintenance (for example cleaning rooms)
  >defmod putzen_DashButton dash_dhcp  
  >attr putzen_DashButton allowed AC:63:BE:2E:19:AF,AC:63:BE:49:23:48,AC:63:BE:49:5E:FD,50:F5:DA:93:2B:EE,AC:63:BE:B2:07:78  
  >attr putzen_DashButton devAlias ac-63-be-2e-19-af:Badezimmer\  
  >ac-63-be-49-23-48:Küche\  
  >ac-63-be-49-5e-fd:Schlafzimmer\  
  >50-f5-da-93-2b-ee:Arbeitszimmer\  
  >ac-63-be-b2-07-78:Wohnzimmer  
  >attr putzen_DashButton event-min-interval .*:5  
  >attr putzen_DashButton port 6767  
  >attr putzen_DashButton userReadings state {return (split(":", @{$hash->{CHANGED}}[0]))[0];;}  
  >attr putzen_DashButton widgetOverride allowed:textField-long devAlias:textField-long  
  >  
  >defmod putzen_monitoring monitoring putzen_DashButton:.*:.short  
  >attr putzen_monitoring errorFuncAdd {$event =~ m/^(.+):/;;\  
  > $name = $1;;\  
  > return 1;;\  
  >}  
  >attr putzen_monitoring errorReturn {return unless(@errors);;\  
  > return("Der Raum \"$errors[0]\" muss wieder geputzt werden.") if(int(@errors) == 1);;\  
  > return(join("\n - ", "Die folgenden Räume müssen wieder geputzt werden:", @errors))\  
  >}  
  >attr putzen_monitoring errorWait 60*60*24*7  

  Several DashButton are used to inform FHEM that the rooms have been cleaned.
  After 7 days, the room is set to the error list.
  However, the room name is not the device name but the readings name and is changed in the errorFuncAdd attribute.

- Operating hours dependent maintenance (for example, clean the Beamer filter)
  >defmod BeamerFilter_monitoring monitoring Beamer_HourCounter:pulseTimeOverall BeamerFilter_DashButton:.*:.short  
  >attr BeamerFilter_monitoring userattr errorInterval  
  >attr BeamerFilter_monitoring errorFuncAdd {return 1\  
  >  if(ReadingsVal($name, "pulseTimeOverall", 0) >= \  
  >    ReadingsVal($name, "pulseTimeService", 0)\  
  >    + (AttrVal($SELF, "errorInterval", 0))\  
  >    && $addMatch\  
  >  );;\  
  >  return;;\  
  >}  
  >attr BeamerFilter_monitoring errorFuncRemove {return unless($removeMatch);;\  
  > $name = "Beamer_HourCounter";;\  
  > fhem(\  
  >    "setreading $name pulseTimeService "\  
  >   .ReadingsVal($name, "pulseTimeOverall", 0)\  
  > );;\  
  > return 1;;\  
  >}  
  >attr BeamerFilter_monitoring errorInterval 60*60*200  
  >attr BeamerFilter_monitoring errorReturn {return unless(@errors);;\  
  > return "Der Filter vom Beamer muss gereinigt werden.";;\  
  >}  
  >attr BeamerFilter_monitoring warningFuncAdd {return}  
  >attr BeamerFilter_monitoring warningFuncRemove {return}  

  An HourCounter is used to record the operating hours of a beamer and a DashButton to tell FHEM that the filter has been cleaned.  
  If the filter has not been cleaned for more than 200 hours, the device is set to the error list.  
  If cleaning is acknowledged with the DashButton, the device is removed from the error list and the current operating hours are stored in the HourCounter device.
