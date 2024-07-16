
## README.md

1. Active Users on Website

```bash
SELECT COUNT(DISTINCT lv.idvisitor) AS live_user_count
FROM matomo.log_visit lv
JOIN matomo.site s ON lv.idsite = s.idsite
WHERE s.name = "$website"
AND lv.visit_last_action_time >= NOW() - INTERVAL 1 MINUTE;
 
```
Purpose:
This query counts the number of distinct active users currently on the website within the last minute. It helps monitor real-time user activity.

Usage:
Used to gauge current website traffic and monitor spikes or drops in active users for immediate responsiveness.

2. Users by Platform (OS)


```bash
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
Purpose:
This query categorizes and counts website visits by different operating systems (OS) and their versions. It helps in understanding the distribution of users across various platforms.

Usage:
Helps in optimizing website design and functionality based on the most popular operating systems and versions among visitors.

3.  Users by Geolocation (Heatmap)
```bash
SELECT lv.location_latitude ,lv.location_longitude
FROM matomo.log_visit lv
JOIN matomo.site s ON lv.idsite = s.idsite
WHERE s.name = "$website";
 
```
Purpose:
This query retrieves latitude and longitude coordinates of users' geolocation based on their IP addresses.

Usage:
Visualizes user distribution geographically to understand where website visitors are located globally or regionally.

4. Users by Browser
```bash
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
Purpose:
This query categorizes and counts website visits by different web browsers and their versions. It helps in understanding the browser preferences of visitors.

Usage:
Useful for optimizing website compatibility and performance across popular browsers and versions.

5. Unique Users on Website

```bash
SELECT COUNT(DISTINCT lv.idvisitor) AS active_user_count
FROM matomo.log_visit lv
JOIN matomo.site s ON lv.idsite = s.idsite
WHERE s.name = 'SSCMS'
AND lv.visit_last_action_time BETWEEN $__timeFrom() AND $__timeTo();
 
```
Purpose:
This query counts the number of distinct users who have visited the website within a specified time range.

Usage:
Provides insights into unique visitor traffic over a period, useful for understanding reach and engagement.


6. Total Users Visited Website

```bash
SELECT SUM(lv.visitor_count_visits) AS total_visitor_on_site
FROM matomo.log_visit lv
JOIN matomo.site s ON lv.idsite = s.idsite
WHERE s.name  = "$website"  
AND lv.visit_last_action_time BETWEEN $__timeFrom() AND $__timeTo();
 
```
Purpose:
This query sums up the total number of visits to the website within a specified time range.

Usage:
Gives an overall count of visitor sessions, useful for measuring overall traffic and engagement levels.

7. Average Session Time
```bash

SELECT AVG(lv.visit_total_time) AS average_visit_time_seconds
FROM matomo.log_visit lv
JOIN matomo.site s ON lv.idsite = s.idsite
WHERE s.name  = "$website"  
AND lv.visit_last_action_time BETWEEN $__timeFrom() AND $__timeTo();
 
```
Purpose:
This query calculates the average duration of user sessions on the website.

Usage:
Helps in assessing user engagement and identifying trends in session length, crucial for optimizing content and user experience.

8. Bounce Rate
```bash

SELECT
    (SUM(CASE WHEN lv.visit_total_actions = 1 THEN 1 ELSE 0 END) / COUNT(*)) * 100 AS bounce_rate_percentage
FROM
    matomo.log_visit lv
JOIN
    matomo.site s ON lv.idsite = s.idsite
WHERE s.name  = "$website"  
AND lv.visit_last_action_time BETWEEN $__timeFrom() AND $__timeTo();
 
```

Purpose:
This query calculates the bounce rate percentage, representing the proportion of single-page visits where the visitor left without interaction.

Usage:
Key metric for evaluating website usability and content relevance. High bounce rates may indicate issues with landing pages or content.

9. Average User Actions per Session

```bash

SELECT FLOOR(AVG(lv.visit_total_interactions)) AS average_visit_interaction
FROM matomo.log_visit lv
JOIN matomo.site s ON lv.idsite = s.idsite
WHERE s.name  = "$website"  
AND lv.visit_last_action_time BETWEEN $__timeFrom() AND $__timeTo();
 
 
```
Purpose:
This query calculates the average number of interactions (actions) performed by users during their sessions on the website.

