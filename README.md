# Formidable-Forms-MYSQL-Unpivot
Here is how to Unpivot the data that is entered into Formidable Forms in MySQL

# Step 1: Create a View called entry_data

```sql
CREATE OR REPLACE VIEW `entry_data` AS
  SELECT
    `fi`.`form_id` AS `form_id`,
    `it`.`field_id` AS `field_id`,
    `it`.`item_id` AS `item_id`,
    `fi`.`field_key` AS `field_key`,
    `it`.`meta_value` AS `meta_value` 
  FROM
    (
      `ilj_frm_item_metas` `it`
      LEFT JOIN `ilj_frm_fields` `fi` ON ((
          `it`.`field_id` = `fi`.`id` 
        ))) 
  WHERE
    ( `it`.`field_id` <> 0 ) 
  ORDER BY
    `fi`.`form_id`,
    `it`.`item_id`

```

# Step 2: Unpivot the entry_data view

```sql
SET @sql = NULL;
SET @form_id = 1;
SELECT
  GROUP_CONCAT(DISTINCT
    CONCAT(
      'max(case when field_key = ''',
      field_key,
      ''' then meta_value end) ',
      field_key
    )
  ) INTO @sql 
FROM
  entry_data
WHERE form_id =  @form_id;
SET @sql = CONCAT('
		SELECT item_id, ', @sql, ' 
		FROM entry_data 
		WHERE form_id = ', @form_id, '
		GROUP BY item_id
	');

PREPARE stmt FROM @sql;
EXECUTE stmt;
DEALLOCATE PREPARE stmt;
```
