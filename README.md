# How to use Blazm.Bluetooth

Blazm.Bluetooth makes it easy to connect Blazor to your Bluetooth devices using Web Bluetooth.

Works both Client-side and Server-side.

## Getting Started

1. Add Nuget package Blazm.Bluetooth
2. In Program.cs add ```builder.Services.AddBlazmBluetooth();```
3. In your wwwrooot/index.html add
```<script src="_content/Blazm.Bluetooth/JSInterop.js"></script>```
4. In the component you want to connect to a device add the Blazm.Bluetooth Namespace
```@using Blazm.Bluetooth```
5. Inject the BluetoothNavigator (the instance that will communicate with your device)
```@inject BluetoothNavigator navigator```

Now you are all setup now it is time to connect to a device.

## Connecting to a device

You need to create a query or a filter to filter devices, you could also specify ```AcceptAllDevices``` but that is only recommended while testing.
To take a look at availible devices you can use (for Microsoft Edge) edge://bluetooth-internals.
You can query devices using a ServiceId or by name, you also have to specify all the services that you want to access in ```OptionalServices```

To connect to an Andersson (SenSun) scale you need to do the following (check Pages/AnderssonScaleDemo for mor info)
Specify the ServiceId and CharacteristicId you want to communicate with.
``` cs
var serviceid = "0000ffb0-0000-1000-8000-00805f9b34fb";
var characteristicId = "0000ffb2-0000-1000-8000-00805f9b34fb";
```

Create a filter
``` cs
var q = new RequestDeviceQuery();
q.Filters.Add(new Filter() { Services = { serviceid } });
```

Request a device
``` cs
var device = await navigator.RequestDeviceAsync(q);
```
This will return a device and it contains an id that you can use to read, write, or setu�p notifications.
Call the ```SetupNotifyAsync``` to get notifications when the value changes.
``` cs
await navigator.SetupNotifyAsync(device.Id, serviceid, characteristics);
navigator.Notification += Value_Notification;
``` 
and add an event listener that parses the data from the notification.
``` cs
private void Value_Notification(object sender, CharacteristicEventArgs e)
{
    var data = e.Value.ToArray();
    weight = (256 * data[4]) + data[5];
    if (data[7] == 1)
    {
        weight *= -1;
    }

    StateHasChanged();
}
```

Same thing goes for read and write
``` cs
//Write
await navigator.WriteValueAsync(device.Id, serviceId, characteristicId, bytearray);

//Read
var value = await navigator.ReadValueAsync(device.Id, serviceId, characteristicId);
```

There is still many scenarios to implement but this should cover the basics.

Please feel free to add any idaes/problems/needs you might have or make a PR.
