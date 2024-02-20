---
sidebar_position: 10
---

# Thermal System

## Temperature Sensor

There is a temperature sensor on the X3 chip, which directly reflects the temperature of the X3 chip DIE.

Under /sys/class/hwmon/, there is a hwmon0 directory containing relevant parameters of the temperature sensor.

Important files: name and temp1_input.

-   name refers to the name of the temperature sensor.
-   temp1_input refers to the temperature value, with a default precision of 0.001 degrees Celsius.

```
# cat /sys/class/hwmon/hwmon0/name
pvt_ts
# cat /sys/class/hwmon/hwmon0/temp1_input
55892
# 55892 represents 55.892 degrees Celsius
```

The temperature of this hwmon0 device directly affects the device at cat /sys/class/thermal/thermal_zone0/temp, and the values of the two are identical.

## Thermal

Linux Thermal is a module related to temperature control in the Linux system. It is mainly used to control the heat generated by chips during system operation, ensuring that the chip temperature and device casing temperature are maintained within a safe and comfortable range.

In order to achieve reasonable control of device temperature, we need to understand the following three modules:

- Device for obtaining temperature: in the Thermal framework, it is abstracted as the Thermal Zone Device, which is the temperature sensor thermal_zone0.
- Device that needs cooling: in the Thermal framework, it is abstracted as the Thermal Cooling Device, including CPU and GPU.
- Temperature control strategy: in the Thermal framework, it is abstracted as the Thermal Governor.

Information and controls for the above modules can be obtained in the /sys/class/thermal directory.

There are a total of three cooling devices in the x3:

- cooling_device0: cnn0
- cooling_device1: cnn1
- cooling_device2: cpu

Currently, the default strategy can be determined by the following command, which uses step_wise.

```
cat /sys/class/thermal/thermal_zone0/policy
```The supported policies can be viewed using the following command: user_space and step_wise, a total of two.

```
cat /sys/class/thermal/thermal_zone0/available_policies
```

- user_space is a strategy that reports the current temperature of the thermal zone, as well as temperature control triggering points and other information to user space through uevent. The temperature control strategy is determined by user space software.
- step_wise is a relatively mild temperature control strategy that gradually increases the cooling state in each polling cycle.

The specific choice of strategy depends on the needs of the product. It can be specified during compilation or dynamically switched through sysfs.

For example, to dynamically switch to the user_space mode:

```
echo user_space > /sys/class/thermal/thermal_zone0/policy 
```

Executing the following command will show three trip points (trigger temperatures).

```
ls -l  /sys/devices/virtual/thermal/thermal_zone0
```

Currently, the default trip point selected is trip_point_1_temp (temperature is 75 degrees).

```
trip_point_*_hyst (*:0 - 2) # hysteresis temperature
trip_point_*_temp (*: 0 - 2) # trigger temperature
trip_point_*_type (*: 0 - 2) # trigger point type
```

If you want to delay frequency reduction until the temperature reaches 85 degrees Celsius:

```
echo 85000 > /sys/devices/virtual/thermal/thermal_zone0/trip_point_1_temp
```

If you want to adjust the shutdown temperature to 105 degrees Celsius:

```
echo 105000 > /sys/devices/virtual/thermal/thermal_zone0/trip_point_2_temp
```

Note: The above settings need to be set again after power-off and restart.

## Thermal Reference Documentation

kernel/Documentation/thermal/