# Pixble API Documentation
---


## Scenario 1
Operations Team would like to know what are the top 3 performing couriers in last 3 months. Performance is calculated based on the number of shipments.

**Result:**
```
[
  {
    "f0_": "uxpx - firxt claxx",
    "easyship_courier_id": "1d7f63cedbfbe2942085b4b755d4be27",
    "total_shipment": "113"
  },
  {
    "f0_": "uxpx - flat rate priority mail - xmaenter code herell",
    "easyship_courier_id": "28f627345fa3baae4c8715002666cad5",
    "total_shipment": "17"
  },
  {
    "f0_": "global poxt - economy",
    "easyship_courier_id": "1116cff307c5ff73dc139d9bb63ae809",
    "total_shipment": "8"
  }
]
```
**Code:**
```
SELECT
	max(courier.name), 
	courier.easyship_courier_id, 
	count (shipment.easyship_courier_id) total_shipment
FROM
	easyship-bi-test.data.es_shipments shipment
LEFT JOIN easyship-bi-test.data.es_couriers courier ON
	shipment.easyship_courier_id=courier.easyship_courier_id
WHERE
	shipment.label_paid_at> TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 90 DAY)
GROUP BY
	courier.easyship_courier_id
ORDER BY
	count (shipment.easyship_courier_id) DESC
LIMIT 3;
```


## Scenario 2
The pricing Team will be adjusting margin based on shipment volume. Could you help them identify what was the % growth in terms of shipment last week compared to a week before? They would only like to see the result for these couriers umbrella names (FedEx and USPS). Week starts on Monday and ends on Sunday

**Sample result:**
```
[
  {
    "courier_name": "uxpx",
    "growth_percentage": "-62.06896551724138",
    "start_week": "2021-08-09 00:00:00 UTC",
    "end_week": "2021-08-16 00:00:00 UTC"
  },
  {
    "courier_name": "uxpx",
    "growth_percentage": "107.14285714285714",
    "start_week": "2021-08-02 00:00:00 UTC",
    "end_week": "2021-08-09 00:00:00 UTC"
  },
  {
    "courier_name": "uxpx",
    "growth_percentage": "-75.0",
    "start_week": "2021-07-26 00:00:00 UTC",
    "end_week": "2021-08-02 00:00:00 UTC"
  },
  {
    "courier_name": "uxpx",
    "growth_percentage": "1300.0",
    "start_week": "2021-07-19 00:00:00 UTC",
    "end_week": "2021-07-26 00:00:00 UTC"
  }
  ....
```
**Code:**
```
SELECT 
    a_week.courier_name ,
    ((b_week.total_shipment - a_week.total_shipment)/a_week.total_shipment)*100 AS growth_percentage, 
    a_week.start_week AS start_week, 
    b_week.start_week AS end_week
FROM
    (
        SELECT 
            courier.umbrella_name AS courier_name, count(*) AS total_shipment, max(TIMESTAMP_TRUNC(shipment.label_paid_at, WEEK(MONDAY))) as start_week
        FROM 
            easyship-bi-test.data.es_shipments shipment
        LEFT JOIN easyship-bi-test.data.es_couriers courier ON
            shipment.easyship_courier_id=courier.easyship_courier_id
        GROUP BY
            courier.umbrella_name, TIMESTAMP_TRUNC(shipment.label_paid_at, WEEK(MONDAY))
        ORDER BY
            start_week desc
    ) a_week
LEFT JOIN 
(
        SELECT 
            courier.umbrella_name AS courier_name, count(*) AS total_shipment, max(TIMESTAMP_TRUNC(shipment.label_paid_at, WEEK(MONDAY))) as start_week
        FROM 
            easyship-bi-test.data.es_shipments shipment
        LEFT JOIN easyship-bi-test.data.es_couriers courier ON
            shipment.easyship_courier_id=courier.easyship_courier_id
        GROUP BY
            courier.umbrella_name, TIMESTAMP_TRUNC(shipment.label_paid_at, WEEK(MONDAY))
        ORDER BY
            start_week desc
    ) b_week
ON
    a_week.start_week=TIMESTAMP_SUB(b_week.start_week, INTERVAL 7 DAY) AND a_week.courier_name=b_week.courier_name
WHERE
    a_week.courier_name is not null and a_week.start_week is not null and b_week.start_week is not null
ORDER BY 
    a_week.start_week DESC
```

## Scenario 3
Customer Success Team would like to know the average delivery times broken down by month and courier name for the last 3 months. Delivery time starts when a shipment has first status record event within (In Transit to Customer, In Transit to Consolidation Center) and Ends when there is first status record event within (Delivered, Out for Delivery, Failed Delivery Attempt, Delivery Expected (End of Updates))

**Sample result:**

