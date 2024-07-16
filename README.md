# Website Analytics Queries

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
