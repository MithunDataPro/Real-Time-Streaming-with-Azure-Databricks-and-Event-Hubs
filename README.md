# End-to-End Real-Time Streaming Project

## Project Overview

This project demonstrates an end-to-end real-time data streaming solution using Azure Event Hubs, Databricks (with Spark Structured Streaming), Delta Lake, and Power BI for visualization. The project follows the Medallion architecture to process data through Bronze, Silver, and Gold layers.

### Architecture

![Solution Overview](image_link_here)

The architecture is divided into three main stages:

1. **Ingest:** Streaming data from various sources such as Kafka, IoT devices, and log data is ingested into Azure Event Hubs.
2. **Process:** The data is processed using Databricks with Spark Structured Streaming. The data is stored in Delta Lake across Bronze, Silver, and Gold layers.
3. **Serve:** The processed data in the Gold layer is made available for reporting in Power BI.

### Medallion Architecture

- **Bronze Layer:**
  - Stores raw, unprocessed data.
  - Minimal transformations are applied to preserve the original data.

- **Silver Layer:**
  - Data is cleaned, filtered, and enriched.
  - The data is in a more consumable format, ready for business use.

- **Gold Layer:**
  - Aggregated and refined data is stored.
  - This layer is used for reporting and business intelligence in Power BI.

## Azure Event Hubs

### Overview

Azure Event Hubs is a big data streaming platform and event ingestion service capable of receiving and processing millions of events per second. Event Hubs can process and store events, data, or telemetry produced by distributed software and devices. It can capture streaming data into an Azure Blob storage or Azure Data Lake for long-term retention or micro-batch processing.

### Features

- **Real-time Data Ingestion:** Event Hubs allows for the ingestion of data in real time with high throughput.
- **Partitioned Consumer Model:** This model enables parallel processing of data, improving scalability and performance.
- **Event Retention:** You can configure the retention period for events to ensure that the data is available for the required duration.
- **Capture:** Automatically capture streaming data into Azure Blob storage or Azure Data Lake Store.

### Test Data Generation

In this project, we're generating fake weather data in JSON format. The data includes attributes such as temperature, humidity, wind speed, wind direction, and precipitation. Below is an example of the JSON format used:

```json
{
  "temperature": 20,
  "humidity": 60,
  "windSpeed": 10,
  "windDirection": "NW",
  "precipitation": 0,
  "conditions": "Partly Cloudy"
}
```
