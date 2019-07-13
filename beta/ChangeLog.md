## iCloud3 Change Log

#### Version 1.1 0 Beta 3 - July 13

**Changes:**

- If the Find-my-Friends account id/password was incorrect when logging into the iCloud account, iCloud3 and the Pyicloud_ic3 iCloud Service Handler would generate all kinds of error messages and and nothing would work. This has been corrected to display error messages and fall back to using the IOS App tracking method..

- Changed the *tracking_method* value for the iCloud Find-my-Phone Location Service from 'icloud' to 'fmphn' for consistency with 'fmf'.

- Changed the way a date is displayed to see if iCloud3 would run on a Windows based linux platform.

- Added a *base_zone* configuration parameter. See below for more information on the *base_zone*. 

  

**Still to do:**

- Update the iCloud3 documentation manual.

  

**Configuration Parameter Updates**

**base_zone**  
Normally, the 'home' zone is the zone used to calculate distances, travel times, update intervals, etc. The *base_zone* lets you do this for a zone other than the 'home' zone. You can, for example, set up a second iCloud3 platform using the same FmF login username/password as you normally do but have the *base_zone* set as a work zone, a second home zone or another zone you want to use as the basis for device tracking calculations. Another Lovelace card can then be set up to display all of the distances, travel times, etc for that zone.  

*Valid values: zone names*

*Default value: home*

 *Format:*  `base_zone: otherzonename, sensorzoneprefix`

*Example:* `base_zone: office, offc`

  

| Field            | Description/Notes                                            |
| ---------------- | ------------------------------------------------------------ |
| otherzonename    | *Required* The other zone to be used as the basis for calculating distances, travel times, intervals, etc. |
| sensorzoneprefix | *Optional*  Sensors for the 'home' zone are normally named as `sensor.devicename_fieldname `. The *sensorzoneprefix* specifies the text to be added to the sensor to make it different then the 'home' zone sensor. The name will become `sensor.devicename_sensorzoneprefix_fieldname`. For example, the sensor for the travel time to 'home' is `sensor.gary_iphone_travel_time` and the travel time to the 'office' zone might be `sensor.gary_iphone_offc_travel_time`  if 'offc' is the text you specified for the *senorzoneprefix*. <br><br>*Note:* If you do not use the *sensorzoneprefix*, the sensor is named as it normally is. e.g., `sensor.gary_iphone_travel_time`. It all depends on how you set up the iCloud3 platform and your Lovelace cards that monitor the iCloud3 sensors.<br><br>*Note:* The field names remain the same across all sensors. Of special importance is that the 'home_distance' sensor is the distance to zone, whether the zone is 'home' zone or another zone. |


------


#### Version 1.1 0 Beta 2 - July 11

**Changes:**

