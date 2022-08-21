+++
title = "A brief overview of OpenHAB"
#description = ""
date = "2015-06-15T15:28:45+00:00"
#updated = ""
#slug = ""
weight = 0
draft = false
#path = 

aliases = [
    "/?p=321",
    "/2019/02/27/a-brief-overview-of-openhab/",
    "/2015/06/15/a-brief-overview-of-openhab/"
]
in_search_index = true

tags = [
    "Automation",
    "Infrastructure",
    "Home",
    "Home Automation",
    "Linux"
]


+++

# What is Openhab?

OpenHAB is a Java based home automation system that allows you to control multiple technologies all from a single location. It is built on the principal of a single “Home automation Bus” where different technologies communicate using a common “Event bus”.

This means that you are able, on any device that can run the JVM, you can run your home automation, without the need to purchase expensive controllers, however, with OpenHAB manual work is required to get the system working, as “batteries are not included”.

OpenHAB is OpenSource which means that the community as a whole are able to fix bugs, and unlike Cloud based services, or propitiatory controllers, you do not need to rely on the company (or in this case, team of people) to still be around to do feature improvements, the community as a whole can do so.

# What can OpenHAB connect to?

OpenHAB has a plethora of addons that are available, ranging from HomeAutomation protocols (such as ZWave, EnOcean, RFXCOM (Lightwave RF can be controlled using this)). Some consumer devices are also able to be integrated (Sonos, Samsung TVs, LG TVs) It also has the ability to integrate with various notification systems including EMail,XMPP (Jabber / Google Talk), NotifyMyAndorid. It can also integrate with Calendaring systems like Google Calendar

# Lets dive in!

For the examples that we’re going to use in the this document we will use ZWave based equipment

The hardware that will be in use is as follows

