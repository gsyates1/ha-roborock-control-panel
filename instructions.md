# Instructions
### Accessing the Files
You can either clone this repository to your machine or cut and paste straight from GitHib.

### Conventions
In code given in these instructions, items in double pointy brackets (<<>>) need to be replaced with your HA values.  For example, if you see <<vacuum's entity ID>> you would replace it with whatever your vacuum's entity ID is in your config (e.g. vacuum.roborock_qrevo_pro). 

### Categories
I highly recommed creating a 'Vacuum' or similar category for each type of HA object (automation, script, helper, etc.) and using it for all objects created for this dashboard; ***there will be a lot of them***.

### Create a Day-of-week Sensor
The vacuum control panel needs the day of the week by name as a sensor.  If you do not already have one named sensor.day_of_week you can create one by adding the YAML in config/day_of_week.yaml to your configuration.yaml under the sensors section.

### Create a Global Variables Sensor
This will create a sensor into which global variables can be stored. Variables can be set or removed using actions set_variable and remove_variable on the sensor. All variables can be cleared using clear_variables.  Note this is not limited to this dashboard.  This can be used in any of your HA automations and scripts.

1. Copy and paste the contents of the file config/templates.yaml in this repository into either your templates.yaml file or your configuration.yaml file under the template: section. (Be sure to fix the indentation if it's the latter.)  If you don't have a templates.yaml file, or templates: section, you will need to create one or the other. See the HA docs.
2. Reload your config YAML using the Develpoer Tools YAML tab.

### Create Room Dictionaries
The rooms in a Roborock map have numeric IDs.  These IDs are needed to tell the vacuum what room to clean.  But, these IDs are not very intuitive and can change. For example, they might change if you need to rebuild your room map.

This step creates some global variables to hold a dictionary of room names to Roborock IDs and a dictionary of room names to internal IDs.  The internal IDs will then be used in the scripts for this dashboard.  Doing this means if the Roborock room IDs change, only this script has to be fixed.

1. Get your vacuum's room IDs.  In the Devoloper Tools Actions tab, execute this action:
```
action: roborock.get_maps
target:
  entity_id: <<vacuum's entity ID>>
data: {}
```  
You should get a result similar to this:
```
vacuum.roborock_qrevo_pro:
  maps:
    - flag: 0
      name: First Floor
      rooms:
        "16": Unknown
        "17": Entry & Library
        "18": Bedroom
        "19": Nook & Bar
        "20": Great Room
        "21": Laundry
        "22": Unknown
```
Record the room numbers for use later.

2. Disovering 'unknown' rooms.  It is very possible that you have some rooms marked as 'Unknown' like in the example above.  You need to discover what these rooms are.  You may be able to do this by a process of elimination.  If not, the easiest way to find the missing IDs is to send the vacuum to clean a room by ID and see where it goes:
	1. In the Roborock app make sure the vacuum is set to only vacuum so it's not needlessly wetting the mop.
	2. Execute the following in the Devoloper Tools Actions tab, inserting one of the missing room IDs:
```
action: vacuum.send_command
target:
  entity_id: <<vacuum entity id>>
data:
  command: app_segment_clean
  params:
    - segments:
        - <<room id>>
      repeat: 1
``` 
	3. Watch which room the vacuum goes to.  Hit the vacuum's return to dock button.
	4. Repeat for each unknown room, recording the IDs.
3. Create the room mapping script.
	1. Create a new script in HA.
	2. Copy the contents of the scripts/create_vacuum_room_number_map.yaml file from this repository into your script.
	3. Edit the script changing the first 'value:' section to the IDs you captured above and using the room names you want to see in the dashboard.  They do not have to match the Roborock app room names.
	4. Edit the second 'value:' section to use the same room names.  Leave the room numbers in this section as consecutive numbers.  These are the numbers that the other scripts will use. 
	5. Save your script as 'Create Vacuum Room Number Map'
4. Run the script to create the variables.
    1. Manually run the script.
    2. In the Developer Tools States tab, search for 'sensor.variables'.  In the attributes column you should see something like this:
```
vacuum_roborock_room_number_map:
  Dining Room: 16
  Entry & Library: 17
  Great Room: 20
  Kitchen: 22
  Laundry Room: 21
  Master Bedroom & Bath: 18
  Nook & Bar Area: 19
  Craft Room: 16
  Bridge & Hall: 20
  Guest Room: 18
  Computer Room: 17
  Bonus Room: 19
vacuum_room_number_map:
  Dining Room: 1
  Entry & Library: 2
  Great Room: 3
  Kitchen: 4
  Laundry Room: 5
  Master Bedroom & Bath: 6
  Nook & Bar Area: 7
  Craft Room: 8
  Bridge & Hall: 9
  Guest Room: 10
  Computer Room: 11
  Bonus Room: 12
```

### Create Helpers
Helpers need to be created for most of the cards that are on the dashboard.  There are a lot of them so they are created using YAML rather than in the UI.  

1. The contents of the files below need to be copied and pasted into one of two places:
	1. The corresponding config files: input_boolean.yaml, input_datetime.yaml, etc.
	2. Your configuration.yaml file under the appropriate section: input_boolen:, input_datetime:, etc.  Be sure to fix the indentation if you use this option.

If you don't have the config files or the needed sections in configuration.yaml, you will need to create one or the other. See the HA docs.
```
config/input_boolean.yaml
config/input_datetime.yaml
config/input_number.yaml
config/input_select.yaml
config/input_text.yaml
config/timer.yaml
```
2. Reload your config YAML using the Developer Tools YAML tab.
3. Verify that the helpers were created in HA (Settings > Devices & Services > Helpers). If they don't show up, you may need to restart Home Assistant.

### Scripts
Add the vacuum scripts to your HA config and modify them for your setup.

1. Copy the contents of the script/scripts.yaml file into your scripts.yaml file.  Alternatively create each script in the UI and paste in the applicable YAML from the above file. There are eight scripts.
2. In the Developer Tools YAML tab, reload your YAML config.
3. Verify the scripts now appear in HA under Ssttings >> Automations & Scenes >> Scripts.  If not, you may need to restart HA.
4. Make the following changes to the scripts:
	1. In the script 'Vacuum Run Announce' there is an 'if' statement that checks if the flag 'Someone Room' is set.  You will need to change this to check whatever condition you use to determine if someone is home.  If you do not have such a mechanism or want the vacuum to run when no one is home, remove this condition. (Only this condition, not the entire 'if' statement.)
	2. In the script 'Vacuum Run Check', you need to make the same change you made in the previous step.
	3. In the script 'Vacuum Run Announce' there is an action that references a script called 'TTS'.  This script is not provided. You need to replace this action with the mechanism you use for whole-house announcements.  The message parameter of this action has an example template to build a message to announce.  This template is easiest to read if you copy it into the Developer Tools Template tab.  You can manually set helpers input_text.vacuum_0_room, input_text.vacuum_0_room_number, and input_number.vacuum_0_announce_offset to test the template.
	4. In the script 'Vacuum Problem Announce', there is an action called 'Define variables vacuum_name'.  You need to modify the 'if' statement condition in its template to use the Friendly Names of your vacuum's error sensor and the dock's error sensor.  (i.e. Replace: 'Roborock Qrevo Pro Dock Error' and 'Roborock Qrevo Pro Vacuum error' with your sensor names).  You can ignore the names in the 'elif' part of the template unless you are setting up two vacuums.  Also change 'downstairs vacuum' to be whatever you want the message to call your vacuum.
	5. In the script 'Vacuum Problem Announce' there is an action that references the script 'Add Reminder'.  This script is not provided. You need to replace this action with the mechanism you use for whole-house announcements.  The message parameter of the script has an example template to build a message to announce.

### Automations
Add the vacuum automations to your HA config and modify them for your setup.

1. Download or copy the file automations/automations.yaml to a file on your machine.  It's easiest to edit it before adding it to HA.
2. In the downloaded file, search for \<\<vacuum entity id\>\> and replace it with the entity id of your vacuum (e.g. vacuum.roborock_qrevo_pro).
3. Search and replace \<\<vacuum error sensor>> with the entity id of your vacuum's error sensor (e.g. sensor.roborock_qrevo_pro_vacuum_error).  
4. Search and replace \<\<vacuum dock error sensor\>\> with the entity id of the dock's error sensor (e.g. sensor.sensor.roborock_qrevo_pro_dock_error).
5. Copy the contents of the file you changed into your automations.yaml file.  Alternatively create each automation in the UI and paste in the applicable yaml from the above file. There are two automtions.
6. In the Developer Tools YAML tab, reload your YAML config.
7. Verify the automations now appear in HA under Settings >> Automations & Scenes >> Automations.  If not, you may need to restart HA.
8. In the automation 'Vacuum Problem Announce' there is a condition that is checking that the input boolean 'Sleep Mode' is off.  If you don't want announcements in the middle of the night, you need to replace this with a condition using your functionality. Otherwise, you can remove this condition.

### Add a Midnight Action
The two input booleans 'Vacuum Ran Today' and 'Vacuum Announced Today' need to be reset at midnight so the vacuum functionality will work the next day.

If you already have an automation that runs at midnight you can add the action below to it.  If not, you will need to create an automation triggered at midnight and add this action to it.

Action code:
```
action: input_boolean.turn_off
metadata: {}
data: {}
target:
  entity_id:
    - input_boolean.vacuum_ran_today
    - input_boolean.vacuum_announced_today
```

### Create the Dashboard
These instructions assume the dashboard will be created as a subview and you will have a card on some existing dashboard that uses a navigate action to open the vacuum control panel.  If you want something else, You will need to adjust these instructions and the YAML.

1. In Settings > Dashboards either pick an exiting dashboard or create a new dashboard to add the vacuum control panel to.
2. Click the dashboard to open it.
3. Click the pencil in the upper right to edit the dashboard.
4. From the three-dots menu in the upper right, select 'Raw configuration editor'.
5. Copy the contents of dashboards/vacuum.yaml and paste it at the bottom of the existing yaml.
6. You now need to fix all the vacuum and sensor entity IDs in the YAML to reflect your entity IDs.  If your entities are using the default IDs which should have the format \<type\>.\<vacuum device name\>\_\<sensor name\> and \<type\>.\<vacuum device name\>\_dock\_\<sensor name\> then you may be able to do a single search and replace to fix all the entity IDs.  Search for 'roborock_qrevo_pro' and replace it with your vacuum's device name.  For example, if your vacuum device name is 'bob_the_robot' and your vacuum entity ID looks like vacuum.bob_the_robot and sensors like sensor.bob_the_robot_battery, search for 'roborock_qrevo_pro' and replacing it with 'bob_the_robot'.  If your entity IDs do not follow this pattern, you will need to go through the YAML line-by-line and fix the entity IDs.
7. Close the YAML editor and you should see the dashboard in edit mode.  Check to see if all cards have values.
8. Click 'Done' to return to normal mode.  Note that the problem cards, the vacuum control buttons (stop, pause, reume, return to dock), and progress card are in conditional cards so they will only show if the vacuum has a problem or is actively running, respectively.

### Configuring Your Vacuum
You should now be able to configure your vacuum using the control panel.

### Start the Timer
When you are ready for your vacuum schedule to run, manually start the 'Vacuum Run Check' timer.  Watch this timer and make sure it is restarting every minute.

You should also make sure this timer automatically runs when HA starts.  If you already have an automation that triggers when HA starts, add the action from the file automations/startup.yaml.  If you don't have a startup automation, you can create one use the contents of the above file. 
 
### Handling Multiple Vacuums

The basic functionality is in the scripts to suport two vacuums.  (In theory more can be added, but it will take a bit more work that listed here. Just follow the same pattern.)

**NOTE: Make sure your first vacuum control panel is working before adding the second vacuum because you will be copying the dashboard.**

To add a second vacuum, do the following:

1. Edit each of the helper files (or your configuration.yaml if you put them there) and make copies of all the helpers EXCEPT the ones below.  Change the names and entity ids of the copies by adding a '2' after 'vacuum'.  For example:
```
Change:
	input_boolean.vacuum_0_day_enabled  
	  name: Vacuum 0-Day Enabled
	  icon: mdi:toggle-switch-off-outline)
To:
	input_boolean.vacuum2_0_day_enabled  
	  name: Vacuum2 0-Day Enabled
	  icon: mdi:toggle-switch-off-outline)
```
Do not copy these helpers:
```
Vacuum 0-Announced Today
Vacuum 0-Ran Today
Vacuum 0-Fan Speed
Vacuum 0-Mop Intensity
Vacuum 0-Mop Mode
Vacuum 0-Vacuum Number
```
2. Copy the contents of automations\multi_vacuums_run_check.yaml and paste it into the automation 'Vacuum Run Check' after the actions already in the file.
3. Copy the contents of automations\multi_vacuums_problem_announce.yaml and paste it into the automation 'Vacuum Problem Announce' after the triggers that are already there.
4.  In Settings > Dashboards open the exiting dashboard where your vacuum control panel subview is.
5. Edit the dashboard.
6. From the three-dots menu select 'Raw configuration editor'.
7. Copy entire set of YAML for your existing vacuum control panel and paste it at the bottom of the existing yaml or another dashboard YAML file.
8. You now need to fix all the helper entity IDs in the YAML to reflect second vacuum helpers.  You should be able to search and replace 'vacuum' to 'vacuum2' and 'Vacuum' to 'Vacuum2', ***but be careful not the change your original dashboard if their in the same file***.
9. You also need to fix the vacuum and sensor entity IDs the way you did for the first vacuum.  See step 6 in the 'Create the Dashboard' section above.
10. If your room dictionay does not already have the rooms for the second vacuum, you need to update it.  Modify the script 'Create Vacuum Room Number Map' to add the rooms and then run it.  If you need to get the room ids, refer back to the 'Create Room Dictionaries' section.
11. In the script 'Vacuum Problem Announce', there is a 'Define variables vacuum_name' action.  For the first vacuum you modified the friendly names in the 'if' statement condition to be your vacuum's and dock's problem sensors, now you need to do the same for the names in the 'elif' part of the template.  Change the names to your second vacuum's and dock's error sensors.  Also change 'unknown vacuum' to be whatever you want the message to call your second vacuum.

