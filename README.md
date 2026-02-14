# CalSyncHAB

Synchronization Of Google Calendar Events With OpenHAB Items

### License

Software licensed under GPL version 3 available on http://www.gnu.org/licenses/gpl.txt and with kind regards to https://github.com/davorf who started this script.

### 
### Usage
Since I (and others) couldn’t make openHAB's CalDAV Personal binding to work with Google Calendar, and, according to this https://community.openhab.org/t/caldav-google-calendar-problem/9219/101?u=davorf, there is a problem with CalDAV Personal binding in OH, https://github.com/davorf made a simple Python script that authenticates via OAuth2, and uses Google Calendar API to populate OH items with an event information. I simply added the ability to use multiple Google Calendars from your one Google Account. Also I added some OH} sample items and rules for being used with the script. Now we are able to not only show google calendar events within openHAB but also use the events to update the status of items and cause actions.

### Installation
#### Step 1: Create OAUth2 Service Account
First, you need to create OAuth2 Service Account (https://console.developers.google.com), download servie-account file in JSON format and place it in CalSyncHAB folder (cloned or downloaded from a github). 
In detail: go to https://console.developers.google.com/apis, and then choose Service Account on the menu (located on the top of the screen). Name your Service Account and give it an ID. Copy the mail address that is shown below the ID. Click "Create and Continue". In the next screen choose "Owner" as role and save your service account. From the console choose your new Service Account, go to "key" and add a new private key. In the next screen choose "JavaScript Object Notation" and click "Create". A file window will be opend by your browser to save your service-Account json file. You will need this file in step 4.

#### Step 2: Install google api
After installing Python3 (sudo apt install python3), you need to install pip (sudo apt-get install python3-pip) and following packages:

    sudo python3 -m pip install --upgrade google-api-python-client
    sudo python3 -m pip install --upgrade oauth2client
    sudo python3 -m pip install configparser
    sudo python3 -m pip install requests

Depending on your privileges on a linux machine you might have to use these commands with 'sudo'.
Using linux you might come across a problem with pip which is descibed here: https://bugs.launchpad.net/ubuntu/+source/python-pip/+bug/1306991. To work around this you have to:

    sudo easy_install --upgrade pip

#### Step 3: Setup CalSyncHAB.ini
In the CalSyncHAB.ini you need to set:

    Application name - name of the application that will ask for an authentication
    Scope - should not be changed
    [CalendarIDs]
    id1 = abc@group.calendar.google.com
    id2 = xyz@group.calendar.google.com
    
Put in here the list of your calendars you want to retrieve information from (if you have only one calendar, “primary” keyword will suffice, if not, you have to use Calendar IDs shown in the settings of that calendar). You may use as many calendars as you like.

    MaxEvents - maximal number of events retrieved (starting from current date and time)
    TimeZone - in format ±HH:mm
    ClientServiceFile - name and path of the OAuth2 service-sccount.json file you saved in step 1.
    HostName - IP address or host name of the OH} server
    Port - Port of the OH server (usually 8080)
    ItemPrefix - prefix of the calendar items in OpenHAB - for example, if you use ItemPrefix gCal_ you should create following items:
    gCal_Event1_Summary (String)
    gCal_Event1_Location (String)
    gCal_Event1_Description (String)
    gCal_Event1_StartTime (DateTime)
    gCal_Event1_EndTime (DateTime)

If you’ve set MaxEvents to 10, you should have 10 sets of those items - for example gCal_Event1_Summary to gCal_Event10_Summary. You will find a sample .items file in this git.

#### Step 4: Setup files and folders
Windows users do not need the chown and chmod commands
- Create a folder “CalSyncHAB” within /etc/openhab/scripts
- Put the python scripts, .ini file and the .json file into this new folder
- Set owner of the new folder:
    ```
    chown openhab:openhab /etc/openhab/scripts/CalSyncHAB
    ```
- Set owner of all your files within the new folder
    ```
    cd CalSyncHAB
    chown openhab:openhab *
    ```
- open CalSynHAB.ini and set property “ClientSecretFile” to the full path where you placed the secret.json file, e.g. 
    ```
    ClientSecretFile: /etc/openhab/scripts/CalSyncHAB/CalSyncHABSecret.json
    ```
- create a shell script “/etc/openhab/scripts/CalSyncHAB.sh” with the following content: 
    ```
    #!/bin/sh
    /usr/bin/python /etc/openhab/scripts/CalSyncHAB/CalSyncHAB.py
    ```
- Set rights and owner for the shell script:
    ```
    chown openhab:openhab /etc/openhab/scripts/CalSyncHAB.sh
    chmod +x /etc/openhab/scripts/CalSyncHAB.sh
    ```

### Samples
You wil find two sample files for openHAB:
    samples/items/calendar.items
    samples/rules/calendar.rules

calendar.items contains all items needed in order to retrieve 10 calendar events from your google calendar(s).
calendar.rules contains the following 3 rules:
- 'GetCalEvents' repeats every half an hour and retrieves your calendar events
- 'ActCal' also retrieves your events but will only be activated when switching item calAct from OFF to ON.  You can place this item into your sitemap for a quicker access to new calendar entries (very fine for testing).
- 'ExecCal' executes the commands you specified in your events. 

'ExecCal' uses the following items:
- **gCal_EventX_Summary** must contain the name of an openhab2-item which you want to update.
- **gCal_EventX_Description** must contain the values which the item will be updated with. You must specify two values, one for the event start and one for the end seperated by a semicolon, e.g. 'ON;OFF' (without '). If you don't want the item to be updated in one of the two cases use '-', e.g. '50;-'. 
- **gCal_EventX_Location** may contain a condition which will be checked before updating your item. No action will take place if the condition is false. I am using my presense detection switch (named: 'aPersonIsPresent') to stop my shades from moving every morning if nobody is at home. A condition has to contain  the name of an openhab-item, a relational operator and a value to be compared with. All these have to be sperated by a space. Example: 'aPersonIsPresent == ON'. Actually relational operators can only be '==' (equals) and '!=' (unequal). If you need more complex conditions you should write your own rules which set switch items to ON or OFF if your condition is true or false. These switch items can later be used for the condition here. Condition must not be used. 
- **gCalEventX_StartTime** gives the time when your event starts. At this time the first part of the description (to the left of the semicolon) will update your item.
- **gCalEventX_EndTime** contains the time when your event ends. At this time the second part of the description (to the right of the semicolon) will update your item.
- **gCalEventX_CalId** contains the ID of the calendar the event was retrieved from. ID the identifier from your CalSyncHAB.ini file within the section [CalendarIDs].

### Time Zone
Be careful (with that axe Eugene) when creating new events or editing start or end time of events when you are staying in a time zone other than your home location. Google normally uses the time zone where you are located while creating new events. If this is another zone as your home location events will start and end at wrong times as 'ExecCal' uses the fully qualified start and end times. You can modify the time zone while creating new events within your google calendar.
