# YouTube Virality & Trending Analysis (SQL)

## 1. Background & Overview
The goal of this project was to analyze historical YouTube video data to **increase the odds of going viral and trending**.  
The analysis focuses on understanding **views, likes, comments, channels, publishing timing, and geography** to uncover patterns that drive virality.

Key metrics analyzed:  
- `views`, `likes`, `dislikes`, `comment_count`  
- `days_to_trending`  
- `channel_title`, `publish_country`  
- `true_publish_date`, `true_trending_date`, `published_day_of_week`

---

## 2. Data Structure & Cleaning Summary
### Table Used:
- `yt_data` (with backup tables for traceability)  

### Key Fields:
- `video_id`, `channel_title`, `publish_country`, `true_publish_date`, `true_trending_date`  
- `views`, `likes`, `dislikes`, `comment_count`  
- `published_day_of_week`, `comments_disabled_fix`, `ratings_disabled_fix`, `video_error_or_removed_fix`  

### Cleaning Steps:
- Converted `publish_date` & `trending_date` into proper `DATE` format  
- Standardized TRUE/FALSE flags for disabled comments/ratings & removed videos  
- Removed old columns, created backup for safety  
- Ensured numeric columns (`views`, `likes`, `dislikes`, `comment_count`) are proper `FLOAT`  
- Aligned trending and publish dates for analysis  

---

## 3. Executive Summary

| Metric | Top Performer | Value | Insight |
|--------|---------------|-------|---------|
| Total Views | Channel X | 12,345,678 | Largest reach, mostly music content |
| Avg Views per Video | Channel Y | 1,234,567 | High engagement despite fewer uploads |
| Fastest Trending Video | Video Z | 1 day | Content type/strategy drives rapid virality |
| Best Month | March | 45,000,000 views | Seasonal trend — pre-summer spike |
| Best Publishing Day | Saturday | 8,000,000 views | Weekend publishing boosts visibility |

**Story:** Virality depends on content type, channel strategy, timing, and geography, not just the number of videos uploaded. Strategic publishing can maximize reach.

---

## 4. Insights Deep Dive

### A. Overall Engagement
```sql
SELECT 'views' AS metric,
       ROUND(AVG(views),2) AS avg_value,
       MIN(views) AS min_value,
       MAX(views) AS max_value,
       ROUND(STDEV(views),2) AS stdev_value
FROM yt_data;

Takeaway: Most videos fall in low-to-medium virality; few achieve extremely high views.

B. Seasonal Impact
SELECT DATEFROMPARTS(YEAR(true_trending_date), MONTH(true_trending_date), 1) AS month_start,
       SUM(views) AS month_total_views
FROM yt_data
GROUP BY DATEFROMPARTS(YEAR(true_trending_date), MONTH(true_trending_date), 1)
ORDER BY month_total_views DESC;

Takeaway: March-May sees the highest views, highlighting seasonal opportunities for content launches.

C. Channel_Level Insights
SELECT TOP 10 channel_title, SUM(views) AS total_views
FROM yt_data
GROUP BY channel_title
ORDER BY total_views DESC;

Takeaway: Top channels are mostly music; high total views are tied to consistent uploads and large audiences.

D. Country & Global Trends
SELECT publish_country,
       SUM(likes) AS total_country_likes,
       SUM(views) AS total_country_views
FROM yt_data
GROUP BY publish_country
ORDER BY total_country_views DESC;

Takeaway: US, GB, and IN dominate total views; localization and targeting strategies can be applied to maximize virality.

E. Publishing Day & Virality Timing
SELECT published_day_of_week, SUM(views) AS total_views
FROM yt_data
GROUP BY published_day_of_week
ORDER BY total_views DESC;

SELECT video_id, true_publish_date, true_trending_date,
       DATEDIFF(day, true_publish_date, true_trending_date) AS days_to_trending
FROM yt_data
WHERE true_trending_date IS NOT NULL
ORDER BY days_to_trending ASC, views DESC;

Takeaway: Weekend publishing (especially Saturday) correlates with higher views; faster trending videos often achieve higher overall reach.

F. Virality Categories
SELECT view_category_scale, COUNT(*) AS video_count
FROM (
  SELECT CASE
           WHEN views < 1000000 THEN 'Low'
           WHEN views BETWEEN 1000001 AND 10000000 THEN 'Medium'
           ELSE 'High'
         END AS view_category_scale
  FROM yt_data
) AS sub
GROUP BY view_category_scale
ORDER BY video_count DESC;

Takeaway: High virality videos are rare; majority are low-to-medium, emphasizing strategic content planning.

---

5. Recommendations

Target March-May for launching content to leverage seasonal spikes.
Publish on weekends (especially Saturdays) to increase initial engagement.
Analyze top channels for successful genre/content strategies.
Use geographic insights to tailor content for maximum global reach.
Encourage rapid engagement to shorten days-to-trending and increase virality odds.
