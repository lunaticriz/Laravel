### Create zip file from putty

zip -r theautomatedweb.zip theautomatedweb  *

### Command to export and import using putty

mysqldump -root -p Automated_group > Automated_group.sql --------------------Export

mysqldump -root -p Automated_group < Automated_group.sql --------------------Import





==================================================================================== Start Export in csv file =================================================================================================
### Export csv using core php

```php
<?php
$fileName = 'export.csv';
$headers = array(
	"Content-type"        => "text/csv",
	"Content-Disposition" => "attachment; filename=$fileName",
	"Pragma"              => "no-cache",
	"Cache-Control"       => "must-revalidate, post-check=0, pre-check=0",
	"Expires"             => "0"
);

$columns = array('First Name', 'Last Name', 'User Role', 'Id', 'Building Id', 'Role Id');

$callback = function() use($userdeatils, $columns) {
	$file = fopen('php://output', 'w');
	fputcsv($file, $columns);

	foreach ($userdeatils as $task) {
		$row['firstname']  = $task->firstname;
		$row['lastname']    = $task->lastname;
		$row['userrole']    = $task->userrole;
		$row['id']  = $task->id;
		$row['building_id']  = $task->building_id;
		$row['roleid']  = $task->roleid;

		fputcsv($file, array($row['firstname'], $row['lastname'], $row['userrole'], $row['id'], $row['building_id'], $row['roleid']));
	}

	fclose($file);
};

return Response::stream($callback, 200, $headers);
?>
```
==================================================================================== End Export in csv file =================================================================================================