- Added step 2 in the set up instructions above to add contacts in the FMF account for the devices to be tracked.
- Added additional messages to the HA log file to better describe the process of matching the devicename-email address to the FmF contact-email address. The messages better describe what is going on and clarify what needs to be done if a devicename can not be matched to a contact (and thus to a device's location)
- Added the ability to exclude sensors you don't use or don't want to see from being created. Also added the ability to specify what sensors you want to see. More about this in the documentation later.

**Still to do:**

- Update the iCloud3 documentation manual.

  

------

#### Version 1.1.0 Beta 1 - July 10

**Breaking Changes**

- Location tracking method *Find-My-Friends* has been added.

- iCloud3 uses a customized version of  the *pyicloud,py* module to interact with Apple iCloud Location Services that supports Find-My-Friends. It must be installed in the same directory containing the iCloud3 custom component. You will need to install both the *device_tracker.py* and *pyicloud_ic3.py* files into the *custom_components/icloud3* directory.

- A new *tracking_method* has been added to select how iCloud3 should track devices (iCloud, Find-My-Friends, IOS App). Default is using Find My Friends.

- A new *track_devices/track_device* has been added to specify the devices that should be tracked.

- The following configuration parameters have been depreciated and are no longer used:

  - include_devices/include_device
  - include_device_types/include_device_type
  - exclude_devices/exclude_device
  - exclude_device_types/exclude_device_type
  -  filter_devices
  - sensor_name_prefix
  - sensor_badge_picture

  These have all been replaced by the new *track_devices/track_device* parameter

- *sensor_distance* entity has been renamed to *sensor_home_distance* to avoid conflicts with the IOS App version 2. You will need to change the name in any automations, scripts and lovelace.yaml files using the *sensor_distance* entity.

**Find-My-Friends (FmF) vs iCloud Location Services**

iCloud Location Services uses the *pyicloud.py* module to connected HA to the Apple iCloud account. In June, 2019, Apple  began logging Home Assistant components out of their account after 30-minutes. To locate a device, iCloud3 has to authenticate and log into the account when it is polled. Logging into an account using 2-factor authentication caused an annoying notification message to be sent to all devices associated with the account. Using Find my Friends with a non-2fa account eliminates this notification and retrieves the location of the device. 

To Use FmF, you will need to set up a new Apple iCloud account with a new email address. You will then set your "real" account (the one you were using before) as a friend to be tracked with your new FmF account. You can set up as many friends as you want and it can be a one-way-street, i.e., the new FmF account will track your "real" account but your "real" account does not have to track your new FmF account.

**Setting up your iCloud Account to use Find-My-Friends tracking method**

- If you are not using 2-factor authorization for your "real" iCloud account, you can continue use that account to locate your device (use *tracking_method: icloud*). If you are using 2fa, however, you will need to create a new iCloud account without 2fa and then use this as the account in iCloud3 device_tracker.yaml configuration file with the *track_devices* parameter. 

- Generally you will:

  1. Create a new FmF account without 2fs. Then log into the account. It is easier to do this on a device you will not be tracking so you can have the device to be tracked (using the "real" account) and the FmF device at the same time.
  
  2. Open the Contacts App and add a contact for the friends you will be following. The only information you need to enter is the first & last name and the email address of the "real" icloud account (the one on the *track_devices* configuration parameter). If you don't add the contact (or the contact's email address is wrong), the email address on the *track_devices* email parameter (and thus the devicename) can not be matched and the devicename will not be tracked. A warning message is added to the HA log file if this happens. 
  
     *Note: FmF uses an ID code (e.g., “MTg2Djk7ODEq”) to identify contacts (friends) with their location data. During initialization, iCloud3 matches the email address in the devicename’s *tracked_devices* configuration parameter to the email address in the contacts (friends) record to get the ID's that will tie everything together. 

  3. Open the Find-My-Friends app on the FmF device logged into your FmF non-2fa account.

  4. Invite the friends (your "real" account) to be followed. With both devices open, you will probably be able use to AirDrop sync the devices. 
  
5. Change the *device_tracker* configuration file parameters to use new FmF account, username and password.
  
6. Set up the devices to be tracked (your "real" iCloud account) using the *track_devices* parameter.



**tracking_method**  
Select the method to be used to track your phone or other device. iCloud3 supports three methods of tracking a device -- iCloud Find-My-Phone Location Services, iCloud Find-My-Friends Location Services and the HA IOS App version 1 and version 2 (version 2 not available yet). 

*Valid values: fmf, fmphn, iosapp1, iosapp2*

*Default value: fmf*

  

**track_devices**  (or  **track_device**) *Required* 
Identifies the devices to be tracked, their associated email addresses, badge pictures and a name that should be uses for the device's sensors.  This replaces the *include_devices & include_device_types* parameter.

 *Format:*   `devicename > email_address, badge_picture_name, reserved, sensor_prefix_name`

  

| Field         | Description/Notes                                            |
| ------------- | ------------------------------------------------------------ |
| email_address | *Required for the tracking_method: fmf*. FmF uses the email address of the "friends" iCloud account to link it with the FmF account. The email address is linked to the device's entity here. |
| | You can use the complete email address ```(gary456@gmail.com)``` or just the name part ```(gary456```). |
| | Not required if you are using the icloud or iosapp tracking methods. (You can omit the comma placeholder) |
| badge_picture_name | The file name containing a picture of the person normally associated with the device. See the sensor_badge section of the iCloud3 documentation for more information about the sensor_badge. |
| | The file is normally in the ```config/www/``` directory (referred to as '/local/' in HA). You can use the full name ```(/local/gary.png)``` or an abbreviated name ```(gary.png)```. |
| reserved | Reserved for future use. |
| sensor_name_prefix | See the Sensor section of the documentation for a complete description of this field. (garyc in the example below) |

  

  ```yaml
  Examples:
  
    - gary_iphone > gary456@gmail.com, /local/gary.png
    - gary_iphone > gary456, gary.png
    - gary_iphone > gary456, gary.png,, garyc
    - gary_iphone > ,gary.png,, garyc
	  - gary_iphone > /local/gary.png
		- gary_iphone > gary.png
	  - gary_iphone  (for icloud or iosapp tracking method if you are not using a badge)
	 
	Notes:
	  - A greater then sign ('>') separates the devicename from the parameters.
    
    
  Example of minimum device_tracker.yaml (using tracking_method: fmf):
  
  - platform: icloud3
    account_name: gary456_icloud
    username: gary456@gmail.com
    password: gary456pw
    track_devices:
      - gary_iphone > garyrealemail@gmail.com, gary.png
   	- lillian_iphone > lillianrealemail@gmail.com, lillian.png
  ```
