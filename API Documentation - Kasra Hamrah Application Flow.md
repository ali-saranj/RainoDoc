## Overview
This document outlines the API flow for the Kasra Hamrah application, focusing on the fragment that handles monthly performance and calendar data. The flow consists of three key APIs: `GetAllWorkPeriods`, `GetCalendarDayItems`, and `GetMonthlyPerformanceAuditSummary`. These APIs fetch work periods, daily calendar data, and performance metrics, respectively.

---

## Flow Summary
1. **GetAllWorkPeriods**: Retrieves a list of work periods (e.g., months) for selection.
2. **GetCalendarDayItems**: Fetches daily calendar data for the selected work period, including day and request statuses, to build the calendar UI.
3. **GetMonthlyPerformanceAuditSummary**: Provides a summary of performance metrics (e.g., work hours, overtime, absences) for the selected work period.

---

## 1. GetAllWorkPeriods API

### Endpoint
```
GET https://kasra.kasraco.net/lego.web/api/KasraHamrah/WorkPeriodAPI/GetAllWorkPeriods?endDate={endDate}
```

### Description
This API retrieves a list of all work periods up to a specified end date, providing the foundation for selecting a period to view calendar data and performance metrics.

### Parameters
| Name      | Type   | Description                             | Example          |
|-----------|--------|-----------------------------------------|------------------|
| `endDate` | String | The end date for filtering work periods | `2025-04-07`     |

### Headers
- `Authorization: Bearer <token>`: Authentication token for secure access.

### Response
#### Status Code
- `200 OK`: Successfully retrieved work periods.

#### Body Example
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

#### Fields Description
| Field         | Type    | Description                                  |
|---------------|---------|----------------------------------------------|
| `Id`          | Integer | Unique identifier for the work period        |
| `StartDate`   | String  | Start date of the work period (ISO format)   |
| `EndDate`     | String  | End date of the work period (ISO format)     |
| `Name`        | String  | Persian name of the work period (e.g., month)|
| `MounthName`  | String  | English name of the month                    |
| `IsCurrent`   | Boolean | Indicates if this is the current work period |
| `IsExpired`   | Boolean | Indicates if the work period has expired     |

---

## 2. GetCalendarDayItems API

### Endpoint
```
GET https://kasra.kasraco.net/lego.web/api/KasraHamrah/CalendarAPI/GetCalendarItems?workPeriodId={workPeriodId}&PersonCode={PersonCode}
```

### Description
This API retrieves daily calendar data for a specific work period and person, including the status of each day and any associated requests. This data is used to construct a calendar view in the application.

### Parameters
| Name           | Type   | Description                           | Example     |
|----------------|--------|---------------------------------------|-------------|
| `workPeriodId` | Integer| ID of the selected work period        | `183`       |
| `PersonCode`   | Integer| Unique code of the person             | `159863725` |

### Headers
- `Authorization: Bearer <token>`: Authentication token for secure access.

### Response
#### Status Code
- `200 OK`: Successfully retrieved calendar day items.

#### Body Example
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

#### Fields Description
| Field                | Type    | Description                                      |
|----------------------|---------|--------------------------------------------------|
| `Id`                 | Integer | Unique identifier for the calendar day item      |
| `Date`               | String  | Date of the calendar day (ISO format)            |
| `DayStatus`          | Integer | Status of the day (see `dayStatus` enum below)   |
| `RequestsStatus`     | Integer | Status of requests for the day (see `requestStatus` enum below) |
| `IsToday`            | Boolean | Indicates if the day is today                    |
| `IsOutOfWorkPeriod`  | Boolean | Indicates if the day is outside the work period  |
| `HaveMissedAttendance`| Boolean | Indicates if attendance was missed on this day   |

#### DayStatus Enum
The `DayStatus` field corresponds to the `dayStatus` enum defined in the application:
```java
public enum dayStatus {
    HOLIDAY_BUT_WORKING {
        @NonNull
        @Override
        public String toString() {
            return "12001";
        }
    }, WORKING {
        @NonNull
        @Override
        public String toString() {
            return "12002";
        }
    }, HOLIDAY {
        @NonNull
        @Override
        public String toString() {
            return "12003";
        }
    }, OFF {
        @NonNull
        @Override
        public String toString() {
            return "12004";
        }
    }
}
```
| Value  | Description              |
|--------|--------------------------|
| `12001`| Holiday but working day  |
| `12002`| Working day              |
| `12003`| Holiday                  |
| `12004`| Off day                  |

**Note**: The response example shows `DayStatus: 12005`, which is not defined in the provided enum. This might indicate an additional status (e.g., "Unknown" or "Other") or a mismatch that needs clarification from the backend.

#### RequestStatus Enum
The `RequestsStatus` field corresponds to the `requestStatus` enum defined in the application:
```java
public enum requestStatus {
    PENDING {
        @NonNull
        @Override
        public String toString() {
            return "201";
        }
    }, APPROVE {
        @NonNull
        @Override
        public String toString() {
            return "203";
        }
    }, DENY {
        @NonNull
        @Override
        public String toString() {
            return "204";
        }
    }, DELETE {
        @NonNull
        @Override
        public String toString() {
            return "205";
        }
    }, CANCEL {
        @NonNull
        @Override
        public String toString() {
            return "209";
        }
    }, NONE {
        @NonNull
        @Override
        public String toString() {
            return "0";
        }
    }, WITHLAW {
        @NonNull
        @Override
        public String toString() {
            return "-1";
        }
    }
}
```
| Value | Description         |
|-------|---------------------|
| `201` | Pending request     |
| `203` | Approved request    |
| `204` | Denied request      |
| `205` | Deleted request     |
| `209` | Canceled request    |
| `0`   | No request          |
| `-1`  | Withdrawn request   |

