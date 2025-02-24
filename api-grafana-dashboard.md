# **API Logs Analysis Summary using Grafana Loki**  

This summary provides a detailed analysis of API logs collected from **Grafana Loki**, covering key insights such as request trends, error rates, response times, and top endpoints.  

---

## **1. Overview of API Logs**
- **Total API Requests in Last 24 Hours**: `XX,XXX`
- **Successful Requests (2xx Status)**: `XX,XXX` (`YY%`)
- **Client Errors (4xx Status)**: `XXX` (`YY%`)
- **Server Errors (5xx Status)**: `XXX` (`YY%`)
- **Peak Traffic Time**: `HH:MM - HH:MM UTC`
- **Total Unique Endpoints Accessed**: `XX`

---

## **2. Error Analysis**
- **Total Errors Logged**: `XXX`
- **Top Error Messages**:
  - `"Timeout connecting to database"` - `XX` times
  - `"Unauthorized access"` - `XX` times
  - `"Internal server error"` - `XX` times
- **Highest Error Rate Period**: `HH:MM - HH:MM UTC`
- **Most Common Error Type**: **(e.g., 500 Internal Server Error, 403 Forbidden, etc.)**
- **Affected Endpoints with High Error Rate**:
  - `/api/orders` - `XX%` error rate
  - `/api/login` - `XX%` error rate

---

## **3. Response Time & Performance Metrics**
- **Average Response Time**: `XXX ms`
- **95th Percentile Response Time**: `XXX ms`
- **Fastest Endpoint**: `/api/status` - `XX ms`
- **Slowest Endpoint**: `/api/reports` - `XX ms`
- **Latency Spikes Detected?**: **Yes/No**  
  - Peak latency recorded at `HH:MM UTC`  
  - Possible causes: **(e.g., high traffic, database slow queries, etc.)**  

---

## **4. API Usage Patterns**
- **Most Frequently Called Endpoints**:
  - `/api/products` - `XX,XXX` requests
  - `/api/orders` - `XX,XXX` requests
  - `/api/customers` - `XX,XXX` requests
- **Peak Usage Hours**: `HH:MM - HH:MM UTC`
- **Unusual Activity Detected?**: **Yes/No**  
  - **Potential issues**: (e.g., repeated failed logins, high traffic spikes)

---

## **5. Security & Authentication Insights**
- **Total Failed Login Attempts**: `XX`
- **Unauthorized API Access Attempts**: `XX`

---

Here is a **sample Grafana dashboard JSON template** for **API Logs Analysis** using **Grafana Loki**. This template covers key metrics such as request trends, error rates, response times, top endpoints, security insights, and recommendations.

### **Steps to Import the Dashboard**  
1. Open **Grafana**.
2. Go to **Dashboards** → **New** → **Import**.
3. Copy and paste the JSON below.
4. Click **Load** and select **Loki** as the data source.
5. Click **Import**.

---

### **📜 Sample JSON Template for API Logs Analysis Dashboard**
```json
{
  "dashboard": {
    "id": null,
    "uid": "api-logs-dashboard",
    "title": "API Logs Analysis",
    "tags": ["Loki", "API Logs", "Monitoring"],
    "timezone": "browser",
    "panels": [
      {
        "title": "Total API Requests",
        "type": "stat",
        "datasource": "Loki",
        "targets": [
          {
            "expr": "count_over_time({app=\"api-service\"}[24h])",
            "refId": "A"
          }
        ]
      },
      {
        "title": "Error Rate (Last 1 Hour)",
        "type": "gauge",
        "datasource": "Loki",
        "targets": [
          {
            "expr": "count_over_time({app=\"api-service\"} |= \"ERROR\" [1h]) / count_over_time({app=\"api-service\"}[1h])",
            "refId": "B"
          }
        ]
      },
      {
        "title": "Top API Endpoints by Request Count",
        "type": "bar",
        "datasource": "Loki",
        "targets": [
          {
            "expr": "topk(5, count_over_time({app=\"api-service\"} | json | line_format \"{{.endpoint}}\" [1h]))",
            "refId": "C"
          }
        ]
      },
      {
        "title": "API Response Time (95th Percentile)",
        "type": "timeseries",
        "datasource": "Loki",
        "targets": [
          {
            "expr": "quantile_over_time(0.95, {app=\"api-service\"} | json | line_format \"{{.response_time}}\" [1h])",
            "refId": "D"
          }
        ]
      },
      {
        "title": "Most Common Error Messages",
        "type": "table",
        "datasource": "Loki",
        "targets": [
          {
            "expr": "topk(5, count_over_time({app=\"api-service\"} |= \"ERROR\" | json | line_format \"{{.message}}\" [1h]))",
            "refId": "E"
          }
        ]
      },
      {
        "title": "Peak Traffic Time (Last 24h)",
        "type": "heatmap",
        "datasource": "Loki",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, sum(rate({app=\"api-service\"}[24h])) by (time))",
            "refId": "F"
          }
        ]
      },
      {
        "title": "Unauthorized Access Attempts",
        "type": "stat",
        "datasource": "Loki",
        "targets": [
          {
            "expr": "count_over_time({app=\"api-service\"} |= \"Unauthorized\" [1h])",
            "refId": "G"
          }
        ]
      },
      {
        "title": "Suspicious IP Addresses",
        "type": "table",
        "datasource": "Loki",
        "targets": [
          {
            "expr": "topk(5, count_over_time({app=\"api-service\"} |= \"Failed login\" | json | line_format \"{{.ip}}\" [1h]))",
            "refId": "H"
          }
        ]
      }
    ],
    "schemaVersion": 30,
    "version": 1,
    "refresh": "5m"
  }
}
```

---

### **📊 Key Features of This Dashboard**
✅ **Total API Requests** – Shows the total number of API calls in the last 24 hours.  
✅ **Error Rate** – Displays errors as a percentage of total API requests.  
✅ **Top API Endpoints** – Identifies the most frequently accessed API routes.  
✅ **Response Time Analysis** – Monitors API latency, focusing on the 95th percentile.  
✅ **Common Error Messages** – Lists the top recurring errors from logs.  
✅ **Peak Traffic Analysis** – Highlights periods of highest API usage.  
✅ **Unauthorized Access Tracking** – Counts unauthorized attempts to access APIs.  

---
