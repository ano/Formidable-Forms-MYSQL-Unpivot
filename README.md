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
Using an external PHP file to Generate the PIVOT SQL Query for a specific formidable form

Usage:
---
```
https://yourdomain.com/query.php?form_id=1
```
where **1** is the id of the form

Code:
---
Create **query.php** file
```php
<?php
	define( 'WP_USE_THEMES', false );
	require_once( "../" . '/wp-load.php'); //PATH to the Wordpress install directory
	if(isset($_GET['form_id'])){
	$form_id = (int) $_GET['form_id'];
	$fields = array();
	$fields = $wpdb->get_results('SELECT DISTINCT(field_key) FROM entry_data WHERE form_id = ' . $form_id);

	foreach ($fields as $field) {
	    $columns .= "\tMAX( CASE WHEN field_key = '{$field->field_key}' THEN meta_value END ) '{$field->field_key}',\n";
	}
	$query = "
	    SELECT  
	    item_id as entry_id, 
	    " . $columns . " 
	    {$form_id} as form_id
	    FROM (
	      SELECT
		`fi`.`form_id` AS `form_id`,
		`it`.`field_id` AS `field_id`,
		`it`.`item_id` AS `item_id`,
		`fi`.`field_key` AS `field_key`,
		`it`.`meta_value` AS `meta_value` 
	      FROM
		(
		  `".$wpdb->prefix."frm_item_metas` `it`
		  LEFT JOIN `".$wpdb->prefix."frm_fields` `fi` ON ((
		      `it`.`field_id` = `fi`.`id` 
		    ))) 
	      WHERE
		( `it`.`field_id` <> 0 ) and `fi`.`form_id` = ". $form_id . "
	      ORDER BY
		`fi`.`form_id`,
		`it`.`item_id`
	    ) as entry
	    WHERE form_id = {$form_id} 
	    GROUP BY item_id;
	";
	print_r($query);
	}
?>
```