**Note**: The response example shows `RequestsStatus: 13003`, which does not match any value in the provided enum. This suggests either an extended status not covered in the enum or a potential error/typo in the response data.

### Example Usage in Code
The following Java code snippet demonstrates how `DayStatus` is used to style calendar items:
```java
if (calendarDayItems.get(position).getDayStatus().toString().equalsIgnoreCase(MyEnums.dayStatus.HOLIDAY.toString())) {
    dayNumber.setTextColor(mContext.getResources().getColor(R.color.colorGray));
    itemLinearLayout.setBackground(mContext.getResources().getDrawable(R.drawable.holiday_background));
} else if (calendarDayItems.get(position).getDayStatus().toString().equalsIgnoreCase(MyEnums.dayStatus.HOLIDAY_BUT_WORKING.toString())) {
    dayNumber.setTextColor(mContext.getResources().getColor(R.color.colorTextDarkGray));
    itemLinearLayout.setBackground(mContext.getResources().getDrawable(R.drawable.holiday_background));
} else if (calendarDayItems.get(position).getDayStatus().toString().equalsIgnoreCase(MyEnums.dayStatus.OFF.toString())) {
    dayNumber.setTextColor(mContext.getResources().getColor(R.color.colorGray));
}
```

---

## 3. GetMonthlyPerformanceAuditSummary API

### Endpoint
```
GET https://kasra.kasraco.net/lego.web/api/KasraHamrah/PerformanceAuditAPI/GetMonthlyPerformanceAuditSummary?workPeriodId={workPeriodId}&PersonCode={PersonCode}
```

### Description
This API retrieves a summary of monthly performance metrics (e.g., work hours, absences, overtime) for a specific person within a selected work period.

### Parameters
| Name           | Type   | Description                           | Example     |
|----------------|--------|---------------------------------------|-------------|
| `workPeriodId` | Integer| ID of the selected work period        | `182`       |
| `PersonCode`   | Integer| Unique code of the person             | `159863725` |

### Headers
- `Authorization: Bearer <token>`: Authentication token for secure access.

### Response
#### Status Code
- `200 OK`: Successfully retrieved performance audit summary.

#### Body Example
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
      "IsDaily": false,
      "IsRegulationTime": false,
      "IsOvertime": true,
      "IsRequestNeed": true,
      "IsSelectable": true,
      "Color": "#FFB82C"
    },
    "Value": "00:26",
    "DaysCount": 0,
    "HoursCount": 0,
    "MinutesCount": 26,
    "RequestNeedCount": 1
  }
]
```

#### Fields Description
| Field             | Type    | Description                                      |
|-------------------|---------|--------------------------------------------------|
| `Id`              | Integer | Unique identifier for the performance entry      |
| `Code`            | Object  | Details about the performance metric             |
| `Code.Id`         | Integer | Unique identifier for the metric type            |
| `Code.Name`       | String  | Name of the metric (e.g., "كسر حضور ساعتي" = Hourly Absence) |
| `Code.Nature`     | Object  | Nature of the metric (e.g., hourly or daily)     |
| `Code.IsDaily`    | Boolean | Indicates if the metric is daily-based           |
| `Code.IsRegulationTime` | Boolean | Indicates if the metric is part of regulated time |
| `Code.IsOvertime` | Boolean | Indicates if the metric represents overtime      |
| `Code.IsRequestNeed` | Boolean | Indicates if a request is needed for this metric |
| `Code.IsSelectable` | Boolean | Indicates if the metric is selectable in the UI  |
| `Code.Color`      | String  | Hex color code for UI representation             |
| `Value`           | String  | Total time in "HH:mm" format                     |
| `DaysCount`       | Integer | Number of days for this metric                   |
| `HoursCount`      | Integer | Number of hours for this metric                  |
| `MinutesCount`    | Integer | Number of minutes for this metric                |
| `RequestNeedCount`| Integer | Number of pending requests for this metric       |

---

## Error Handling
For all APIs:
- `400 Bad Request`: Invalid or missing parameters.
- `401 Unauthorized`: Missing or invalid authentication token.
- `500 Internal Server Error`: Server-side issue.

---

## Notes
- **Authentication**: All requests require a valid Bearer token in the `Authorization` header.
- **Time Format**: Time values (e.g., `Value` in `GetMonthlyPerformanceAuditSummary`) are returned in "HH:mm" format.
- **Color Codes**: The `Color` field in `GetMonthlyPerformanceAuditSummary` can be used to visually distinguish metrics in the UI.
- **Enum Mismatches**: 
  - `DayStatus: 12005` and `RequestsStatus: 13003` in `GetCalendarDayItems` responses do not match the provided enums. These values should be verified with the backend team to ensure correct mapping or to extend the enums if necessary.
- **Dependencies**: 
  - `GetCalendarDayItems` and `GetMonthlyPerformanceAuditSummary` rely on the `workPeriodId` obtained from `GetAllWorkPeriods`.
  - `PersonCode` is assumed to be available from the user context.