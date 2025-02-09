# **Structured Logging Best Practices**

## **Introduction**
Structured logging is the practice of storing logs in a format that is consistent, machine-readable, and optimized for querying and analysis. Unlike traditional plain-text logs, structured logs store information in a structured format, typically JSON, which makes it easier to parse, filter, and analyze logs, especially in distributed systems. This guide outlines structured logging best practices, including good and bad examples, as well as the details each log entry should contain for each log type.

---

## **1. What is Structured Logging?**

Structured logging involves logging information in a well-defined structure, typically using JSON, rather than just plain text. Each log message is treated as a structured record with fields that are relevant to the context of the log message.

### **Benefits of Structured Logging**
- **Machine-readable**: Logs are easier to query, filter, and visualize.
- **Better troubleshooting**: Rich context in logs helps you pinpoint issues faster.
- **Easier integration**: Structured logs integrate well with log aggregators (e.g. Loki, Elasticsearch, or Datadog) for centralized logging.
- **Consistency**: Logs follow a consistent format, making them more reliable for automated systems to parse.

---

## **2. Best Practices for Structured Logging**

### **2.1 Use Consistent Log Levels**

Log levels allow you to categorize log messages based on their severity. The most common log levels are:

- **Trace**: Detailed information for debugging (e.g., variable values).
- **Debug**: Debugging information, useful during development.
- **Information**: Business-as-usual messages that show the application's general flow.
- **Warning**: Indicates something unexpected but non-critical (e.g., low disk space).
- **Error**: Indicates errors that cause issues but don’t stop the application.
- **Critical**: Severe issues requiring immediate attention (e.g., application crashes).

Each log level should be used appropriately to reflect the severity of the event.

#### **Good Example (Log Level Usage)**

```json
{
  "timestamp": "2021-09-27T20:46:37.1025564Z",
  "level": "information",
  "message": "get all access groups",
  "actionname": "asm.api.controllers.accessgroupcontroller.getall (asm.api)",
  "requestpath": "/api/v1.0/access-groups",
  "applicationname": "asm-api",
  "applicationversion": "1.0.0",
  "environment": "production",
  "loggername": "asm.util",
  "requestmethod": "get",
  "referer": "https://localhost:5001/swagger/index.html",
  "correlationid": "035cad88-3aa5-4708-a0dc-cc3f2d871a9a",
  "hostname": "172010-gvclw33",
  "clientip": "192.168.1.1",
  "clientagent": "mozilla/5.0 (windows nt 10.0; win64; x64) applewebkit/537.36 (khtml, like gecko) chrome/93.0.4577.82 safari/537.36",
  "processid": 46084,
  "processname": "asm.api",
  "threadid": 7,
  "servicename": "access-groups",
  "serviceversion": "1.0"
}

```

#### **Bad Example (Improper Log Level)**

```json
{
  "timestamp": "2025-02-08T10:00:00Z",
  "level": "Error",
  "message": "User logged in successfully",
}
```

In the bad example, logging a successful login with an `Error` level is misleading. This should be an `Information` log.

---

### **2.2 Include Relevant Contextual Information**

Each log should include context that makes the log entry valuable. The most common fields are:
- **timestamp**: The time the event occurred.
- **level**: The log level.
- **message**: A brief description of the event.
- **userId**: (Optional) The user related to the event.
- **sessionId**: (Optional) The session ID for user interactions.
- **ipAddress**: (Optional) The IP address of the user or service generating the log.
- **errorDetails**: (Optional) Detailed information about errors, including stack traces.

#### **Good Example (With Contextual Information)**

```json
{
  "timestamp": "2025-02-08T10:15:00Z",
  "level": "Error",
  "message": "Failed to process order",
  "orderId": 123456,
  "customerId": 78910,
  "errorDetails": {
    "exceptionType": "System.NullReferenceException",
    "stackTrace": "at OrderProcessor.ProcessOrder()"
  }
}
```

#### **Bad Example (Missing Contextual Information)**

```json
{
  "timestamp": "2025-02-08T10:15:00Z",
  "level": "Error",
  "message": "Failed to process order"
}
```

In the bad example, the lack of `orderId`, `customerId`, and `errorDetails` makes it difficult to debug the issue. The log is not useful without this context.

---

### **2.3 Avoid Logging Sensitive Data**

Never log sensitive data such as passwords, tokens, or personally identifiable information (PII). Mask or exclude sensitive data before logging.

#### **Good Example (Sensitive Data Masked)**

```json
{
  "timestamp": "2025-02-08T11:00:00Z",
  "level": "Information",
  "message": "User login attempt",
  "userId": 12345,
  "ipAddress": "192.168.1.1"
}
```

#### **Bad Example (Sensitive Data Logged)**

```json
{
  "timestamp": "2025-02-08T11:00:00Z",
  "level": "Information",
  "message": "User login attempt",
  "userId": 12345,
  "password": "password123",
  "ipAddress": "192.168.1.1"
}
```

In the bad example, logging the password is a security risk and violates best practices.

---

### **2.4 Use Structured Fields for Error Details**

When logging errors, ensure that you capture key information such as the exception type, message, stack trace, and any relevant parameters that caused the issue.

#### **Good Example (Error Logging with Full Context)**

```json
{
  "timestamp": "2025-02-08T11:30:00Z",
  "level": "Error",
  "message": "An error occurred while processing the order",
  "orderId": 123456,
  "customerId": 78910,
  "errorDetails": {
    "exceptionType": "System.ArgumentNullException",
    "message": "Argument cannot be null.",
    "parameterName": "order",
    "stackTrace": "at OrderProcessor.ProcessOrder()"
  }
}
```

#### **Bad Example (Error Logging without Details)**

