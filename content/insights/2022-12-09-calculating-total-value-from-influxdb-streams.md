---
title: "Calculating total value from InfluxDB streams"
date: 2022-12-09
draft: false
description: "Learn how to calculate total values from InfluxDB streams using Flux queries"
tags: ["influxdb", "flux", "data-analysis", "iot"]
categories: ["Data Engineering"]
featuredImage: "/images/insights/influxdb/grafana-dashboard.png"
---

The energy data in my home is collected using Shelly devices. For the total power consumed in the house, I'm using the Shelly 3EM 3-Phase Energy Meter. Data points are pushed to a Mosquitto MQTT server and imported into InfluxDB using Telegraf. Finally, the data is displayed in a Grafana dashboard. The entire setup will be the topic of another post (someday).

<div class="grid grid-cols-2 gap-4 my-8">
  <div class="space-y-4">
    <figure>
      <img src="/images/insights/influxdb/shelly-3em.png" alt="Shelly 3EM" class="w-full rounded-lg shadow-lg">
      <figcaption class="text-sm text-gray-600 mt-2">Shelly 3EM 3-Phase Energy Meter</figcaption>
    </figure>
    <figure>
      <img src="/images/insights/influxdb/grafana-dashboard.png" alt="Total power displayed in Grafana dashboard" class="w-full rounded-lg shadow-lg">
      <figcaption class="text-sm text-gray-600 mt-2">Power consumption displayed in Grafana dashboard</figcaption>
    </figure>
  </div>
  <div>
    <figure>
      <img src="/images/insights/influxdb/mosquitto-management.png" alt="Mosquitto Management Center" class="w-full h-full object-cover rounded-lg shadow-lg">
      <figcaption class="text-sm text-gray-600 mt-2">Mosquitto MQTT broker management interface</figcaption>
    </figure>
  </div>
</div>

Because the Shelly devices provide the current power data only separately for the three electric current phases, I wanted and needed to calculate the total power consumption from these values. It took some time to understand the necessary basic flow of data InfluxDB and it's query language Flux, but then it is actually very easy to do all kinds of calculations on streaming data.

The steps for writing a Flux query are the same for the majority of queries:

* Source – define the source of the data
* Filter – filter for relevant data as needed
* Shape – group the data for processing
* Process – run calculations on the shaped data

```flux
from(bucket: "example-bucket")              // ── Source
    |> range(start: -1d)                    // ── Filter on time
    |> filter(fn: (r) => r._field == "foo") // ── Filter on column values
    |> group(columns: ["sensorID"])         // ── Shape
    |> mean()                               // ── Process
```

The goal was displaying the power consumption data of the main energy meter in our home.

* Source: The bucket with the latest data from the devices.
* Filter: Data tagged with the tag "device_name" = "House-Power" and the measurement "power" within the given time range
* Shape: Group the data streams by phase, resulting in three "tables", one per phase
* Process: Calculate the average power within each interval to display.

The calculated mean value of each interval results in _one single value / per phase / per interval_. The key to calculating the sum of the three phases is simply having just one mean value per interval with common timestamp between the three phases.

```flux
from(bucket: "energy-live")
  |> range(start: -1h)
  |> filter(fn: (r) => r["device_name"] == "House-Power")
  |> filter(fn: (r) => r["_measurement"] == "power")
  |> drop(columns: ["host", "topic"]) // drop some unneeded columns
  |> aggregateWindow(every: 20s, fn: mean, createEmpty: false)
```

![Sample data with three virtual tables](/images/insights/influxdb/by-phases.png)

After that, it is clear that the total value will simply be the sum of the three phases for each timestamp.

```flux
from(bucket: "energy-live")
  |> range(start: 2022-12-08T15:05:00Z, stop: 2022-12-08T15:06:20Z)
  |> filter(fn: (r) => r["device_name"] == "House-Power" and r["_measurement"] == "power")
  |> group(columns: ["phase"]) // group the data by phases
  // Calculate mean value of all data points in small time windows
  |> aggregateWindow(every: 20s, fn: mean, createEmpty: true)
  |> yield(name: "phases") // yields "By phases" table (with three groups by phase)
  |> group(columns: ["_time"])
  // Calculate sum of each time window (all values with same timestamp)
  |> sum()
  |> yield(name: "total") // yields "Total" table
```

![Resulting output](/images/insights/influxdb/sample-data.png)

More information about data aggregation can be found in the InfluxDB documentation: [InfluxDB Documentation](https://docs.influxdata.com/influxdb/cloud/query-data/flux/window-aggregate/)
