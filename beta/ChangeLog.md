## iCloud3 Change Log

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

  - Create a new FmF account without 2fs. Then log into the account. It is easier to do this on a device you will not be tracking so you can have the device to be tracked (using the "real" account) and the FmF device at the same time.
  - Open the Find-My-Friends app on the FmF device logged into your FmF non-2fa account.
  - Invite the friends (your "real" account) to be followed. With both devices open, you will probably be able use to AirDrop sync the devices. 
  - Change the device_tracker.yaml file to use new FmF account, username and password.
  - Set up the devices to be tracked (your "real" account) using the *track_devices* parameter.

  

  **tracking_method**  
  iCloud3 supports three methods of tracking a device -- iCloud Location Services, Find-My-Friends and the HA IOS App version 1 and version 2 (version 2 not available yet).

  *Valid values: fmf, icloud, iosapp1, iosapp2*

  *Default value: fmf*

  

  **track_devices**  (or  **track_device**) *Required* 
  Identifies the devices to be tracked, their associated email addresses, badge pictures and a name that should be uses for the device's sensors.  This replaces the *include_devices & include_device_types* parameter.

  *Format:*
    ```devicename > email_address, badge_picture_name, reserved, sensor_prefix_name```

  

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
  
  


```

```