# Website Analytics Queries

- [Live User Counts (Updated Every Minute)](#live-user-counts-updated-every-minute)  <!-- Updated link position -->
- [Active Users on Website](#active-users-on-website)
- [Users by Platform (OS)](#users-by-platform-os)
- [Users by Geolocation (Heatmap)](#users-by-geolocation-heatmap)
- [Users by Browser](#users-by-browser)
- [Unique Users on Website](#unique-users-on-website)
- [Total Users Visited Website](#total-users-visited-website)
- [Average Session Time](#average-session-time)
- [Bounce Rate](#bounce-rate)
- [Average User Actions per Session](#average-user-actions-per-session)

---

## Live User Counts (Updated Every Minute)

Purpose:
This section shows the live user counts updated every minute using an automated event in the database.

```sql
CREATE TABLE IF NOT EXISTS matomo.live_user_counts (
    name VARCHAR(255),
    time DATETIME,
    live_user_count INT,
    PRIMARY KEY (name, time)
);
```

```sql

-- Drop the event if it already exists
DROP EVENT IF EXISTS update_live_user_counts;

DELIMITER $$

-- Create the event to update live_user_counts table every minute
CREATE EVENT update_live_user_counts
ON SCHEDULE EVERY 1 MINUTE
DO
BEGIN
    INSERT INTO matomo.live_user_counts (name, time, live_user_count)
    SELECT
        s.name,
        DATE_FORMAT(lv.visit_last_action_time, '%Y-%m-%d %H:%i:00') AS time,
        COUNT(DISTINCT lv.idvisitor) AS live_user_count
    FROM
        matomo.log_visit lv
    JOIN 
        matomo.site s ON lv.idsite = s.idsite
    WHERE 
        lv.visit_last_action_time >= NOW() - INTERVAL 24 HOUR
        AND NOT EXISTS (
            SELECT 1
            FROM matomo.live_user_counts luc
            WHERE luc.name = s.name
              AND luc.time = DATE_FORMAT(lv.visit_last_action_time, '%Y-%m-%d %H:%i:00')
        )
    GROUP BY 
        time, s.name
    ORDER BY 
        time
    ON DUPLICATE KEY UPDATE
        live_user_count = VALUES(live_user_count);
END$$

DELIMITER ;


```
## RUM Report

```sql
CREATE TABLE IF NOT EXISTS rum_report (
    date DATE,
    name VARCHAR(255),
    unique_user_count INT,
    total_visitor_count INT,
    bounce_rate DECIMAL(10, 2),
    average_visit_time_seconds DECIMAL(10, 2),
    average_visit_interaction DECIMAL(10, 2),
    PRIMARY KEY (date, name)
);
```
```sql
DELIMITER $$

-- Enable event scheduling if it's not already enabled
SET GLOBAL event_scheduler = ON$$

-- Create the event for daily aggregation if it doesn't exist
CREATE EVENT IF NOT EXISTS daily_aggregation_event
ON SCHEDULE EVERY 1 DAY
STARTS CURRENT_DATE + INTERVAL 1 DAY
DO
BEGIN
    -- Perform the daily aggregation into rum_report table
    INSERT INTO rum_report (date, name, unique_user_count, total_visitor_count, bounce_rate, average_visit_time_seconds, average_visit_interaction)
    SELECT 
        DATE(lv.visit_last_action_time) AS date,
        s.name AS name,
        COUNT(DISTINCT lv.idvisitor) AS unique_user_count,
        SUM(lv.visitor_count_visits) AS total_visitor_count,
        (SUM(CASE WHEN lv.visit_total_actions = 1 THEN 1 ELSE 0 END) / COUNT(*)) * 100 AS bounce_rate,
        AVG(lv.visit_total_time) AS average_visit_time_seconds,
        FLOOR(AVG(lv.visit_total_interactions)) AS average_visit_interaction
    FROM 
        matomo.log_visit lv
    JOIN 
        matomo.site s ON lv.idsite = s.idsite
    WHERE 
        lv.visit_last_action_time >= NOW() - INTERVAL 1 DAY
    GROUP BY 
        date, name;
END$$

DELIMITER ;

```
---
## Active Users on Website

Purpose:
This query counts the number of distinct active users currently on the website within the last minute. It helps monitor real-time user activity.

```sql
SELECT COUNT(DISTINCT lv.idvisitor) AS live_user_count
FROM matomo.log_visit lv
JOIN matomo.site s ON lv.idsite = s.idsite
WHERE s.name = "$website"
AND lv.visit_last_action_time >= NOW() - INTERVAL 1 MINUTE;
```

---

## Users by Platform (OS)

Purpose:
This query categorizes and counts website visits by different operating systems (OS) and their versions. It helps in understanding the distribution of users across various platforms.

```sql
SELECT
    lv.config_os AS os_name,
    lv.config_os_version AS os_version,
    COUNT(*) AS visit_count
FROM
    matomo.log_visit lv
JOIN
    matomo.site s ON lv.idsite = s.idsite
WHERE
    s.name = "$website"
    AND lv.visit_last_action_time BETWEEN $__timeFrom() AND $__timeTo()
GROUP BY
    lv.config_os, lv.config_os_version
ORDER BY
    visit_count DESC;

 
```

---
## Users by Geolocation (Heatmap)

Purpose:
This query retrieves latitude and longitude coordinates of users' geolocation based on their IP addresses.

```sql
SELECT lv.location_latitude, lv.location_longitude
FROM matomo.log_visit lv
JOIN matomo.site s ON lv.idsite = s.idsite
WHERE s.name = "$website";

```

---
## Users by Browser

Purpose:
This query categorizes and counts website visits by different web browsers and their versions. It helps in understanding the browser preferences of visitors.

```sql
SELECT
    lv.config_browser_name AS browser_name,
    lv.config_browser_version AS browser_version,
    COUNT(*) AS visit_count
FROM
    matomo.log_visit lv
JOIN
    matomo.site s ON lv.idsite = s.idsite
WHERE
    s.name = "$website"
    AND lv.visit_last_action_time BETWEEN $__timeFrom() AND $__timeTo()
GROUP BY
    lv.config_browser_name, lv.config_browser_version
ORDER BY
    visit_count DESC;

```

---
## Unique Users on Website

Purpose:
This query counts the number of distinct users who have visited the website within a specified time range. It provides insights into unique visitor traffic.

```sql
SELECT COUNT(DISTINCT lv.idvisitor) AS active_user_count
FROM matomo.log_visit lv
JOIN matomo.site s ON lv.idsite = s.idsite
WHERE s.name = 'SSCMS'
AND lv.visit_last_action_time BETWEEN $__timeFrom() AND $__timeTo();

```

---
## Total Users Visited Website

Purpose:
This query sums up the total number of visits to the website within a specified time range. It measures overall visitor sessions.

```sql
SELECT SUM(lv.visitor_count_visits) AS total_visitor_on_site
FROM matomo.log_visit lv
JOIN matomo.site s ON lv.idsite = s.idsite
WHERE s.name = "$website"
AND lv.visit_last_action_time BETWEEN $__timeFrom() AND $__timeTo();

```

---

## Average Session Time

Purpose:
This query calculates the average duration of user sessions on the website. It helps assess user engagement.
```sql
SELECT AVG(lv.visit_total_time) AS average_visit_time_seconds
FROM matomo.log_visit lv
JOIN matomo.site s ON lv.idsite = s.idsite
WHERE s.name = "$website"
AND lv.visit_last_action_time BETWEEN $__timeFrom() AND $__timeTo();

```

---

## Bounce Rate

Purpose:
This query calculates the bounce rate percentage, representing the proportion of single-page visits where the visitor left without interaction.



```sql
SELECT
    (SUM(CASE WHEN lv.visit_total_actions = 1 THEN 1 ELSE 0 END) / COUNT(*)) * 100 AS bounce_rate_percentage
FROM
    matomo.log_visit lv
JOIN
    matomo.site s ON lv.idsite = s.idsite
WHERE s.name = "$website"
AND lv.visit_last_action_time BETWEEN $__timeFrom() AND $__timeTo();

```

---

## Average User Actions per Session

Purpose:
This query calculates the average number of interactions (actions) performed by users during their sessions on the website.

```sql
SELECT FLOOR(AVG(lv.visit_total_interactions)) AS average_visit_interaction
FROM matomo.log_visit lv
JOIN matomo.site s ON lv.idsite = s.idsite
WHERE s.name = "$website"
AND lv.visit_last_action_time BETWEEN $__timeFrom() AND $__timeTo();

```

---
