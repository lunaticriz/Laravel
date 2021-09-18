### How to use push notifications

### Steps to follow

### Create an migration for device_key

```php
php artisan make:migration add_column_device_token
```

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class AddColumnDeviceTokenToUsersTable extends Migration
{
  
    public function up()
    {
        Schema::table('users', function (Blueprint $table) {
            $table->string('device_token')->nullable();
        });
    }

    public function down()
    {
        Schema::table('users', function (Blueprint $table) {
            //
        });
    }
}
```

### Add this in app/Models/User.php

```php
<?php

namespace App\Models;

use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable
{
    use HasFactory, Notifiable;

  
    protected $fillable = [
        'name',
        'email',
        'password',
        'device_token'
    ];

  
    protected $hidden = [
        'password',
        'remember_token',
    ];

    protected $casts = [
        'email_verified_at' => 'datetime',
    ];
}
```

### Then migrate it

```php
php artisan migrate
```

### Don't forget to save device token.

* You can save/update device_token or device_key on login function.

```php
if(!empty($request->device_token)) {
    $deviceTOken = User::where('email', $request->email)->update(array('device_token' => $request->device_token));
}
```


### Create Common function in helpers and use this function wherever you want

```php
function send_notification_FCM($notification_id, $title, $message, $id, $type) 
{
 
    $accesstoken = 'use your server key here';
    $URL = 'https://fcm.googleapis.com/fcm/send';
    if(is_array($notification_id)) 
    { 
        $msg = array ('body'  => $message, 'title' => $title, 'type' => $type, 'id' => $id);
        $fields = array ('registration_ids'  => $notification_id, 'data'  => $msg, 'priority' => 'high');
        $data1 = [
             'registration_ids' => $notification_id,
             'body' => $message,
             'title' => $title,
             'type' => $type,
             'id' => $id,
        ];
    } 
    else 
    {
        $msg = array ('body'  => $message, 'title' => $title, 'type' => $type, 'id' => $id);
        $fields = array ('to'  => $notification_id, 'data'  => $msg, 'priority' => 'high');
        $data1 = [
             'to' => $notification_id,
             'body' => $message,
             'title' => $title,
             'type' => $type,
             'id' => $id,
        ];
    }
 
    $crl = curl_init();
 
    $headr = array();
    $headr[] = 'Content-type: application/json';
    $headr[] = 'Authorization: key=' . $accesstoken;
    curl_setopt($crl, CURLOPT_SSL_VERIFYPEER, false);
 
    curl_setopt($crl, CURLOPT_URL, $URL);
    curl_setopt($crl, CURLOPT_HTTPHEADER, $headr);
 
    curl_setopt($crl, CURLOPT_POST, true);
    curl_setopt($crl, CURLOPT_POSTFIELDS, json_encode($fields));
    curl_setopt($crl, CURLOPT_RETURNTRANSFER, true);
 
    $rest = curl_exec($crl);
 //return $rest;
    return response()->json(['data' => $data1], trans('messages.status'));
}


function send_notification_ios($notification_id, $title, $message, $id, $type) 
{
 
    $accesstoken = 'use your server key here';
 
    $URL = 'https://fcm.googleapis.com/fcm/send';
    if(is_array($notification_id)) 
    { 
	foreach($notification_id as $noti_id) {
        $data = '{
            "to" : "' . $noti_id . '",
            "data" : {
              "body" : "",
              "title" : "' . $title . '",
              "type" : "' . $type . '"
              "message" : "' . $message . '",
            },
            "notification" : {
                 "body" : "' . $message . '",
                 "title" : "' . $title . '",
                  "type" : "' . $type . '"
                 "message" : "' . $message . '",
                "icon" : "new",
                "sound" : "default"
                },
          }';
	}
        $data1 = [
             'registration_ids' => $notification_id,
             'body' => $message,
             'title' => $title,
             'type' => $type
        ];
    } 
    else 
    {
        $data = '{
            "to" : "' . $notification_id . '",
            "data" : {
              "body" : "",
              "title" : "' . $title . '",
              "type" : "' . $type . '",
              "id" : "' . $id . '",
              "message" : "' . $message . '",
            },
            "notification" : {
                 "body" : "' . $message . '",
                 "title" : "' . $title . '",
                  "type" : "' . $type . '",
                 "id" : "' . $id . '",
                 "message" : "' . $message . '",
                "icon" : "new",
                "sound" : "default"
                },
 
          }';
	$data1 = [
             'to' => $notification_id,
             'body' => $message,
             'title' => $title,
             'type' => $type,
             'id' => $id,
        ];
        
    }
 
    $crl = curl_init();
 
    $headr = array();
    $headr[] = 'Content-type: application/json';
    $headr[] = 'Authorization: key=' . $accesstoken;
    curl_setopt($crl, CURLOPT_SSL_VERIFYPEER, false);
 
    curl_setopt($crl, CURLOPT_URL, $URL);
    curl_setopt($crl, CURLOPT_HTTPHEADER, $headr);
 
    curl_setopt($crl, CURLOPT_POST, true);
    curl_setopt($crl, CURLOPT_POSTFIELDS, $data);
    curl_setopt($crl, CURLOPT_RETURNTRANSFER, true);
 
    $rest = curl_exec($crl);
    return response()->json(['data' => $data1], trans('messages.status'));
    
}
```

### Use it like this below in your function from which you want to send notification on mobile devices.

* For single 

```php
/** Send Push Notification on android and iOS devices */
$device_token_ios = DB::table('users')->where('id', $reciverID)->where('device_key', 'ios')->value('device_token');
$device_token = DB::table('users')->where('id', $reciverID)->where('device_key', NULL)->value('device_token');
$title = "Property Manager";
$message = $request->message_body;
$id = $reciverID;
$type = "OneToOneMessage";
if(!empty($device_token)) {
    $res = send_notification_FCM($device_token, $title, $message, $id, $type);
    } elseif(!empty($device_token_ios)) {
    $res1 = send_notification_ios($device_token_ios, $title, $message, $id, $type);
}
/** End Send Push Notification on android and iOS devices */
```

* For multiple devices

```php
/** Send push notifications to all users */
$FCMTokenDataAndroid = User::where('manager_id', $sender_id)->where('device_key', NULL)->get();
foreach ($FCMTokenDataAndroid as $value) 
{
    $device_token[] = $value->device_token;
    $id[] = $value->id;
}
$FCMTokenDataIos = User::where('manager_id', $sender_id)->where('device_key', 'ios')->get();
foreach ($FCMTokenDataIos as $value) 
{
    $device_tokenIos[] = $value->device_token;
    $id[] = $value->id;
}
$title = "Broadcast Message";
$message = $request->message_body;
$id = $request->resident_id;
$type = "broadcast";
if(!empty($device_token)) {
    $res = send_notification_FCM($device_token, $title, $message, $id, $type);
    } 
if(!empty($device_tokenIos)) {
    $res1 = send_notification_ios($device_tokenIos, $title, $message, $id, $type);
}
/** End Send push notifications to all users */
```