Usage:
Provides insights into user engagement levels and helps in understanding how visitors interact with the website content.






    
## Authors

- [@octokatherine](https://www.github.com/octokatherine)



1. Active Users on Website

SELECT COUNT(DISTINCT lv.idvisitor) AS live_user_count
FROM matomo.log_visit lv
JOIN matomo.site s ON lv.idsite = s.idsite
WHERE s.name = "$website"
  AND lv.visit_last_action_time >= NOW() - INTERVAL 1 MINUTE;
 

Purpose:
This query counts the number of distinct active users currently on the website within the last minute. It helps monitor real-time user activity.

Usage:
Used to gauge current website traffic and monitor spikes or drops in active users for immediate responsiveness.

2. Users by Platform (OS)

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
 
 
Purpose:
This query categorizes and counts website visits by different operating systems (OS) and their versions. It helps in understanding the distribution of users across various platforms.

Usage:
Helps in optimizing website design and functionality based on the most popular operating systems and versions among visitors.

3. Users by Geolocation (Heatmap)

SELECT lv.location_latitude ,lv.location_longitude
FROM matomo.log_visit lv
JOIN matomo.site s ON lv.idsite = s.idsite
WHERE s.name = "$website";

Purpose:
This query retrieves latitude and longitude coordinates of users' geolocation based on their IP addresses.

Usage:
Visualizes user distribution geographically to understand where website visitors are located globally or regionally.

4. Users by Browser

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

Purpose:
This query categorizes and counts website visits by different web browsers and their versions. It helps in understanding the browser preferences of visitors.

Usage:
Useful for optimizing website compatibility and performance across popular browsers and versions.

5. Unique Users on Website

SELECT COUNT(DISTINCT lv.idvisitor) AS active_user_count
FROM matomo.log_visit lv
JOIN matomo.site s ON lv.idsite = s.idsite
WHERE s.name = 'SSCMS'
  AND lv.visit_last_action_time BETWEEN $__timeFrom() AND $__timeTo();
 

Purpose:
This query counts the number of distinct users who have visited the website within a specified time range.

Usage:
Provides insights into unique visitor traffic over a period, useful for understanding reach and engagement.

6. Total Users Visited Website

SELECT SUM(lv.visitor_count_visits) AS total_visitor_on_site
FROM matomo.log_visit lv
JOIN matomo.site s ON lv.idsite = s.idsite
WHERE s.name  = "$website"  
AND lv.visit_last_action_time BETWEEN $__timeFrom() AND $__timeTo();

Purpose:
This query sums up the total number of visits to the website within a specified time range.

Usage:
Gives an overall count of visitor sessions, useful for measuring overall traffic and engagement levels.

7. Average Session Time

SELECT AVG(lv.visit_total_time) AS average_visit_time_seconds
FROM matomo.log_visit lv
JOIN matomo.site s ON lv.idsite = s.idsite
WHERE s.name  = "$website"  
AND lv.visit_last_action_time BETWEEN $__timeFrom() AND $__timeTo();

Purpose:
This query calculates the average duration of user sessions on the website.

Usage:
Helps in assessing user engagement and identifying trends in session length, crucial for optimizing content and user experience.

8. Bounce Rate

SELECT
    (SUM(CASE WHEN lv.visit_total_actions = 1 THEN 1 ELSE 0 END) / COUNT(*)) * 100 AS bounce_rate_percentage
FROM
    matomo.log_visit lv
JOIN
    matomo.site s ON lv.idsite = s.idsite
WHERE s.name  = "$website"  
AND lv.visit_last_action_time BETWEEN $__timeFrom() AND $__timeTo();

Purpose:
This query calculates the bounce rate percentage, representing the proportion of single-page visits where the visitor left without interaction.

Usage:
Key metric for evaluating website usability and content relevance. High bounce rates may indicate issues with landing pages or content.

9. Average User Actions per Session

 SELECT FLOOR(AVG(lv.visit_total_interactions)) AS average_visit_interaction
FROM matomo.log_visit lv
JOIN matomo.site s ON lv.idsite = s.idsite
WHERE s.name  = "$website"  
AND lv.visit_last_action_time BETWEEN $__timeFrom() AND $__timeTo();
 

Purpose:
This query calculates the average number of interactions (actions) performed by users during their sessions on the website.

Usage:
Provides insights into user engagement levels and helps in understanding how visitors interact with the website content.