* CentOS 7 Virtual Machine (this could be done on any operating system, and any hardware, such as a Raspberry Pi)
* [Z-Wave.me USB Stick](http://www.vesternet.com/z-wave-me-usb-stick)
* NorthQ [Electricity](http://www.vesternet.com/z-wave-data-logger-for-e-meters)and [Gas](http://www.vesternet.com/z-wave-data-logger-for-gas-meters) Meters
* [Aoen Labs MultiSensor (Gen5)](http://www.vesternet.com/z-wave-aeon-labs-multisensor-gen5)
* [Qubino Relay](http://www.vesternet.com/z-wave-qubino-flush-relay-x-1)

# Initial Setup of OpenHAB

As OpenHAB doesn’t have an installer it has to be installed by hand. We’re going to do this into `/opt`

```bash
# Create directory structure
cd /opt
mkdir /opt/openhab /opt/oh-addons-dist

# Download Runtime
curl -LO https://bintray.com/artifact/download/openhab/bin/distribution-1.7.0-runtime.zip

# Download Addons
curl -LO https://bintray.com/artifact/download/openhab/bin/distribution-1.7.0-addons.zip

# Unzip app
cd /opt/openhab
unzip /opt/*runtime.zip

# Unzip addons
cd /opt/oh-addons-dist
unzip /opt/*addons.zip
```

# Installation of Addons

We’re going to copy the addons mentioned above into place

```bash
cd /opt/openhab/addons

mv /opt/oh-addons-dist/org.openhab.persistence.rrd4j-1.7.0.jar /opt/openhab/addons/
```

#### Installation of HABMIN

OpenHAB does not provide a WebUI for administration so we’re going to install a community project called HABMin to provide us this

```bash
# Install habmin
cd /opt/openhab/addons
curl -Lo org.openhab.io.habmin-1.7.0.jar https://github.com/cdjackson/HABmin/blob/master/addons/org.openhab.io.habmin-1.7.0-SNAPSHOT.jar?raw=true
curl -Lo org.openhab.binding.zave-1.7.0.jar https://github.com/cdjackson/HABmin/blob/master/addons/org.openhab.binding.zwave-1.7.0-SNAPSHOT.jar?raw=true
cd /opt/openhab/webapps/
curl -Lo /opt/openhab/webapps/habmin.zip https://github.com/cdjackson/HABmin/archive/master.zip
unzip habmin.zip
mv HABmin-master/ habmin

```

# Configuration of OpenHAB

At this point we have OpenHAB installed, we now however need to provide it a Configuration file, which should be in `/opt/openhab/configurations//openhab.cfg`

You will need to use your favourite Text Editor (vi, nano, emacs) to edit this file

```ini
zwave:port = /dev/ttyACM0
zwave:healtime = 2
zwave:masterController = true
tcp:refreshinterval=250
folder:items=10,items
folder:sitemaps=10,sitemap
folder:rules=10,rules
folder:scripts=10,script
folder:persistence=10,persist
security:option=OFF
persistence:default=rrd4j
mainconfig:refresh=60
chart:provider=default
logging:pattern=%date{ISO8601} – %-25logger: %msg%n
```

In this file we’ve set the ZWave controller to be the first ACM device (`/dev/ttyACM0`), if you don’t have any 3g Modules or other ZWave controllers plugged into this machine this will (likely) be the device you would need to use.

We are also disabling security for the purpose of this demo, you may want to enable this going forward!

# Starting OpenHAB

This is done using the start.sh script `/opt/openhab/start.sh`

# Setup of ZWave Devices

We’re going to use HABMin to handle the inclusion of devices into the network.

* Open a webbrowser to the machines IP address into the /habmin/ folder. For instance if this is 1.2.3.4 the address you would go to is [http://1.2.3.4/habmin/](http://1.2.3.4/habmin/)
* Click the Configuration tab.
* Click Bindings (Bottom Left corner)
* Click the ZWave binding
* Click the Devices Tab

For each device you wish to add, click the include button and follow the instructions for the device to include it into the network.

Once this is completed the Devices tab should show the list of devices, similar to below (be aware this has more devices that we’re going to look at configuring!)

# Persistence

It is useful to be able to see what an item’s previous value has been, and as such we want to store these so that they survive over restarts. We’re going to use the rrd4j addon for this. It is configured by putting the following in `/opt/openhab/configurations/persistence/rrd4j.persist`

```
// persistence strategies have a name and a definition and are referred to in the “Items” section

Strategies {
 everyHour : “0 0 * * * ?”
 everyDay : “0 0 0 * * ?”
 everyMinute : “0 * * * * ?”

 // if no strategy is specified for an item entry below, the default list will be used
 default = everyChange
}

/*
 * Each line in this section defines for which item(s) which strategy(ies) should be applied.
 * You can list single items, use “\*” for all items or “groupitem\*” for all members of a group
 * item (excl. the group item itself).
 */

Items {
 // persist all items once a day and on every change and restore them from the db at startup
 * : strategy = everyChange, everyMinute, everyDay, restoreOnStartup
}
```

# Items

In OpenHAB an item is a individual attribute of a device that is configured within OpenHAB.

The first set of Items we will configure is the AeonLabs multi Sesnsor

We need to create a file in `/opt/openhab/configurations/items/`. Lets create `/opt/openhab/configurations/items/livingroom.items`

Within this file we define a set of items

```
Number sensor_1_temp “Temperature [%.1f °C]” {zwave=”3:command=sensor_multilevel,sensor_type=1″} 
Number sensor_1_humidity “Humidity [%.0f %%]” {zwave=”3:command=sensor_multilevel,sensor_type=5”}
Number sensor_1_luminance “Luminance [%.0f Lux]” {zwave=”3:command=sensor_multilevel,sensor_type=3”}
Contact sensor_1_motion “Motion [%s]” {zwave=”3:command=sensor_binary”}
Number sensor_1_battery “Battery [%s %%]” {zwave=”3:command=battery”}
```

The format of these files is as follows

`ItemType ItemName ItemLabel <ItemIcon> (ItemGroup) {ItemBinding}`

We are not going to cover off Icons or Groups in this guide.

The ItemTypes that are available are

- Color
- Contact
- DateTime
- Dimmer
- Group
- Number
- RollerShutter
- String
- Switch

The ItemLabel has the ability to be formatted using standard Java formatter syntax, which will not be covered here other than to say `[%.1f °C]` will display the temperature to 1 decimal point, ie 23.4.

The Binding is where we configure which device we actually are querying. In this example I’m using ZWave Node 3. We specify the command class that needs be be used, for instance SENSOR_MULTILEVEL and then the sensor type, this is documented at [https://github.com/openhab/openhab/wiki/Z-Wave-Binding](https://github.com/openhab/openhab/wiki/Z-Wave-Binding)

When this file is saved, you should now be able to see the items in HABMin under Configuration -> Items and Groups

The next devices we will configure are the NorthQ meters. These can be placed in any filename in the items directory. Lets create them as `power.items`

```
Group Power <energy>
Number power_1_battery “Electricity Meter Battery [%s %%]” <battery> (Power) {zwave=”4:command=BATTERY,refresh_interval=3600″}
Number power_1_usage_total “KWH [%s]” (Power) {zwave=”4:command=METER,meter_scale=E_KWh,refresh_interval=450″}
Number power_1_usage “Watt [%.2f]” (Power)
```

Lets put the gas in `gas.items`

```
Group Gas
Number gas_1_battery “Gas Meter Battery [%s %%]” (Gas) {zwave=”6:command=BATTERY,refresh_interval=3600″}
Number gas_1_usage_total “m3 [%s]” (Gas) {zwave=”6:command=METER,refresh_interval=450″}
Number gas_1_usage “m3 [%.2f]” (Gas)
```

You will notice that we have a `power_1_usage` and `gas_1_usage` item that does not have a binding. We’ll look at this in a few moments

Our final set of items that we want to create are for the Qubino Relay. Lets create this in `relay.items`

```
Switch light1_state “light1 [%s]” {zwave=”10:command=switch_binary”}
Number light1_power “light1 Watt [%s]” {zwave=”10:command=meter,refresh_interval=60″}
```

# Rules

Rules allow you to create logic within the OpenHAB system, for instance when there is movement, turn on the light.

We’re going to create two rules, one for our gas meter and one for our power meter.

Both of these devices will, by default, only return the amount of power or gas used since they were installed, but I find it more interesting / useful to have a “point in time” amount that is being used.

Create the file `/opt/openhab/configuration/rules/northq.rules`

```
rule gas1_current_usage_update
 when
 // When power meter is updated
 Item gas_1_usage_total received update
 then
 // Update current usage with the difference between this, and the previous update to get our spot usage of m3.
   gas_1_usage.postUpdate(
     gas_1_usage_total.deltaSince(now.minusMinutes(5)).value
   )
 );
 end

rule power1_current_usage_update
 when
 // When power meter is updated
 Item power_1_usage_total received update
 then
 // Update current usage with the difference between this, and the previous update and multiply by 1000 to give us the total in watts.
   power_1_usage.postUpdate(
     power_1_usage_total.deltaSince(now.minusMinutes(5)).value * 1000
   )
 );
 end
```
These rules simply take the last value that the usage total was 5 minutes ago, and sets the usage to the difference between the two readings

# Sitemaps

Sitemaps provide us a way to display items on a device, such as within OpenHAB’s applications or on the web page.

They can get quite complex and as such I’m not going to cover them off in details, but a sample sitemap would be `/opt/openhab/configuration/sitemap/default.xml`
```
sitemap default label=”Home”
{
 Frame label=”Hallway” {
   Switch item=light1_state
   Text item=light1_power
 }

 Frame label=”Sensor1″ {
   Text item=sensor_1_temp valuecolor=[&gt;25=”orange”,&gt;15=”green”,&gt;5=”orange”,&lt;=5=”blue”] {
   Text item=sensor_1_humidity
   Text item=sensor_1_luminance
   Text item=sensor_1_battery
   Text item=sensor_1_motion
 }

 Frame label=”Energy” {
   Text item=power_1_usage label=”Power usage [%.0f Watts]” icon=”energy”
   Chart item=power_1_usage period=h refresh=6000
   Text item=power_1_battery
   Text item=gas_1_usage label=”Gas usage [%.2f m3]” icon=”fire-on”
   Chart item=gas_1_usage period=h refresh=6000
   Text item=gas_1_battery
 }
}
```