```
[
  {
    "f0_": "uxpx - firxt claxx",
    "day_diff": "1.055214723926381",
    "month": "2021-08-01 00:00:00 UTC"
  },
  {
    "f0_": "uxpx - firxt claxx",
    "day_diff": "1.3640552995391706",
    "month": "2021-07-01 00:00:00 UTC"
  },
  {
    "f0_": "uxpx - firxt claxx",
    "day_diff": "1.75",
    "month": "2021-06-01 00:00:00 UTC"
  },
  {
    "f0_": "uxpx - flat rate priority mail - large",
    "day_diff": "0.0",
    "month": "2021-07-01 00:00:00 UTC"
  },
  ...
```
**Code:**
```
SELECT
	max(courier.name), 
	AVG(TIMESTAMP_DIFF(end_shipment.status_date, start_shipment.status_date, DAY)) AS day_diff, 
	max(TIMESTAMP_TRUNC(start_shipment.status_date, MONTH)) AS month
FROM
	(
		SELECT 
            shipment.easyship_shipment_id AS id, shipment.easyship_courier_id as courier_id, status.created_at as status_date, message.name
        FROM
            easyship-bi-test.data.es_shipments shipment
        LEFT JOIN easyship-bi-test.data.status_records status ON
            shipment.easyship_shipment_id=status.easyship_shipment_id
        LEFT JOIN easyship-bi-test.data.status_messages message ON
            status.status_message_id=message.status_message_id
        WHERE
            message.name IN ("In Transit to Customer", "In Transit to Consolidation Center") AND 
            shipment.label_paid_at >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 90 DAY)			
	) AS start_shipment,
	(
		SELECT 
            shipment.easyship_shipment_id AS id, shipment.easyship_courier_id as courier_id, status.created_at as status_date, message.name
        FROM
            easyship-bi-test.data.es_shipments shipment
        LEFT JOIN easyship-bi-test.data.status_records status ON
            shipment.easyship_shipment_id=status.easyship_shipment_id
        LEFT JOIN easyship-bi-test.data.status_messages message ON
            status.status_message_id=message.status_message_id
        WHERE
            message.name IN ("Delivered", "Out for Delivery", "Failed Delivery Attempt", "Delivery Expected (End of Updates)") AND 
            shipment.label_paid_at >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 90 DAY)		
	) AS end_shipment

LEFT JOIN
	easyship-bi-test.data.es_couriers courier ON
		courier.easyship_courier_id=end_shipment.courier_id

WHERE 
    start_shipment.id=end_shipment.id
GROUP BY 
    TIMESTAMP_TRUNC(start_shipment.status_date, MONTH), courier.name
ORDER BY
    courier.name, month DESC
```

## Scenario 4
Business Team would like to know which couriers are used most on each day of the week. So the output will look like
|day_of_week|courier_name|
|----------|-------------|
|Monday|FedEx - International Priority|
|Tuesday|DHL - Express Worldwide|
|Wednesday|USPS - First Class|

**Sample result:**
```
[
  {
    "day_of_week": "Wednesday",
    "courier_name": "uxpx - firxt claxx"
  },
  {
    "day_of_week": "Wednesday",
    "courier_name": "qxprexx - domextic"
  },
  {
    "day_of_week": "Wednesday",
    "courier_name": "uxpx - priority mail"
  },
  {
    "day_of_week": "Thursday",
    "courier_name": "fedex international economy®"
  }
  ...
```
**Code:**
```
SELECT 
    agg.day_of_week AS day_of_week,
    agg.courier_name AS courier_name
FROM
(
    SELECT 
        MAX(day_of_week_agg.num),
        MAX(CASE day_of_week_agg.day_of_week
            WHEN 1 THEN 'Sunday'
            WHEN 2 THEN 'Monday'
            WHEN 3 THEN 'Tuesday'
            WHEN 4 THEN 'Wednesday'
            WHEN 5 THEN 'Thursday'
            WHEN 6 THEN 'Friday'
            WHEN 7 THEN 'Saturday'
        END) as day_of_week,
        MAX(day_of_week_agg.courier_name) courier_name 
    FROM
        (
        SELECT
            max(EXTRACT(DAYOFWEEK FROM shipment.label_paid_at)) as day_of_week, MAX(courier.name) AS courier_name, count(*) as num
        FROM
            easyship-bi-test.data.es_shipments shipment
        LEFT JOIN easyship-bi-test.data.es_couriers courier ON
            courier.easyship_courier_id=shipment.easyship_courier_id
        GROUP BY
            courier.name, EXTRACT(DAYOFWEEK FROM shipment.label_paid_at)
        ORDER BY 
            count(*) DESC
        ) day_of_week_agg
    GROUP BY 
        day_of_week_agg.courier_name
    ORDER BY 
        MAX(num) DESC
) agg

WHERE
    courier_name is not null and day_of_week is not null
```

## Scenario 5
Marketing Team would like to know, on average how much time does it take for a client to get activated. Activation Time is calculated from onboarding to the first shipment made

> Note: I tried to join the ``es_shipments`` and ``es_clients`` tables and group by ``easyship_client_id`` to calculate the average date difference between the ``onboarding_date`` and the ``label_paid_at`` columns. However all the matching rows have either ``onboarding_date`` or ``label_paid_at`` information missing, unfortunately there is no result returns.

**Sample result:**
```
[]
```
**Code:**
```
SELECT
    clients.easyship_client_id, AVG(DATE_DIFF(clients.onboarding_date, EXTRACT(DATETIME FROM shipment.label_paid_at),DAY)) AS day_diff
FROM
    easyship-bi-test.data.es_shipments shipment

LEFT JOIN easyship-bi-test.data.es_clients clients ON

	shipment.easyship_client_id=clients.easyship_client_id

GROUP BY
	
	clients.easyship_client_id
```







