## iCloud3 Change Log

#### Version 1.0.5 - June 15, 2019

**Significant change - Track devices when the iCloud Location Service is not available**

- When iCloud3 starts, the iCloud account is accessed for device and location information. If the iCloud account can not be access (the Apple iCloud service is down, an error authorization error is returned from the iCloud service, the account can not be found, etc.), iCloud3 will now startup in an *iCloud Disabled Operating Mode*. The following occurs:

  - iCloud3 will rely on HA IOS app to provide Zone Exit, Zone Enter, Background Fetch, Significant Location Update and Manual triggers to know where the device is located.
  
  - iCloud3 will not poll the device on a regular basis since it can't accessed the iCloud Find-My-Friends location service. The decreasing interval as you approach home will be not be done. Automations and scripts based on a short distance from home will not trigger. Automations and scripts triggered on a zone change should continue to work.
  
  - The devices to be followed must be listed in the *include_devices* configuration parameter. This is described in the iCloud3 documentation [here.](https://github.com/gcobb321/icloud3#user-account-and-device-configuration-items)
  - The device is not located when HA starts. It may take a few minutes to process the next IOS app notification to locate the device.
  
- The *include_device_type & exclude_device_type* configuration parameters will not work since they are used to select devices from an iCloud account. You do not have to remove these entries from the configuration file, they will be ignored.
  
- A new configuration parameter, *icloud_disabled* has been added. This can be used to *not* use an iCloud service even if it is available. iCloud3 will then run as described above in the *icloud_disabled* operating mode. 

- iCloud3 can be restarted using the service call  *icloud_update* with the *restart* command or the service call *icloud_reset*. If you did not use the *icloud_disabled* configuration parameter, the iCloud Location Service will be rechecked and used if it is available. 

**Other Changes**

- If there was an error with the iCloud account name in the configuration parameters, no iCloud account would be found and an error message is displayed in the log file.  Displaying this error message generated another error message that is now fixed.

- There was a fix for a problem in v1.0.4 that never made it into the update. The iCloud Service Call (Set Interval, Lost Phone Alert) for a device when multiple iCloud3 device tracker platforms are being used now works as expected.

- Removed a log file warning message related to time zones.

- If no devices were set up for a valid account, no error message was displayed. However, the error *'Icloud' object has no attribute 'any_device_being_updated_flag'* was. This has been fixed.

  

#### Version 1.0.4 - June 6, 2019

- **Support for Home Assistant version 0.94.0 released June 6, 2019.**

- Fixed problem where iCloud Service Calls may not be executed for a device when multiple iCloud3 device tracker platforms were set up for different iCloud accounts.

- Fixed a problem where the last update, the next update and last located dates were displayed as a UTC time rather than the local time. This has been fixed. It appears that newer installs of Hass.io that are based on the Hass.io Operating System rather than Raspbian are using UTC for log file times rather than the local time. This was effecting the how times are used and reported.

- Fixed a problem where the Old Location count was being updated twice on the same poll.

- If a device has an old location, it was repolled every 15-seconds to try to establish a good location. This has been changed to 30-seconds. Every 10th time, the interval will be set to 5-minutes. If the the Old Location counter reaches 41or the last good location is more than 30-minutes old and the Old Location counter reaches 21, the device is put into a 'Paused' status and no more polling will be done until it is resumed using the 'icloud_update resume' service call. This might happen in large dead zone or when a device is put into 'airplane' mode with no wifi coverage.

- Began preparations for version 2 of the IOS App.

  

#### Version 1.0.3 - March 28, 2019

- Fixed a problem where Waze would sometimes not resume when leaving home if it had been previously paused when traveling towards home and 1km away from home.
- Fixed a problem where the zone would not change when using the *icloud_update* (i.e., *zone not_home* command) service call in an automation or script.
- Fixed a problem generating a 'self.any_device_being_updated_flag is undefined' error message.
- Changed the Waze out-of-range messagedisplayed in the WazeDist field to not display anything if less than 1km from home, *DistLow* if it is less than the *waze_min_distance* configuration parameter and *DistHigh* if greater than the *waze_max_distance* configuration parameter.

-------------------------------------------------------------------------------------------------------------------------------------------------------

#### Version 1.0.2 - March 24, 2019

- Waze Changes:

  - The Waze current location, distance from home, travel time information returned from the Waze route calculator is now being saved. On the next poll of any device using iCloud3 on the same account, the saved data is searched to see if there is any entry that is close to your current location instead of calling the Waze route calculator again. This speeds up the polling process and shares data between devices that are close to each other. 

    *Note:* Only devices on the same iCloud account can share Waze data. In HA, each device_tracker platform operates independently of each other even though they are both using the iCloud3 platform. For example, devices set up in one platform (e.g., account_name: gary_icloud) and devices set up on another platform (e.g., account_name: lillian_iphone) don't know about each other so they can not share Waze data

  - There are times when no distance and travel time is returned from the Waze servers and nothing is displayed in the WazeDist or TravTime fields. iCloud3 will now retry getting data from the Waze servers up to 4 times if this occurs.

  - A status message is now displayed in the WazeDist field if there is a problem (WazeOff, Paused, WazeError, etc.) rather than leaving the field blank.

  - Fixed a problem where Waze could not be toggled on or off using the icloud_command service call.

- Badge Changes:

  - Fixed the problem where the person's picture would not be displayed when the '_badge Sensor' information was changed. Originally, in order to use a badge, you had to set up a template sensor with value_template and entity_picture_template attribute fields and iCloud3 would update the state of the badge after each poll. When iCloud3 did this update, all of the badge's attributes were erased by HA.

    To see the 'Special Sensors' section in the documentation for more information about the '_badge Sensor', go [here](https://github.com/gcobb321/icloud3#special-sensors).

    To fix this, the template sensor is no longer needed and the picture for each person is now specified in the device_tracker: icloud3 platform. 

    - Delete (or rename) the '_badge Sensor'  template sensor you are now using. In the sample configuration files distributed with iCloud3, it is named  *gary_iphone_badge* and *lillian_iphone_badge* in the *sn_badge.yaml* file.

    - Specify the person's picture several ways using the *sensor_prefix_name* or *sensor_badge_picture* parameters:

      - If you are using the sensor_name_prefix parameter, add the person's picture after the custom name. For example:

        ```yaml
        sensor_name_prefix:
          - gary_iphone @ garyc, /local/gary.png
          - lillian_iphone @ lillianc, /local/lillian.png
        ```

      - If you are not using a custom name but using the default 'devicename' for all of the template sensors, add the new *sensor_badge_prefix* parameter. For example

        ```yeml
        sensor_badge_picture:
          - gary_iphone @ /local/gary.png
          - lillian_iphone @ /local/lillian.png
        ```

      - You can also continue to use your template sensor and have the value_template attribute point to the actual template sensor created by iCloud3.

        ```yaml
        - platform: template
          sensors:
            gary_badge:  
              friendly_name: Gary
              value_template: '{{states.sensor.gary_iphone_badge.state}}'  >>>> iCloud3 sensor with badge info
              entity_picture_template: /local/gary.png
        ```

- Fixed the problem where an error message was displayed in the HA log file.

  ```reStructuredText
  File "/config/custom_components/icloud3/device_tracker.py", line 1233, in _polling_loop_5_sec_device
  self.iosapp_update_flag[devicename] = False
  UnboundLocalError: local variable 'devicename' referenced before assignment
  ```

  

#### Version 1.0.1 - March 20, 2019

- Fixed a problem updating the '_badge' sensor. If the entity 'sensor.devicename_badge' did not exist, an error message was displayed. Now, if it does not exist, one will be created that can be used in your own badge type of sensor as a value_template item; e.g., 

  ```
  value_template: '{{sensor.gary_iphone_badge}}'
  ```

  *See the 'Special Sensors' section in the documentation for more information about the '_badge' sensor.*

  

#### Version 1.0.0 - March 15, 2019

- HA 89+ -- Create a *config/custom_component/icloud3* directory on the device (Raspberry Pi) running Home Assistant. Copy the  ``` device_tracker.py ``` file into that directory.
- HA 88 and earlier --   Create a *config/custom_component/device_tracker* directory on the device (Raspberry Pi) running Home Assistant. Copy the  ``` device_tracker.py ```  file into that directory and rename it ``` to ``` ``` icloud3.py ``` .

***Significant Enhancements***

1. [HA IOS App Integration]() - The IOS App forwards zone (and beacon) enter and exit notifications, background fetch location data and significant location change data to HA. HA updates the location (gps latitude and longitude) information. iCloud3 now scans for these zone and location changes every 5-seconds and processes the new information immediately. This improves location accuracy, speeds up zone change processing and reduces the number of calls to iCloud Location Services. 

   There are many times when the IOS App sends bad data to HA that contains locations that might be 2, 5 or 10 minutes old, locations with gps accuracy errors, zones that have been entered into or exited from when they really haven't, locations that are just outside of a zone that is current but no movement has really taken place, etc. iCloud3 discards all IOS App notifications older than 2 minutes, notifications  that are within 1km of a zone when no Zone Exit notification has been received, and double checks notifications that don't seem to make sense. When in a zone and location notification is received that also indicates the device is in the zone, the coordinates are changed to the zone center.

   iCloud3 will work without the IOS App installed on the device, however, this can result in delayed zone enter/exit notifications, increased gps wandering, reduced battery efficiency, and more interactions with the iCloud location services. It is recommended that it be installed on all devices using iCloud3.

2. [Dynamic Stationary Zone]() - The Dynamic Stationary Zone code was completely rewritten. The device is now put into a stationary zone if it hasn't moved significantly for 8-minutes (can be changed via the 'stationary_time' configuration variable) rather than checking from one poll to the next. The Stationary Zone distance, size and minimum movement is based on the Home zone radius. The distance is 2.5 times, the zone size is 2 times and the device's movement is 1.5 times the Home zone radius. A stationary zone can now be closer, bigger and more precise than before.

   The DSZ is now set to a passive state so it will not be used by other components (mapping, proximity) and is set to the North Poll (instead of in the Gulf of Guinea off the coast of western Africa) when initialized and not in use.

3. [Sensor Templates]() - Sensor templates were made available that were based on the device_tracker's attributes (e.g., ``` sensor.gary_iphone_distance ```) for displaying the attribute values on lovelace cards. iCloud3 now creates and updates these sensors when the device's state and attributes are changed. The new attribute's values are available immediately without relying on HA to monitor the attribute's value to see if it has been changed, and if it has, convert the template to the new value and then update the sensor entity while it is doing all the other stuff it does. A  *zone* sensor with several display formats has been added, along with a *badge* sensor that displays either the zone name or the distance from home.

   The sensors can be named several ways using new the *sensor_name_prefix* configuration parameter. For example, the distance sensor can be named with the devicename prefix from the IOS App and the device_tracker entity ( ``` sensor.gary_iphone_distance ```), the device's friendly name from the iCloud account (``` sensor.gary_distance ```) or by a custom name parameter (``` gary_iphone @ garyc ```) that creates ``` sensor.garyc_distance ```.  See the Sensor Entity section of the documentation for more information.

   It is now much easier to code automations and scripts with the new template sensors than it is with the device_tracker's state and attributes. For example, in a value_template: 

   ```  {{states.device_tracker.gary_iphone.attributes.distance | float <= 0.2}}   ```  becomes  ``` {{states.sensor.gary_iphone_distance.state | float <= 0.2}} ```

   There are times when gps wanders and you receive a zone exit state change when the device has not moved in the middle of the night. The sequence of events that takes place under the covers is (1) a zone change notification is sent by the IOS App based on bad gps information, (2) the device's state and location is changed, (3) triggering an automation that runs when you exit the Home zone. (4) iCloud3 sees the new state and location and processes the data and (5) sees the notification data is old and it was caused an incorrect state change and (6) puts the device back into the Home zone where it belongs. The net effect is HA triggers the automation before iCloud3 gets control so the correction takes place after the automation has already run.

   The solution to eliminating this problem is to not trigger automations based on device state changes but to trigger them on zone changes. A *zone* and *last_zone* template sensor, updated by iCloud3, is used to do this. These template sensors are only updated by iCloud3 so they are not effected by incorrect device state changes.  See the example *gary_leaves_zone* automation in the ``` sn_home_away_gary.yaml ``` sample file where the ``` sensor.gary_iphone_zone ``` is used as a trigger. 

   > The  ``` sn_device_tracker_attributes.yaml ``` file containing the attribute template sensors distributed in the configuration section of the iCloud3 repository must be deleted (or commented out so it will not load). The new version of the  ``` sn_badges ``` sensor template file must be used.

***Other New Features and Bug Fixes***

1. The WazeRouteCalculator component normally loads with Home Assistant and Hass.io. However, if it was not installed on your server, it would crash iCloud3. Now, if Waze is not loaded or not available  when HA starts, the Waze calculation routines are automatically turned off.
2. Added significant error trapping and notifications. A service call command has been added that will start (and stop) logging detailed debug information to the HA log file. See the service call section of the iCloud3 documentation for more information.
3. Fixed the problem where constant polling would occur late at night when the next update time rolled over to the next day. UTC time is now used to keep all timed last and next update time checks.
4. Fixed all of the issues causing an 'Internal Error' message to be written to the HA log file.
5. The poll count attribute has been changed to to report the iCloud, IOS App and discarded transaction counts. It's format is '##:##:##', e.g., '10:14:21' with the iCloud count (10) first, the IOS App (14) second and discarded transactions (21) third.
6. The number of calls to iCloud at HA startup has been reduced.
7. Information messages in the HA Log File are more informative and easier to understand.
8. The device's state is now set to it's name in zone.yaml rather than capitalizing it.
9. Added several new attributes, including travel_distance (distance traveled since the last poll), authenticated (the last time the account was authenticated), timestamp (when the update occurred) and trigger (what caused the last update). The account has been added to the tracked devices (*gary_icloud/gary_iphone*). This is useful when more than one iCloud account is used.
10. The HA *icloud_reset* Service name was changed to *icloud_restart* Service name to better explain what was actually going to be done. The *reset_icloud* name could be interpreted as resetting your Apple iCloud account, which this service does not do.
11. Fixed *icloud_lost_iphone*  alert sound so it would play the Find-my-Phone Alert Sound.



#### Version 0.86.2 - January 27, 2019

1. The iCloud3 zone state was capitalized’, i.e., *school* went to *School*, *HOME* went to *Home*. The zone’s name as it is in zone.yaml is now used without any reformatting.
2. If the location data was 'old', an "Error updating device_name" message was displayed in the log file due to inconsistent data. Additional error checking has been added.
3. Devices in a zone were repolled via Find-my-Friends if another device on the account needed to be repolled. If the device in a zone (that didn't need to be repolled) was experiencing poor GPS accuracy, that lead to device location errors and it would potentially drop in-and-out of the zone until the GPS accuracy was restored and may get into a relocate loop for several cycles. Now, the device in the zone will only be updated if there were no GPS accuracy issues.
4. *Note: There is a lot of 'Trace' code that writes records to the HA log file to determine what is causing the Old Location error. It will be removed when released. To log very debug information, use the 'info logging' command..*



#### Version 0.86.1 - 1/25/2019

1. Fixed a problem where an error returning location data from the iCloud Find-my-Friends location request would crash iCloud3 and put it into an endless retry loop.
2. Changed the way log records (information, error, debug) are written to the HA log file. Also added a new *info logging* command that starts (and stops) writing debug records to the HA log file to make troubleshooting easier.
3. Changed the coordinates of the Dynamic Stationary Zone from 0°, 0° (Papa New Guinea) to 0°,180° (where the international date line crosses the equator).
4. Add Version information to the iCloud3.py file to support the Custom Control Updater program.
5. Added a Prerelease directory to the Github Repository for early versions of iCloud3.

#### Version 0.86 - January 22, 2019

1. Fixed a problem where devices were being excluded when they shouldn't have been. For example: if you had an *include_device: gary* and an *exclude_device: garyold* in the configuration file, gary was not being included in the devices being handled by iCloud3 when it should have been.
2. Added better support for more than one iCloud3 platform for different Apple iCloud accounts. The attributes now show the correct devices being tracked and the accounts they are associated with.
3. Updated the documentation to better explain naming devices on the device (iPhone, iPad, etc.) and in the HA IOS app. Also better explained what happens if you do not use the IOS app on the device.