```json
{
  "timestamp": "2025-02-08T11:30:00Z",
  "level": "Error",
  "message": "An error occurred while processing the order"
}
```

In the bad example, there is no information on what the error was, what caused it, or how to troubleshoot it.

---

### **2.5 Ensure Consistency in Log Structure**

All logs should follow a consistent structure, which makes it easier to query and analyze. Use the same field names for the same types of information across all log messages.

#### **Good Example (Consistent Structure)**

```json
{
  "timestamp": "2025-02-08T10:00:00Z",
  "level": "Information",
  "message": "User logged in",
  "userId": 12345,
  "ipAddress": "192.168.1.1",
  "sessionId": "abcd1234"
}
```

#### **Bad Example (Inconsistent Structure)**

```json
{
  "time": "2025-02-08T10:00:00Z",
  "severity": "Information",
  "text": "User logged in",
  "userId": 12345,
  "ip": "192.168.1.1",
  "sid": "abcd1234"
}
```

In the bad example, field names like `time`, `severity`, and `text` are inconsistent with the good example and can lead to confusion when querying logs.

---

## **3. Detailed Breakdown of Log Fields**

Each log entry should contain key fields that provide both high-level and detailed context.

### **3.1 Required Log Fields**
- **timestamp**: The time when the log entry was created. It should follow the **ISO 8601** format (`YYYY-MM-DDTHH:mm:ssZ`).
- **level**: The severity of the log message (e.g., `Information`, `Error`, `Debug`).
- **message**: A human-readable description of the log event.
- **userId**: (Optional) The ID of the user performing the action, useful for user-related events.
- **sessionId**: (Optional) The session ID of the user or service making the request.
- **ipAddress**: (Optional) The IP address of the user or system generating the log.
- **correlationId**: (Optional) A unique identifier that ties together logs from distributed services (useful in microservices architectures).

### **3.2 Error-Specific Fields**
- **exceptionType**: The type of exception (e.g., `NullReferenceException`, `ArgumentException`).
- **exceptionMessage**: The error message associated with the exception.
- **stackTrace**: The full stack trace for the exception.
- **innerException**: If the exception has an inner exception, log it as well.

### **3.3 Performance-Related Fields**
- **responseTime**: (Optional) The time taken to process a request.
- **requestId**: (Optional) The unique ID of the request being logged, useful for tracing requests across systems.
- **elapsedTime**: (Optional) The time taken for a specific operation or method.

---

Here’s a structured table definition for the JSON log data:

| Column Name         | Data Type     | Required/Optional | Description                                              | Use Case / Sample Value                                      |
|---------------------|--------------|-------------------|----------------------------------------------------------|--------------------------------------------------------------|
| `timestamp`        | `DATETIME`    | Required          | The time the log entry was created                      | `2021-09-27T20:46:37.1025564Z`                               |
| `loglevel`         | `VARCHAR(20)` | Required          | The severity level of the log                           | `information`                                                |
| `message`          | `TEXT`        | Required          | Descriptive log message                                 | `get all access groups`                                      |
| `actionname`       | `TEXT`        | Optional          | Fully qualified action name                             | `asm.api.controllers.accessgroupcontroller.getall (asm.api)` |
| `requestpath`      | `TEXT`        | Optional          | API request endpoint                                    | `/api/v1.0/access-groups`                                    |
| `applicationname`  | `VARCHAR(100)`| Required          | Name of the application generating the log              | `asm-api`                                                    |
| `applicationversion` | `VARCHAR(20)` | Optional       | Version of the application                             | `1.0.0`                                                      |
| `environment`      | `VARCHAR(50)` | Required          | Environment where the log was generated (e.g., Dev, QA, Prod) | `production`                                           |
| `loggername`       | `VARCHAR(100)`| Optional          | Name of the logger component                           | `asm.util`                                                   |
| `requestmethod`    | `VARCHAR(10)` | Optional          | HTTP method of the request                             | `GET`                                                        |
| `referer`          | `TEXT`        | Optional          | URL of the referring page                              | `https://localhost:5001/swagger/index.html`                  |
| `correlationid`    | `UUID`        | Required          | Unique identifier for tracking request logs            | `035cad88-3aa5-4708-a0dc-cc3f2d871a9a`                       |
| `hostname`        | `VARCHAR(255)` | Optional          | Name of the host machine                               | `172010-gvclw33`                                             |
| `clientip`        | `VARCHAR(45)`  | Optional          | IP address of the client making the request           | `192.168.1.1`                                               |
| `clientagent`     | `TEXT`        | Optional          | User-Agent string of the client browser/device        | `mozilla/5.0 (windows nt 10.0; win64; x64)...`               |
| `processid`       | `INT`         | Optional          | ID of the running process                             | `46084`                                                      |
| `processname`     | `VARCHAR(255)` | Optional          | Name of the process running the application           | `asm.api`                                                    |
| `threadid`        | `INT`         | Optional          | Thread ID handling the request                        | `7`                                                          |
| `servicename`     | `VARCHAR(255)` | Optional          | Name of the service handling the request              | `access-groups`                                              |
| `serviceversion`  | `VARCHAR(20)`  | Optional          | Version of the service                                | `1.0`                                                        |

This schema ensures structured storage of log data with relevant details for debugging, monitoring, and tracing system behavior.

## **4. Conclusion**

Structured logging is a powerful technique that enhances debugging, performance monitoring, and observability in applications. By following best practices such as using consistent log levels, including detailed context, avoiding sensitive data, and maintaining a consistent log structure, you ensure your logs are valuable for debugging and monitoring.

Structured logs help you quickly identify problems, monitor performance, and maintain reliable and secure applications.

---

Would you like additional details on integrating structured logging with specific tools like **Grafana**, **Elasticsearch**, or **Loki**?
