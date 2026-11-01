# Fabric Real-Time Bike Analytics

End-to-end real-time analytics solution built with Microsoft Fabric Trial using the London Bike Share sample stream.

The project demonstrates real-time ingestion, stream processing, KQL analytics, operational monitoring, visualization, alerting, and GitHub source control.

## Solution Overview

The solution continuously processes London Bike Share events and supports two independent data paths:

1. **Raw data path**
   - Preserves incoming bike-share events without transformation.
   - Stores the events in the `BikeShareRaw` KQL table.

2. **Minute aggregation path**
   - Groups events by `Neighbourhood`.
   - Calculates `SUM(No_Bikes)` in one-minute tumbling windows.
   - Stores the results in the `BikeAvailabilityMinute` KQL table.

## Architecture

```text
London Bike Share Sample Stream
                |
                v
      BikeAnalyticsEventstream
                |
        +-------+------------------------------+
        |                                      |
        v                                      v
BikeAnalyticsDestination       GroupBikeAvailabilityByNeighbourhood
        |                                      |
        v                                      v
  BikeShareRaw                     BikeAvailabilityDestination
                                               |
                                               v
                                  BikeAvailabilityMinute
```

## Microsoft Fabric Components

- **Workspace:** `Fabric Real-Time Bike Analytics`
- **Eventhouse:** `BikeAnalyticsEventhouse`
- **KQL Database:** `BikeAnalyticsKQL`
- **Eventstream:** `BikeAnalyticsEventstream`
- **Raw source:** `BikeShareSource`
- **Raw destination:** `BikeAnalyticsDestination`
- **Stream transformation:** `GroupBikeAvailabilityByNeighbourhood`
- **Aggregated destination:** `BikeAvailabilityDestination`
- **Raw KQL table:** `BikeShareRaw`
- **Aggregated KQL table:** `BikeAvailabilityMinute`
- **KQL Queryset:** `BikeAnalyticsKQL_queryset`
- **Real-Time Dashboard:** `BikeAnalyticsDashboard`
- **Activator:** `BikeAvailabilityActivator`

## Streaming Transformation

The Eventstream applies the following transformation:

- Aggregation: `SUM(No_Bikes)`
- Output field: `TotalBikes`
- Grouping field: `Neighbourhood`
- Window type: `Tumbling`
- Window duration: `1 minute`
- Window offset: `0 seconds`

The resulting table contains:

- `Neighbourhood`
- `TotalBikes`
- `Window_End_Time`

## Data Quality Validation

The raw stream was validated for:

- missing bike-point identifiers;
- missing street names;
- missing neighbourhood names;
- invalid latitude values;
- invalid longitude values;
- negative bike counts;
- negative empty-dock counts.

The minute-level aggregation was validated for:

- total record count;
- distinct neighbourhood count;
- distinct time-window count;
- missing neighbourhood names;
- negative aggregated bike counts.

The validation returned:

- 19 distinct neighbourhoods;
- no missing neighbourhood values;
- no negative bike-count values.

## KQL Examples

### Raw data validation

```kusto
BikeShareRaw
| summarize
    TotalRows = count(),
    NullBikepointID = countif(isempty(BikepointID)),
    NullStreet = countif(isempty(Street)),
    NullNeighbourhood = countif(isempty(Neighbourhood)),
    InvalidLatitude = countif(Latitude < -90 or Latitude > 90),
    InvalidLongitude = countif(Longitude < -180 or Longitude > 180),
    NegativeBikeCount = countif(No_Bikes < 0),
    NegativeEmptyDockCount = countif(No_Empty_Docks < 0)
```

### Minute aggregation validation

```kusto
BikeAvailabilityMinute
| summarize
    TotalRows = count(),
    Neighbourhoods = dcount(Neighbourhood),
    TimeWindows = dcount(Window_End_Time),
    NullNeighbourhood = countif(isempty(Neighbourhood)),
    InvalidBikeCount = countif(TotalBikes < 0)
```

### Total bike availability trend

```kusto
BikeAvailabilityMinute
| summarize TotalAvailableBikes = sum(TotalBikes) by Window_End_Time
| order by Window_End_Time asc
| render timechart
```

## Real-Time Dashboard

`BikeAnalyticsDashboard` provides operational monitoring through five visuals:

- bike availability by neighbourhood;
- top 10 neighbourhoods by available bikes;
- total available bikes;
- total available empty docks;
- total bike availability trend.

The visuals refresh from data stored in the KQL database.

## Activator Alert

`BikeAvailabilityActivator` contains the rule:

```text
Low Bike Availability Alert
```

Configuration:

- Grouping field: `Neighbourhood`
- Monitored value: `TotalBikes`
- Condition: `TotalBikes < 50`
- Evaluation frequency: every 5 minutes
- Action: Microsoft Teams notification
- Context: `Neighbourhood`, `TotalBikes`

The alert was verified using a test action delivered to Microsoft Teams.

## Real-Time Hub

Real-Time Hub exposes and previews:

- `BikeAnalyticsEventstream-stream`
- `BikeShareRaw`

It provides a central location for discovering and inspecting the streaming assets used by the solution.

## Source Control

The Microsoft Fabric workspace is connected to this GitHub repository through native Git integration.

Fabric-managed artifacts include:

- Real-Time Dashboard;
- Eventhouse and KQL databases;
- Eventstream;
- Activator.

The workspace is synchronized with the `main` branch.

## Trial and Platform Considerations

The project was built entirely in Microsoft Fabric Trial using built-in sample data.

A classic Power BI workspace app was not created because the primary visualization is a Real-Time Dashboard, which is shared directly rather than included as content in a classic workspace app.

A separate Power BI report was intentionally not added because it would duplicate the operational analytics already provided by the Real-Time Dashboard.

## Skills Demonstrated

- Microsoft Fabric Real-Time Intelligence
- Real-Time Hub
- Eventstream ingestion
- No-code stream processing
- Eventhouse
- KQL Database
- KQL Queryset
- Kusto Query Language
- Real-Time Dashboard development
- Activator alerts
- Microsoft Teams notifications
- Data-quality validation
- Operational monitoring
- GitHub source control

## Repository Structure

```text
BikeAnalyticsDashboard.KQLDashboard/
BikeAnalyticsEventhouse.Eventhouse/
BikeAnalyticsEventstream.Eventstream/
BikeAvailabilityActivator.Reflex/
README.md
```

## Project Status

The real-time ingestion, transformation, storage, validation, dashboard, alerting, and source-control components are operational.

## Author

Kamil Soszka
