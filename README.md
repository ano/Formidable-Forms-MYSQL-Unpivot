# Formidable-Forms-MYSQL-Unpivot
How to Unpivot Formidable Forms entry data in MySQL

# Step 1: Create a View called **entry_data**

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
