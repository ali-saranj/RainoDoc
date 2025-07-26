Below is a **comprehensive and professional documentation** for the Kasra Hamrah API endpoints, combining the provided structure, the new **API Response Value Descriptions** section, and enhancements for clarity, readability, and professionalism. The document is structured to serve as a definitive guide for developers, with a focus on workflow, integration, and practical use cases. It incorporates all suggestions, addresses enum mismatches, and includes optimization and security best practices.

---

# **Kasra Hamrah API Endpoint Workflow Documentation**

**A Comprehensive Guide to API Endpoints, Their Workflow, and Integration in the Application**

---

**Version**: 1.0  
**Date Created**: April 26, 2025  
**Author**: Grok 3, built by xAI  

---

## **Table of Contents**

1. [Overview](#overview)
2. [Flow Summary](#flow-summary)
3. [API Endpoints and Workflow](#api-endpoints-and-workflow)
   - [Health Check](#1-health-check)
   - [User Login (Authentication)](#2-user-login-authentication)
   - [Fetch User Info](#3-fetch-user-info)
   - [Fetch User Access Permissions](#4-fetch-user-access-permissions)
   - [Get All Work Periods](#5-get-all-work-periods)
   - [Get Calendar Day Items](#6-get-calendar-day-items)
   - [Get Monthly Performance Audit Summary](#7-get-monthly-performance-audit-summary)
   - [Fetch Workplace Changes](#8-fetch-workplace-changes)
   - [Fetch Logs Grouped by Workplace](#9-fetch-logs-grouped-by-workplace)
   - [Fetch Badges](#10-fetch-badges)
4. [API Response Value Descriptions](#api-response-value-descriptions)
5. [Error Handling](#error-handling)
6. [Notes](#notes)
7. [Application Workflow Diagram](#application-workflow-diagram)
8. [Real-World Use Cases](#real-world-use-cases)
9. [Security & Best Practices](#security--best-practices)
10. [Optimization Strategies](#optimization-strategies)
11. [Glossary](#glossary)
12. [Conclusion](#conclusion)

---

## **Overview**

The **Kasra Hamrah** application is a workplace management and attendance tracking platform designed to streamline employee work periods, attendance, and performance metrics. This document provides a **detailed guide** to the API endpoints, their workflow, and their integration into the application, with a focus on the **monthly performance and calendar data fragment**. Key APIs include `GetAllWorkPeriods`, `GetCalendarDayItems`, and `GetMonthlyPerformanceAuditSummary`, alongside supporting endpoints like `/Health`, `/UsersInfo`, and `/WorkplacesChanges`.

The documentation is structured to assist developers in:
- Understanding endpoint purposes, parameters, headers, and responses.
- Implementing efficient workflows for authentication, data fetching, and UI rendering.
- Optimizing performance through caching and error handling.
- Ensuring secure and user-friendly application behavior.

---

## **Flow Summary**

The application workflow follows these stages, each supported by specific API endpoints:

1. **Initialization**: Verifies backend availability (`/Health`).
2. **Authentication**: Authenticates users and retrieves a JWT token (`/token`).
3. **User Profile**: Loads user data and permissions (`/UsersInfo`, `/GetUserAccesses`).
4. **Work Period Selection**: Fetches available work periods (`GetAllWorkPeriods`).
5. **Calendar & Performance Data**: Retrieves daily calendar data and monthly performance metrics (`GetCalendarDayItems`, `GetMonthlyPerformanceAuditSummary`).
6. **Workplace Management**: Syncs workplace data and attendance logs (`/WorkplacesChanges`, `/LogsGroupedByWorkplace`).
7. **Notifications**: Displays pending tasks and issues (`/GetBadges`).

---

## **API Endpoints and Workflow**

### **1. Health Check**

#### **Endpoint**
```
GET https://kasra.kasraco.net/lego.web/api/Avid/Health
```

#### **Description**
Verifies the operational status of the backend service, ensuring the application can proceed with user interactions.

#### **Parameters**
| Name | Type | Description | Example |
|------|------|-------------|---------|
| None | -    | -           | -       |

#### **Headers**
- `Authorization: Bearer <token>`: Required for secure access.

#### **Response**
- **Status Code**: `200 OK`
- **Body Example**:
  ```json
  {
    "health": true,
    "connectionStatus": true,
    "isHTTPS": true
  }
  ```
- **Fields Description**:
  | Field              | Type    | Description                                      |
  |--------------------|---------|--------------------------------------------------|
  | `health`           | Boolean | Indicates if the service is operational          |
  | `connectionStatus` | Boolean | Indicates database connectivity status           |
  | `isHTTPS`          | Boolean | Confirms if the request uses HTTPS               |

#### **Workflow Integration**
- **When**: Triggered on app launch or during critical operations.
- **Role**: Ensures backend availability before proceeding with authentication or data fetching.
- **Next Steps**: If successful, proceeds to authentication; if failed, displays a maintenance message or retries.

#### **Use Case**
On app startup, confirms the server is operational, preventing errors due to downtime.

#### **Code Reference**
- **Class/Method**: `HealthCheckResponse` in `com.kasra.avid.data.model.ip`

---

### **2. User Login (Authentication)**

#### **Endpoint**
```
POST https://kasra.kasraco.net/kevlar/oauth/token
```

#### **Description**
Authenticates users and issues a JWT token for subsequent API requests.

#### **Parameters**
| Name        | Type   | Description                     | Example         |
|-------------|--------|---------------------------------|-----------------|
| `grant_type`| String | Type of grant                   | `password`      |
| `username`  | String | User’s username                 | `user123`       |
| `password`  | String | User’s password                 | `pass123`       |
| `client_id` | String | Client identifier               | `kasra-client`  |

#### **Headers**
- None required.

#### **Response**
- **Status Code**: `200 OK`
- **Body Example**:
  ```json
  {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expires_in": 3600,
    "token_type": "Bearer"
  }
  ```

#### **Workflow Integration**
- **When**: Triggered on the login screen when users submit credentials.
- **Role**: Validates credentials and provides a token for authenticated API calls.
- **Next Steps**: Stores the token securely and fetches user profile data.

#### **Use Case**
An employee logs in, and the app stores the token to authenticate requests for profile data or attendance logs.

#### **Code Reference**
- **Class/Method**: `AppDataManager` in `com.kasra.avid.data`

---

### **3. Fetch User Info**

#### **Endpoint**
```
GET https://kasra.kasraco.net/lego.web/api/avid/UsersInfo
```

#### **Description**
Retrieves user profile data (e.g., name, role, department) to personalize the application experience.

#### **Parameters**
| Name | Type | Description | Example |
|------|------|-------------|---------|
| None | -    | -           | -       |

#### **Headers**
- `Authorization: Bearer <token>`: Required for secure access.

#### **Response**
- **Status Code**: `200 OK`
- **Body Example**:
  ```json
  {
    "ImageUrl": "/lego.web/api/Avid/UsersInfo/GetUserImage?...",
    "DepartmentName": "روزر",
    "RoleTitle": "روزروزروزر",
    "FullName": "موزروزروزر",
    "Person": {
      "Id": 159863838,
      "Name": "موزروزروزر",
      "Code": "159863838",
      "CodeMask": "1403222"
    },
    "Culture": "fa-IR",
    "EmailConfirmed": false
  }
  ```
- **Fields Description**:
  | Field              | Type    | Description                                      |
  |--------------------|---------|--------------------------------------------------|
  | `ImageUrl`         | String  | URL to retrieve user’s profile image             |
  | `DepartmentName`   | String  | Name of the user’s department                    |
  | `RoleTitle`        | String  | User’s role in the system                        |
  | `FullName`         | String  | User’s full name                                 |
  | `Person.Id`        | Integer | Unique identifier for the person                 |
  | `Person.Code`      | String  | Employee code                                    |
  | `Culture`          | String  | User’s language/culture setting (e.g., `fa-IR`)  |

#### **Workflow Integration**
- **When**: Called after successful login.
- **Role**: Loads user data to customize the dashboard and provide context (e.g., `PersonCode` for other APIs).
- **Next Steps**: Triggers fetches for permissions and settings.

#### **Use Case**
Displays the user’s name and department on the dashboard, tailoring the UI based on their role.

#### **Code Reference**
- **Class/Method**: `UserInfo` in `com.kasra.avid.data.model`

---

### **4. Fetch User Access Permissions**

#### **Endpoint**
```
GET https://kasra.kasraco.net/lego.web/api/avid/UsersInfo/GetUserAccesses
```

#### **Description**
Retrieves a list of features or actions the user is authorized to perform, controlling UI visibility and functionality.

#### **Parameters**
| Name | Type | Description | Example |
|------|------|-------------|---------|
| None | -    | -           | -       |

#### **Headers**
- `Authorization: Bearer <token>`: Required for secure access.

#### **Response**
- **Status Code**: `200 OK`
- **Body Example**:
  ```json
  [
    {"Id": 2800037, "SValue": "AllPk"},
    {"Id": 34201452, "SValue": "BtnExcel"},
    {"Id": 2800011, "SValue": "L.M.Avid.Mobile.Attendance.RegisterAttendance"}
  ]
  ```
- **Fields Description**:
  | Field    | Type    | Description                                      |
  |----------|---------|--------------------------------------------------|
  | `Id`     | Integer | Unique identifier for the permission            |
  | `SValue` | String  | Permission key (e.g., `BtnExcel`, `AllPk`)       |

#### **Workflow Integration**
- **When**: Called after user info is loaded.
- **Role**: Determines which UI elements and features are accessible.
- **Next Steps**: Configures the UI to reflect permissions (e.g., enable/disable buttons).

#### **Use Case**
Hides the “Export to Excel” button for users without `BtnExcel` permission.

#### **Code Reference**
- **Class/Method**: `PersonAccess` in `com.kasra.avid.data.model`

---

### **5. Get All Work Periods**

#### **Endpoint**
```
GET https://kasra.kasraco.net/lego.web/api/KasraHamrah/WorkPeriodAPI/GetAllWorkPeriods?endDate={endDate}
```

#### **Description**
Retrieves a list of work periods (e.g., months) up to a specified end date, enabling users to select a period for viewing calendar data and performance metrics.

#### **Parameters**
| Name      | Type   | Description                             | Example      |
|-----------|--------|-----------------------------------------|--------------|
| `endDate` | String | End date for filtering work periods     | `2025-04-07` |

#### **Headers**
- `Authorization: Bearer <token>`: Required for secure access.

#### **Response**
- **Status Code**: `200 OK`
- **Body Example**:
  ```json
  [
    {
      "Id": 183,
      "StartDate": "2025-04-21T00:00:00",
      "EndDate": "2025-05-21T00:00:00",
      "Name": "اردیبهشت 1404",
      "MounthName": "April",
      "IsCurrent": false,
      "IsExpired": false
    },
    {
      "Id": 182,
      "StartDate": "2025-03-21T00:00:00",
      "EndDate": "2025-04-20T00:00:00",
      "Name": "فروردین 1404",
      "MounthName": "March",
      "IsCurrent": true,
      "IsExpired": false
    }
  ]
  ```
- **Fields Description**:
  | Field         | Type    | Description                                  |
  |---------------|---------|----------------------------------------------|
  | `Id`          | Integer | Unique identifier for the work period        |
  | `StartDate`   | String  | Start date of the work period (ISO format)   |
  | `EndDate`     | String  | End date of the work period (ISO format)     |
  | `Name`        | String  | Persian name of the work period (e.g., month)|
  | `MounthName`  | String  | English name of the month                    |
  | `IsCurrent`   | Boolean | Indicates if this is the current work period |
  | `IsExpired`   | Boolean | Indicates if the work period has expired     |

#### **Workflow Integration**
- **When**: Called when navigating to the monthly performance or calendar fragment.
- **Role**: Provides a list of work periods for user selection, foundational for calendar and performance data.
- **Next Steps**: The selected `workPeriodId` is used for `GetCalendarDayItems` and `GetMonthlyPerformanceAuditSummary`.

#### **Use Case**
Displays a dropdown of work periods (e.g., “April 1404”), allowing users to select a month for viewing metrics.

#### **Code Reference**
- **Class/Method**: `Month` in `com.kasra.avid.utility.calendar.model`

---

### **6. Get Calendar Day Items**

#### **Endpoint**
```
GET https://kasra.kasraco.net/lego.web/api/KasraHamrah/CalendarAPI/GetCalendarItems?workPeriodId={workPeriodId}&PersonCode={PersonCode}
```

#### **Description**
Fetches daily calendar data for a specific work period and person, including day and request statuses, to construct the calendar UI.

#### **Parameters**
| Name           | Type   | Description                           | Example     |
|----------------|--------|---------------------------------------|-------------|
| `workPeriodId` | Integer| ID of the selected work period        | `183`       |
| `PersonCode`   | Integer| Unique code of the person             | `159863725` |

#### **Headers**
- `Authorization: Bearer <token>`: Required for secure access.

#### **Response**
- **Status Code**: `200 OK`
- **Body Example**:
  ```json
  [
    {
      "Id": 38017,
      "Date": "2025-04-19T00:00:00",
      "DayStatus": 12005,
      "RequestsStatus": 13003,
      "IsToday": false,
      "IsOutOfWorkPeriod": true,
      "HaveMissedAttendance": false
    },
    {
      "Id": 38019,
      "Date": "2025-04-21T00:00:00",
      "DayStatus": 12005,
      "RequestsStatus": 13003,
      "IsToday": false,
      "IsOutOfWorkPeriod": false,
      "HaveMissedAttendance": false
    }
  ]
  ```
- **Fields Description**:
  | Field                | Type    | Description                                      |
  |----------------------|---------|--------------------------------------------------|
  | `Id`                 | Integer | Unique identifier for the calendar day item      |
  | `Date`               | String  | Date of the calendar day (ISO format)            |
  | `DayStatus`          | Integer | Status of the day (e.g., working, holiday)       |
  | `RequestsStatus`     | Integer | Status of requests for the day (e.g., pending)   |
  | `IsToday`            | Boolean | Indicates if the day is today                    |
  | `IsOutOfWorkPeriod`  | Boolean | Indicates if the day is outside the work period  |
  | `HaveMissedAttendance`| Boolean | Indicates if attendance was missed on this day   |

- **DayStatus Enum**:
  | Value  | Description              |
  |--------|--------------------------|
  | `12001`| Holiday but working day  |
  | `12002`| Working day              |
  | `12003`| Holiday                  |
  | `12004`| Off day                  |
  | `12005`| Unknown (pending clarification) |

- **RequestStatus Enum**:
  | Value | Description         |
  |-------|---------------------|
  | `201` | Pending request     |
  | `203` | Approved request    |
  | `204` | Denied request      |
  | `205` | Deleted request     |
  | `209` | Canceled request    |
  | `0`   | No request          |
  | `-1`  | Withdrawn request   |
  | `13003`| Unknown (pending clarification) |

#### **Workflow Integration**
- **When**: Called after selecting a work period.
- **Role**: Provides daily data to render the calendar UI, with visual indicators for statuses.
- **Next Steps**: Populates the calendar UI, enabling interaction with days.

#### **Use Case**
Renders a calendar for April 2025, highlighting holidays and working days, with indicators for pending requests.

#### **Code Reference**
- **Class/Method**: `Day` in `com.kasra.avid.utility.calendar.model`

---

### **7. Get Monthly Performance Audit Summary**

#### **Endpoint**
```
GET https://kasra.kasraco.net/lego.web/api/KasraHamrah/PerformanceAuditAPI/GetMonthlyPerformanceAuditSummary?workPeriodId={workPeriodId}&PersonCode={PersonCode}
```

#### **Description**
Retrieves monthly performance metrics (e.g., work hours, overtime, absences) for a specific person and work period.

#### **Parameters**
| Name           | Type   | Description                           | Example     |
|----------------|--------|---------------------------------------|-------------|
| `workPeriodId` | Integer| ID of the selected work period        | `182`       |
| `PersonCode`   | Integer| Unique code of the person             | `159863725` |

#### **Headers**
- `Authorization: Bearer <token>`: Required for secure access.

#### **Response**
- **Status Code**: `200 OK`
- **Body Example**:
  ```json
  [
    {
      "Id": 2,
      "Code": {
        "Id": 10102,
        "Name": "كسر حضور ساعتي",
        "Nature": {
          "Id": 1,
          "Name": "ساعتي",
          "Symbol": "ساعت",
          "UseNature": false
        },
        "IsDaily": false,
        "IsRegulationTime": false,
        "IsOvertime": false,
        "IsRequestNeed": true,
        "IsSelectable": true,
        "Color": "#FC5C65"
      },
      "Value": "13:46",
      "DaysCount": 0,
      "HoursCount": 13,
      "MinutesCount": 46,
      "RequestNeedCount": 5
    },
    {
      "Id": 3,
      "Code": {
        "Id": 10103,
        "Name": "مازادحضور",
        "Nature": {
          "Id": 1,
          "Name": "ساعتي",
          "Symbol": "ساعت",
          "UseNature": false
        },
        "IsOvertime": true,
        "Color": "#FFB82C"
      },
      "Value": "00:26",
      "HoursCount": 0,
      "MinutesCount": 26,
      "RequestNeedCount": 1
    }
  ]
  ```
- **Fields Description**:
  | Field             | Type    | Description                                      |
  |-------------------|---------|--------------------------------------------------|
  | `Id`              | Integer | Unique identifier for the performance entry      |
  | `Code.Id`         | Integer | Unique identifier for the metric type            |
  | `Code.Name`       | String  | Name of the metric (e.g., "Hourly Absence")       |
  | `Code.Nature`     | Object  | Nature of the metric (e.g., hourly)              |
  | `Code.IsOvertime` | Boolean | Indicates if the metric represents overtime      |
  | `Code.Color`      | String  | Hex color code for UI representation             |
  | `Value`           | String  | Total time in "HH:mm" format                     |
  | `HoursCount`      | Integer | Number of hours for this metric                  |
  | `MinutesCount`    | Integer | Number of minutes for this metric                |
  | `RequestNeedCount`| Integer | Number of pending requests for this metric       |

#### **Workflow Integration**
- **When**: Called after selecting a work period, in parallel with `GetCalendarDayItems`.
- **Role**: Provides performance metrics for the monthly performance fragment.
- **Next Steps**: Visualizes metrics in the UI with color-coded indicators.

#### **Use Case**
Displays 13:46 hours of absences (red) and 0:26 hours of overtime (yellow) for March 2025.

#### **Code Reference**
- **Class/Method**: `Month` in `com.kasra.avid.utility.calendar.model`

---

### **8. Fetch Workplace Changes**

#### **Endpoint**
```
GET https://kasra.kasraco.net/lego.web/api/avid/Workplaces/WorkplacesChanges
```

#### **Description**
Retrieves updates to workplace data (e.g., new locations, deletions) for accurate attendance tracking.

#### **Parameters**
| Name | Type | Description | Example |
|------|------|-------------|---------|
| None | -    | -           | -       |

#### **Headers**
- `Authorization: Bearer <token>`: Required for secure access.
- `If-Modified-Since`: Optional, for caching.

#### **Response**
- **Status Code**: `200 OK`
- **Body Example**:
  ```json
  [
    {
      "Id": "6d598f47-406c-4041-90d0-0065383275ea",
      "IsDeleted": true,
      "Name": "بيضنين",
      "Address": "بيضنين",
      "Latitude": 32.623804789070256,
      "Longitude": 51.625985890986584,
      "Radius": 100,
      "CompanyId": 1,
      "Recorders": [
        {
          "Id": "95c8b91e-8634-473d-8d3b-3b76be438d3b",
          "Name": "بيضنين",
          "IsPrimary": true
        }
      ]
    }
  ]
  ```
- **Fields Description**:
  | Field             | Type    | Description                                      |
  |-------------------|---------|--------------------------------------------------|
  | `Id`              | String  | Unique identifier for the workplace (UUID)       |
  | `IsDeleted`       | Boolean | Indicates if the workplace is marked as deleted  |
  | `Name`            | String  | Name of the workplace                            |
  | `Address`         | String  | Address of the workplace                         |
  | `Latitude`        | Float   | Latitude of the workplace                        |
  | `Longitude`       | Float   | Longitude of the workplace                       |
  | `Radius`          | Integer | Effective area around the workplace              |
  | `Recorders`       | Array   | List of recorders associated with the workplace  |

#### **Workflow Integration**
- **When**: Called during dashboard initialization or workplace data refresh.
- **Role**: Syncs workplace data for location-based attendance tracking.
- **Next Steps**: Updates the workplace list for check-in/out operations.

#### **Use Case**
Syncs a new workplace, ensuring employees can check in at the correct location.

#### **Code Reference**
- **Class/Method**: `Workplace` in `com.kasra.avid.data.model.RegisterAttendance`

---

### **9. Fetch Logs Grouped by Workplace**

#### **Endpoint**
```
GET https://kasra.kasraco.net/lego.web/api/avid/Logs/LogsGroupedByWorkplace?date={date}
```

#### **Description**
Retrieves attendance logs (check-ins/outs) grouped by workplace for a specific date, supporting history and compliance checks.

#### **Parameters**
| Name  | Type   | Description                           | Example                  |
|-------|--------|---------------------------------------|--------------------------|
| `date`| String | Date for which logs are requested     | `2025-04-21T23:59:59.000`|

#### **Headers**
- `Authorization: Bearer <token>`: Required for secure access.

#### **Response**
- **Status Code**: `200 OK`
- **Body Example**:
  ```json
  {
    "Id": 1,
    "HasMissedLog": true,
    "MissedLogsCount": 3,
    "Logs": [
      {
        "Id": "abecc16e-8826-46d6-a49b-bd8a578 personally identifiable information3a13",
        "Date": "2025-04-21T23:59:59.000+03:30",
        "Enter": {
          "Id": "1b41d9e5-5107-4af0-9c2a-c2edd3058812",
          "Date": "2025-04-21T15:07:42.863+03:30",
          "Latitude": 32.625331341309135,
          "IsMissed": false
        },
        "Exit": {
          "Id": "eaf8279c-8624-4b21-a690-0b539f611fe6",
          "IsMissed": true
        }
      }
    ]
  }
  ```
- **Fields Description**:
  | Field             | Type    | Description                                      |
  |-------------------|---------|--------------------------------------------------|
  | `Id`              | Integer | Unique identifier for the log group              |
  | `HasMissedLog`    | Boolean | Indicates if there are missed logs               |
  | `MissedLogsCount` | Integer | Number of missed logs                            |
  | `Logs`            | Array   | List of log entries (check-in/check-out pairs)   |

#### **Workflow Integration**
- **When**: Called when viewing attendance history or reviewing team logs.
- **Role**: Provides detailed attendance data, highlighting missed logs.
- **Next Steps**: Allows corrections for missing logs (if permitted).

#### **Use Case**
A manager reviews team attendance for April 21, 2025, identifying missing check-outs.

#### **Code Reference**
- **Class/Method**: `LogsGroupByDate` in `com.kasra.avid.data.model.attendance`

---

### **10. Fetch Badges**

#### **Endpoint**
```
GET https://kasra.kasraco.net/lego.web/api/avid/UsersInfo/GetBadges
```

#### **Description**
Displays pending tasks or issues (e.g., incomplete attendances) as dashboard notifications.

#### **Parameters**
| Name | Type | Description | Example |
|------|------|-------------|---------|
| None | -    | -           | -       |

#### **Headers**
- `Authorization: Bearer <token>`: Required for secure access.

#### **Response**
- **Status Code**: `200 OK`
- **Body Example**:
  ```json
  [
    {
      "Id": 1,
      "Count": 0,
      "Type": {
        "Id": 71,
        "Name": "كرفته كرفته كرفته",
        "Value": 11501
      }
    },
    {
      "Id": 2,
      "Count": 0,
      "Type": {
        "Id": 102,
        "Name": "رفته كرفته",
        "Value": 11502
      }
    }
  ]
  ```
- **Fields Description**:
  | Field         | Type    | Description                                      |
  |---------------|---------|--------------------------------------------------|
  | `Id`          | Integer | Unique identifier for the badge                  |
  | `Count`       | Integer | Number of items (e.g., incomplete attendances)   |
  | `Type.Name`   | String  | Description of the badge                         |
  | `Type.Value`  | Integer | Unique code for the badge type                   |

#### **Workflow Integration**
- **When**: Called during dashboard initialization.
- **Role**: Alerts users to pending actions.
- **Next Steps**: Enables navigation to resolve issues.

#### **Use Case**
Notifies an employee of incomplete attendances, prompting record corrections.

#### **Code Reference**
- **Class/Method**: `Badge` in `com.kasra.avid.data.model`

---

## **API Response Value Descriptions**

This section explains the role and impact of each value returned by the API endpoints within the Kasra Hamrah application, providing clarity for developers on how to leverage these values.

### **1. Health Check**
- **`health`**: Determines if the app can proceed with user interactions. A `false` value triggers a maintenance message.
- **`connectionStatus`**: Ensures database connectivity, critical for data-driven features.
- **`isHTTPS`**: Verifies secure communication, essential for user data protection.

### **2. User Login (Authentication)**
- **`access_token`**: Authenticates all subsequent API requests, stored securely in the app.
- **`expires_in`**: Guides token refresh to maintain session continuity.
- **`token_type`**: Specifies the token format (`Bearer`) for header configuration.

### **3. Fetch User Info**
- **`ImageUrl`**: Personalizes the UI with the user’s profile image.
- **`DepartmentName`**, **`RoleTitle`**, **`FullName`**: Customize the dashboard for context.
- **`Person.Id`**, **`Person.Code`**: Enable user-specific API calls (e.g., performance data).
- **`Culture`**: Localizes content based on user language settings.

### **4. Fetch User Access Permissions**
- **`Id`**: Tracks permissions internally.
- **`SValue`**: Controls UI visibility and functionality (e.g., `BtnExcel` enables export features).

### **5. Get All Work Periods**
- **`Id`**: Links to calendar and performance data fetches.
- **`StartDate`**, **`EndDate`**: Define the period for data filtering.
- **`Name`**, **`MounthName`**: Display period options in the UI.
- **`IsCurrent`**: Highlights the active period.
- **`IsExpired`**: Manages visibility of past periods.

### **6. Get Calendar Day Items**
- **`Id`**: Manages calendar day data.
- **`Date`**: Positions data in the calendar UI.
- **`DayStatus`**: Colors days (e.g., gray for holidays).
- **`RequestsStatus`**: Indicates request actions (e.g., pending requests).
- **`IsToday`**: Highlights the current day.
- **`IsOutOfWorkPeriod`**: Hides irrelevant days.
- **`HaveMissedAttendance`**: Prompts attendance corrections.

### **7. Get Monthly Performance Audit Summary**
- **`Id`**: Tracks performance entries.
- **`Code`**: Defines the metric type and styling (e.g., `Color` for UI).
- **`Value`**, **`HoursCount`**, **`MinutesCount`**: Display performance summaries.
- **`RequestNeedCount`**: Prompts request-related actions.

### **8. Fetch Workplace Changes**
- **`Id`**: Identifies workplaces.
- **`IsDeleted`**: Updates workplace availability.
- **`Name`**, **`Address`**: Provide context in the UI.
- **`Latitude`**, **`Longitude`**, **`Radius`**: Enable geofencing for attendance.
- **`Recorders`**: Link attendance devices to workplaces.

### **9. Fetch Logs Grouped by Workplace**
- **`Id`**: Organizes log groups.
- **`HasMissedLog`**, **`MissedLogsCount`**: Highlight issues for resolution.
- **`Logs`**: Display detailed attendance history.

### **10. Fetch Badges**
- **`Id`**: Tracks notifications.
- **`Count`**: Quantifies pending actions.
- **`Type`**: Describes the issue, guiding user actions.

---

## **Error Handling**

- **400 Bad Request**: Invalid/missing parameters. Validate inputs before requests.
- **401 Unauthorized**: Invalid/missing token. Prompt re-login.
- **500 Internal Server Error**: Server issue. Retry with exponential backoff.
- **Custom Handling**:
  - Cache responses for offline use.
  - Display user-friendly messages (e.g., “Please log in again” for `401`).
  - Implement retry logic for transient errors.

---

## **Notes**

- **Authentication**: All requests require a `Bearer` token in the `Authorization` header.
- **Dependencies**:
  - `GetCalendarDayItems` and `GetMonthlyPerformanceAuditSummary` require `workPeriodId` from `GetAllWorkPeriods`.
  - `PersonCode` is sourced from `/UsersInfo`.
- **Enum Mismatches**:
  - `DayStatus: 12005` and `RequestsStatus: 13003` in `GetCalendarDayItems` are undefined. Verify with the backend team to extend enums or clarify mappings.
- **Time Format**: `Value` fields use `HH:mm` format.
- **Caching**:
  - Cache `GetAllWorkPeriods`, `/UsersInfo`, and `/GetUserAccesses` for infrequent changes.
  - Use `If-Modified-Since` for `/WorkplacesChanges`.
- **Offline Support**: Store logs locally if `LiveTrackingAllowOffline` is enabled, syncing when online.

---

## **Application Workflow Diagram**

```
[App Launch]
   ↓
[Health Check] → Success? → [Login Screen]
   ↓                       ↓
[Login] → Token → [Fetch User Info] → [Fetch User Access]
   ↓
[Get All Work Periods] → Select Period
   ↓
[Get Calendar Day Items] + [Get Monthly Performance Audit Summary]
   ↓
[Fetch Badges] → [Fetch Workplace Changes] → [Fetch Logs Grouped by Workplace]
   ↓
[Render Dashboard: Calendar, Performance Metrics, Notifications]
   ↓
[User Actions: Check-in, Resolve Tasks, View Reports]
```

---

## **Real-World Use Cases**

### **Use Case 1: Employee Views Monthly Performance**
- **Scenario**: An employee checks their performance for April 2025.
- **Workflow**:
  1. **Health Check**: Confirms backend availability.
  2. **Login**: Retrieves token.
  3. **User Info**: Loads `PersonCode` (e.g., `159863725`).
  4. **Get All Work Periods**: Displays periods (e.g., “April 1404”).
  5. **Get Calendar Day Items**: Renders calendar with statuses.
  6. **Get Monthly Performance Audit Summary**: Shows 13:46 hours of absences, 0:26 hours of overtime.
  7. **Badges**: Notifies of pending requests.
  8. **Action**: Employee submits a request to address absences.

### **Use Case 2: Manager Reviews Team Attendance**
- **Scenario**: A manager reviews team attendance and workplace data.
- **Workflow**:
  1. **Health Check & Login**: Authenticates manager.
  2. **User Access**: Confirms `BtnExcel` permission.
  3. **Logs Grouped by Workplace**: Retrieves team logs.
  4. **Workplace Changes**: Syncs workplace data.
  5. **Action**: Exports logs to Excel.

---

## **Security & Best Practices**

- **Authentication**:
  - Use `Bearer` tokens for all requests.
  - Store tokens in encrypted storage.
- **HTTPS**:
  - Enforce HTTPS (`isHTTPS` in `/Health`).
- **Caching**:
  - Cache stable data (`GetAllWorkPeriods`, `/UsersInfo`).
  - Use `If-Modified-Since` for `/WorkplacesChanges`.
- **Error Handling**:
  - Handle `401` by prompting re-login.
  - Retry transient errors with backoff.
- **Permissions**:
  - Validate `/GetUserAccesses` before enabling features.
- **Offline Support**:
  - Store logs locally for offline tracking, syncing when online.

---

## **Optimization Strategies**

- **Minimize API Calls**:
  - Cache `GetAllWorkPeriods` and `/UsersInfo`.
  - Fetch `GetCalendarDayItems` and `GetMonthlyPerformanceAuditSummary` in parallel.
- **Efficient Data Handling**:
  - Filter `LogsGroupedByWorkplace` client-side.
  - Parse `Value` (HH:mm) client-side for performance.
- **UI Responsiveness**:
  - Prioritize `GetBadges` and `GetAllWorkPeriods` for quick rendering.
  - Use lazy loading for large calendars.

---

## **Glossary**

- **Bearer Token**: A security token for authenticating API requests.
- **ISO Format**: Standard date/time format (e.g., `2025-04-21T00:00:00`).
- **Enum**: A set of named values (e.g., `DayStatus`).
- **HTTP Status Codes**: Codes indicating request results (e.g., `200 OK`).
- **JSON**: Lightweight data interchange format.

---

## **Conclusion**

This documentation provides a **professional and comprehensive guide** to the Kasra Hamrah API endpoints, detailing their workflow, integration, and practical applications. Developers can use it to:
- Implement endpoints with clear parameters and responses.
- Optimize performance through caching and parallel requests.
- Enhance user experience with permission-based UI and error handling.
- Ensure security with authentication and HTTPS.

For clarification on enum mismatches (e.g., `DayStatus: 12005`) or additional endpoints, consult the backend team.

---

This document is designed to be a **definitive resource** for Kasra Hamrah API integration. If you need further enhancements, code snippets, or troubleshooting sections, please let me know!