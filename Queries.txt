-- Fractional retention and total joined per day --
SELECT
  joined AS Day,
  COUNT(player_id) AS total_joined,
  SUM(retained) AS total_retained,
  ROUND((SUM(retained) / COUNT(player_id)),2) AS fractional_retention

FROM ( -- created a subquery to call from to aggrigate all the columns, subquery creates a new column indicating retained(1) players and non-retained(0)
  SELECT
    p.player_id,
    p.joined,
    CASE -- used a case(if) statement to distuingish between player thats were retained and not
      WHEN (MAX(m.day) - p.joined) > 29 THEN 1
    ELSE
    0
  END
    AS retained
  FROM
    `mobile-game-project-349100.Video_Game_Data.player_info` AS p
  LEFT JOIN  -- created a left join to make sure all recorded players are accounted for
    `mobile-game-project-349100.Video_Game_Data.matches_info` AS m
  ON
    p.player_id = m.player_id
  GROUP BY
    p.player_id,
    p.joined )
GROUP BY
  joined
ORDER BY
  Day
LIMIT
  335


-- Player spending query -- 
SELECT
  r.player_id,
  ROUND(SUM(price),2) AS total_spent,
  r.retained

FROM ( -- player retained query
  SELECT
    p.player_id,
    p.joined,
    CASE
      WHEN (MAX(m.day) - p.joined) > 29 THEN 1
    ELSE
    0
  END
    AS retained
  FROM
    `mobile-game-project-349100.Video_Game_Data.player_info` AS p
  LEFT JOIN
    `mobile-game-project-349100.Video_Game_Data.matches_info` AS m
  ON
    p.player_id = m.player_id
  GROUP BY
    p.player_id,
    p.joined ) AS r

-- joined the purchase table and joined it to the item table as well as it has the item price
JOIN `mobile-game-project-349100.Video_Game_Data.purchase_info` AS pu 
ON pu.player_id = r.player_id 
JOIN `mobile-game-project-349100.Video_Game_Data.item_info` AS i 
ON i.item_id = pu.item_id
GROUP BY
  r.player_id, r.retained



-- AVG spending retained vs non-retained -- 
SELECT
  retained,
  ROUND(AVG(total_spent),2) AS avg_spent,
  ROUND(SUM(total_spent),2) AS total_spent
FROM (-- Player spending query
  SELECT
    r.player_id,
    ROUND(SUM(price),2) AS total_spent,
    r.retained
  FROM ( -- Player retained query
    SELECT
      p.player_id,
      p.joined,
      CASE
        WHEN (MAX(m.day) - p.joined) > 29 THEN 1
      ELSE
      0
    END
      AS retained
    FROM
      `mobile-game-project-349100.Video_Game_Data.player_info` AS p
    LEFT JOIN
      `mobile-game-project-349100.Video_Game_Data.matches_info` AS m
    ON
      p.player_id = m.player_id
    GROUP BY
      p.player_id,
      p.joined ) AS r
  JOIN
    `mobile-game-project-349100.Video_Game_Data.purchase_info` AS pu
  ON
    pu.player_id = r.player_id
  JOIN
    `mobile-game-project-349100.Video_Game_Data.item_info` AS i
  ON
    i.item_id = pu.item_id
  GROUP BY
    r.player_id,
    r.retained ) AS s 
GROUP BY retained


-- total spending per day --
SELECT
  pi.day,
  ROUND(SUM(ii.price),2) AS total_spent
FROM -- Joining two tables 
  `mobile-game-project-349100.Video_Game_Data.purchase_info` AS pi
JOIN
  `mobile-game-project-349100.Video_Game_Data.item_info` AS ii
ON
  pi.item_id = ii.item_id
GROUP BY
  pi.day
ORDER BY
  pi.day ASC
