
## 📌 Overview

Geo Search is a spatial query technique that allows filtering geographic data based on location. One common method is using a **bounding box**, which is a rectangular area defined by two corner points: the bottom-left and top-right (latitude and longitude).

This guide demonstrates how to:

1. Set up a PostGIS-enabled table for storing geolocation points.

2. Insert sample data.

3. Perform a spatial query to find all points within a bounding box.


---

## 🧱 1. Table Creation

Create a table named `places` that stores names and geographic coordinates:

```sql
CREATE TABLE places (
    id SERIAL PRIMARY KEY,
    name TEXT,
    location GEOGRAPHY(Point, 4326) -- SRID 4326: WGS 84 (Lat/Lon)
);
```

> 🔍 Note: You need the PostGIS extension installed. If not enabled, run:
> 
> ```sql
> CREATE EXTENSION postgis;
> ```

---

## 🗺 2. Insert Sample Data

Add some sample places with latitude and longitude:

```sql
INSERT INTO places (name, location) VALUES
('Tehran Cafe',      ST_SetSRID(ST_MakePoint(51.4, 35.7), 4326)),
('Central Restaurant', ST_SetSRID(ST_MakePoint(51.6, 35.75), 4326)),
('City Library',     ST_SetSRID(ST_MakePoint(51.45, 35.65), 4326)),
('Public Park',      ST_SetSRID(ST_MakePoint(51.35, 35.72), 4326)),
('Far Mall',         ST_SetSRID(ST_MakePoint(51.9, 36.0), 4326));
```

Each `ST_MakePoint(longitude, latitude)` creates a geographic point. Then `ST_SetSRID(..., 4326)` assigns the standard SRID.

---

## 🔎 3. Bounding Box Search

Now, let's say we want to search for all locations within a bounding box defined by:

- Bottom-left corner: **longitude = 51.3, latitude = 35.6**

- Top-right corner: **longitude = 51.6, latitude = 35.8**


### 🧮 Query:

```sql
SELECT *
FROM places
WHERE location && ST_MakeEnvelope(51.3, 35.6, 51.6, 35.8, 4326);
```

### Explanation:

|Function|Description|
|---|---|
|`ST_MakeEnvelope(xmin, ymin, xmax, ymax, SRID)`|Creates a rectangular bounding box|
|`&&`|Checks if the geography object overlaps with the bounding box|

> ⚠️ Longitude comes before latitude in all PostGIS geometry functions.

---

## 🖼 Example Visualization

![Geo Search Box Diagram](https://chatgpt.com/mnt/data/A_digital_vector_illustration_demonstrates_%22Geo_Se.png)

The rectangle (bounding box) filters only the points located inside its area. The red dots represent sample data, and the blue box represents the bounding box used in the query.

---

## ✅ Output

Based on the data, the following places are inside the bounding box:

- **Tehran Cafe**

- **City Library**

- **Public Park**


---

## 🛠 Additional Tips

- To use **dynamic bounding box values** (e.g., from an API or frontend), you can bind query parameters.

- For complex shapes (like polygons), use `ST_Contains` or `ST_Within` instead.

- Use `GEOGRAPHY` for accurate distance-based queries, or `GEOMETRY` if you don’t need spherical calculations.
