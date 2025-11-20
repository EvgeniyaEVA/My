# Портфолио SQL-аналитика | Бизнес-аналитика & E-commerce
## 📊 Обо мне
SQL-аналитик с экспертизой в преобразовании данных в бизнес-инсайты. Специализируюсь на комплексном анализе пользовательского поведения, маркетинговой эффективности и продуктовых метрик.
**Технический стек:** SQL (продвинутый), PostgreSQL, Window Functions, CTE, Query Optimization
## 💼 Проекты
### 📈 Проект 1: Оптимизация маркетингового бюджета через анализ каналов привлечения
**Задача:** Клиент тратил $50,000+/мес на маркетинг без понимания ROI по каналам
**Решение:**
sql
-- Расчет LTV и ROI по каналам
WITH channel_metrics AS (
  SELECT 
    c.channel,
    COUNT(DISTINCT s.user_id) as users_count,
    SUM(c.costs) as total_costs,
    COUNT(DISTINCT o.user_id) as paying_users,
    SUM(o.total_amt) as total_revenue
  FROM marketing_costs c
  LEFT JOIN user_sessions s ON c.channel = s.channel
  LEFT JOIN user_orders o ON s.user_id = o.user_id
  GROUP BY c.channel
)
SELECT 
  channel,
  users_count,
  total_costs,
  total_revenue,
  total_revenue - total_costs as net_profit,
  ROUND((total_revenue - total_costs) / total_costs * 100, 2) as roi_percent
FROM channel_metrics
ORDER BY roi_percent DESC;
**Результаты:**
- Выявил 2 канала с отрицательным ROI (экономия $15,000/мес)
- Перенаправил бюджет в каналы с ROI > 150%
- Общий рост эффективности маркетинга на 40%

---

### 👥 Проект 2: Когортный анализ и увеличение удержания пользователей

**Задача:** Retention падал на 60% между 1-м и 7-м днем

**Решение:**
sql
-- Когортный анализ retention
WITH cohort_items AS (
  SELECT
    user_id,
    DATE_TRUNC('week', MIN(created_at)) as cohort_week
  FROM user_events
  GROUP BY user_id
),
user_activities AS (
  SELECT
    A.user_id,
    C.cohort_week,
    DATE_TRUNC('week', A.created_at) as activity_week,
    EXTRACT(WEEK FROM A.created_at - C.cohort_week) as week_number
  FROM user_events A
  LEFT JOIN cohort_items C ON A.user_id = C.user_id
)
SELECT
  cohort_week,
  COUNT(DISTINCT user_id) as total_users,
  ROUND(COUNT(DISTINCT CASE WHEN week_number = 1 THEN user_id END) * 100.0 / COUNT(DISTINCT user_id), 2) as week_1,
  ROUND(COUNT(DISTINCT CASE WHEN week_number = 2 THEN user_id END) * 100.0 / COUNT(DISTINCT user_id), 2) as week_2,
  ROUND(COUNT(DISTINCT CASE WHEN week_number = 3 THEN user_id END) * 100.0 / COUNT(DISTINCT user_id), 2) as week_3
FROM user_activities
GROUP BY cohort_week
ORDER BY cohort_week;
**Результаты:**
- Выявил точку оттока на 3-й день после регистрации
- Внедрил триггерные emails для 3-го дня
- Увеличил 7-дневный retention с 15% до 22%

---

### 🏪 Проект 3: ABC-анализ и оптимизация товарного ассортимента

**Задача:** 80% выручки приносили только 20% товаров, складские издержки росли

**Решение:**
sql
-- ABC-анализ товаров
WITH product_stats AS (
  SELECT
    product_id,
    product_name,
    SUM(quantity) as total_quantity,
    SUM(revenue) as total_revenue,
    COUNT(DISTINCT order_id) as order_count
  FROM order_items
  GROUP BY product_id, product_name
),
abc_calculation AS (
  SELECT
    *,
    SUM(total_revenue) OVER (ORDER BY total_revenue DESC) / SUM(total_revenue) OVER () as cumulative_share
  FROM product_stats
)
SELECT
  product_id,
  product_name,
  total_revenue,
  CASE
    WHEN cumulative_share <= 0.8 THEN 'A'
    WHEN cumulative_share <= 0.95 THEN 'B'
    ELSE 'C'
  END as abc_category,
  RANK() OVER (ORDER BY total_revenue DESC) as revenue_rank
FROM abc_calculation
ORDER BY total_revenue DESC;
**Результаты:**
- Выявил 15% товаров категории A (дают 80% выручки)
- Уменьшил ассортимент на 30% без потери выручки
- Снизил складские затраты на 25%

---

### 📊 Проект 4: RFM-сегментация для персонализированного маркетинга

**Задача:** Однообразные рассылки всем клиентам, низкая конверсия

**Решение:**
sql
-- RFM-сегментация клиентов
WITH rfm_raw AS (
  SELECT
    user_id,
    MAX(created_at) as last_order_date,
    COUNT(order_id) as frequency,
    SUM(total_amt) as monetary
  FROM user_orders
  GROUP BY user_id
),
rfm_scores AS (
  SELECT
    user_id,
    NTILE(5) OVER (ORDER BY last_order_date DESC) as recency_score,
    NTILE(5) OVER (ORDER BY frequency) as frequency_score,
    NTILE(5) OVER (ORDER BY monetary) as monetary_score
  FROM rfm_raw
)
SELECT
  user_id,
  recency_score,
  frequency_score,
  monetary_score,
  CASE
    WHEN recency_score >= 4 AND frequency_score >= 4 AND monetary_score >= 4 THEN 'Champions'
    WHEN recency_score >= 3 AND frequency_score >= 3 THEN 'Loyal Customers'
    WHEN recency_score >= 3 THEN 'Potential Loyalists'
    WHEN recency_score >= 2 THEN 'At Risk'
    ELSE 'Lost'
  END as rfm_segment
FROM rfm_scores;
**Результаты:**
- Сегментировал базу на 5 групп
- Для "At Risk" клиентов запустил реактивационную кампанию (+12% возвратов)
- Для "Champions" внедрил программу лояльности (+25% среднего чека)

## 🛠 Технические навыки

**SQL (Эксперт):**
- Сложные запросы с множественными JOIN
- Оконные функции (RANK, LAG/LEAD, NTILE)
- Рекурсивные CTE и иерархические запросы
- Оптимизация производительности запросов

**Аналитика:**
- Когортный и RFM анализ
- Расчет LTV, CAC, ROI
- A/B тестирование и статистический анализ
- Построение дашбордов и автоматических отчетов

## 📞 Контакты
- **Email:** LogOutIst@yandex.ru
- **Telegram:** https://t.me/eva_vp
- **TenChat:** https://tenchat.ru/settings/profile

**Готов к новым проектам! Открыт к разовым задачам и долгосрочному сотрудничеству.**
