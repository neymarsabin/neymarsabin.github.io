+++
layout = 'post'
title = 'batterarch'
date = 2024-08-18T12:00:00+05:45
draft = false
+++

<!--more-->
Check battery performance of your laptop if you are running Arch Linux. This binary is only tested in Arch Linux.

#### how it works? 
The applications reads the file /sys/class/power_supply/BAT0/uevent and saves the data into a SQlite db table. The file looks something like this:

    cat /sys/class/power_supply/BAT0/uevent

    # POWER_SUPPLY_NAME=BAT0
    # POWER_SUPPLY_TYPE=Battery
    # POWER_SUPPLY_STATUS=Discharging
    # POWER_SUPPLY_PRESENT=1
    # POWER_SUPPLY_TECHNOLOGY=Li-poly
    # POWER_SUPPLY_CYCLE_COUNT=14
    # POWER_SUPPLY_VOLTAGE_MIN_DESIGN=11520000
    # POWER_SUPPLY_VOLTAGE_NOW=11178000
    # POWER_SUPPLY_POWER_NOW=8014000
    # POWER_SUPPLY_ENERGY_FULL_DESIGN=57000000
    # POWER_SUPPLY_ENERGY_FULL=58030000
    # POWER_SUPPLY_ENERGY_NOW=17210000
    # POWER_SUPPLY_CAPACITY=29
    # POWER_SUPPLY_CAPACITY_LEVEL=Normal
    # POWER_SUPPLY_MODEL_NAME=LNV-5B11C73244
    # POWER_SUPPLY_MANUFACTURER=SMP
    # POWER_SUPPLY_SERIAL_NUMBER=772

#### building
Make sure you have golang installed. To build run the following command:

    go build .

This will create a binary called batterarch that we can run in the terminal.

#### usage
Now that you have the binary ready, there are multiple ways you can run the app. The binary comes up the following options:

    ./batterarch <options>

    options can be one of the following:
    g => graph => view usage graph in the browser.
    j => json => view usage data as JSON in the stdout/terminal.
    s => server => start a TCP server in port 42069 that has routes

#### server test
When you spin up a batterarch server, the API responds to JSON in http://localhost:42069/

    curl -X GET http://localhost:42069/ | jq ".data[] | .BatteryLevel"

#### data storage?
This uses sqlite database as data storage. All the data that we get in the graph, json or server mode are retrieved using SQL queries from the SQlite database. The location of the database is $HOME/.config/batterarch/batterarch.db The config folder and database is automatically created. If you want to write your own queries against the database, you can explore the batterarch.db file and write SQL queries.
