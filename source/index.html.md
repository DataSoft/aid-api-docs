---
title: Automatic Injury Detection API Reference

language_tabs:
  - java

toc_footers:
  - <a href='https://github.com/tripit/slate'>Documentation Powered by Slate</a>

search: true
---

# Introduction

This document describes the API for accessing the Automatic Injury Detection devices from third party Android applications.  Requires version 1.6.1 or later of the AID [application](https://play.google.com/store/apps/details?id=datasoft.com.aid).

There is also a bare bones example API test [app](https://github.com/DataSoft/AIDAPItest).

# Intents

```java
Intent intent = new Intent();
intent.setClassName("datasoft.com.aid", "com.datasoft.aid.ConnectionService");
```
> Note that the package name for AID apk is `datasoft.com.aid` and not `com.datasoft.aid`.

Communications to the AID ConnectionService is done by `startService` with an explicit intent.  Use of implicit intents with intent filters is no longer available in Android 5.0+.  Use `Intent.setClassName` with a package name of `datasoft.com.aid` and a class name of `com.datasoft.aid.ConnectionService` for all commands going to the AID ConnectionService.  The service returns all responses as Broadcast Intents.

### Commands
Commands supported so far by the service:

* com.datasoft.aid.action.GET_STATUS
* com.datasoft.aid.action.START_SCAN
* com.datasoft.aid.action.STOP_SCAN

### Events
Events reported by the service (broadcast intents):

* com.datasoft.aid.event.UPDATE_STATUS
* com.datasoft.aid.event.DEVICE_FOUND
* com.datasoft.aid.event.DEVICE_CONNECTED
* com.datasoft.aid.event.DEVICE_DISCONNECTED
* com.datasoft.aid.event.INJURY

# Scanning for Devices

## Start Scanning

Use the com.datasoft.aid.action.START_SCAN action to scan for devices.  This will disconnect any currently connected devices while scanning.  There is currently not terribly useful because there isn't a way to select the AID devices using the API.

```java
Intent intent = new Intent("com.datasoft.aid.action.START_SCAN");
intent.setClassName("datasoft.com.aid", "com.datasoft.aid.ConnectionService");
startService(intent);
```

## Device Found

While scanning, devices are returned with a BroadcastIntent with action com.datasoft.aid.event.DEVICE_FOUND.  The service reports every device advertisement seen, it's up to the client to keep track of duplicates.  Device name and address are included in the Intent's extras bundle.

### DEVICE_FOUND parameters

Parameter | Type | Description
--------- | ---- | -----------
device-address | String | BLE address of the AID device
device-name | String | Advertised name of the AID device

```java
BroadcastReceiver deviceFoundReceiver = new BroadcastReceiver() {
    @Override
    public void onReceive(Context context, Intent intent) {
        String address = intent.getStringExtra("device-address");
        String name = intent.getStringExtra("device-name");
        // Do stuff..
    }
};
IntentFilter deviceFilter = new IntentFilter("com.datasoft.aid.event.DEVICE_FOUND");
registerReceiver(deviceFoundReceiver, deviceFilter);
```

## Stop Scanning

To stop scanning for devices, use a com.datasoft.aid.action.STOP_SCAN action.

```java
Intent intent = new Intent("com.datasoft.aid.action.STOP_SCAN");
intent.setClassName("datasoft.com.aid", "com.datasoft.aid.ConnectionService");
startService(intent);
```

# Device Status

To get the status of AID devices, send a com.datasoft.aid.action.GET_STATUS action.

```java
Intent intent = new Intent("com.datasoft.aid.action.GET_STATUS");
intent.setClassName("datasoft.com.aid", "com.datasoft.aid.ConnectionService");
startService(intent);
```

## Status results

The service sends a com.datasoft.aid.event.UPDATE_STATUS BroadcastIntent in response.  Device status fields are in the Intent's extras bundle.

### UPDATE_STATUS parameters

Parameter | Type | Description
--------- | ---- | -----------
front-device-name | String | Advertised name of AID device attached to front of the vest
front-device-address | String | BLE address of front device
front-device-connected | boolean | True if phone is currently connected to this device
front-device-battery | int | Battery level of device in percent
front-device-injury | int | Injury status: 1=Upper panel, 2=Lower panel, 3=Both
back-device-name | String | Advertised name of AID device attached to back of the vest
back-device-address | String | BLE address of back device
back-device-connected | boolean | True if phone is currently connected to this device
back-device-battery | int | Battery level of device in percent
back-device-injury | int | Injury status: 1=Upper panel, 2=Lower panel, 3=Both

```java
BroadcastReceiver statusReceiver = new BroadcastReceiver() {
    @Override
    public void onReceive(Context context, Intent intent) {
        String frontName = intent.getStringExtra("front-device-name");
        String frontAddress = intent.getStringExtra("front-device-address");
        boolean frontConnected = intent.getBooleanExtra("front-device-connected", false);
        int frontBattery = intent.getIntExtra("front-device-battery", -1);
        int frontInjury = intent.getIntExtra("front-device-injury", 0);

        String backName = intent.getStringExtra("back-device-name");
        String backAddress = intent.getStringExtra("back-device-address");
        boolean backConnected = intent.getBooleanExtra("back-device-connected", false);
        int backBattery = intent.getIntExtra("back-device-battery", -1);
        int backInjury = intent.getIntExtra("back-device-injury", 0);

        // Do stuff..
    }
};
IntentFilter statusFilter = new IntentFilter("com.datasoft.aid.event.UPDATE_STATUS");
registerReceiver(statusReceiver, statusFilter);
```

# Other Device Events

These events are broadcast based on device events and not as a response to an action.

## Device Disconnected

The com.datasoft.aid.event.DEVICE_DISCONNECTED event notifies the client that the AID devices are no longer connected to the service.  For example, when the phone goes out of range of the devices.  No extra data is included in this intent.

```java
BroadcastReceiver disconnectedReceiver = new BroadcastReceiver() {
    @Override
    public void onReceive(Context context, Intent intent) {
        Log.d(TAG, "AID Devices have been disconnected");
    }
};
IntentFilter disconnectedFilter = new IntentFilter("com.datasoft.aid.event.DEVICE_DISCONNECTED");
registerReceiver(disconnectedReceiver, disconnectedFilter);
```

## Device Connected

The com.datasoft.aid.event.DEVICE_CONNECTED event notifies the client that the AID devices are both connected to the service.  No extra data is included in this intent.

```java
BroadcastReceiver connectedReceiver = new BroadcastReceiver() {
    @Override
    public void onReceive(Context context, Intent intent) {
        Log.d(TAG, "AID Devices have been connected");
    }
};
IntentFilter connectedFilter = new IntentFilter("com.datasoft.aid.event.DEVICE_CONNECTED");
registerReceiver(connectedReceiver, connectedFilter);
```

## Injury Detected

The com.datasoft.aid.event.INJURY event notifies the client that one of the AID devices has detected a piercing event.  The injury location is included in the intent's extras bundle.

### INJURY parameters

Parameter | Type | Description
--------- | ---- | -----------
front-upper | boolean | True if the upper part of the front panel has been pierced.
front-lower | boolean | True if the lower part of the front panel has been pierced.
back-upper | boolean | True if the upper part of the back panel has been pierced.
back-lower | boolean | True if the lower part of the back panel has been pierced.

```java
BroadcastReceiver injuryReceiver = new BroadcastReceiver() {
    @Override
    public void onReceive(Context context, Intent intent) {
        boolean frontUpper = intent.getBooleanExtra("front-upper", false);
        boolean frontLower = intent.getBooleanExtra("front-lower", false);
        boolean backUpper = intent.getBooleanExtra("back-upper", false);
        boolean backLower = intent.getBooleanExtra("back-lower", false);

        // Do stuff
    }
};
IntentFilter injuryFilter = new IntentFilter("com.datasoft.aid.event.INJURY");
registerReceiver(injuryReceiver, injuryFilter);
```

