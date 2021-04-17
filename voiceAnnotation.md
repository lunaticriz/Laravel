### Text to speech in laravel
#### First signup in Voice RSS and get API Key and add this key in env file
* In env file
 ```php
 VOICE_RSS_API_KEY=f2146e0f555149da830ad238a50996d4
 ```
* Add a folder in App named Library, in this librabry folder create file named VoiceRSS.php and add this below code.
```php
<?php

namespace App\Library;

class VoiceRSS
{
	public function speech($settings) {
	    $this->_validate($settings);
	    return $this->_request($settings);
	}

	private function _validate($settings) {
	    if (!isset($settings) || count($settings) == 0) throw new Exception('The settings are undefined');
        if (!isset($settings['key']) || empty($settings['key'])) throw new Exception('The API key is undefined');
        if (!isset($settings['src']) || empty($settings['src'])) throw new Exception('The text is undefined');
        if (!isset($settings['hl']) || empty($settings['hl'])) throw new Exception('The language is undefined');
	}

	private function _request($settings) {
    	$url = ((isset($settings['ssl']) && $settings['ssl']) ? 'https' : 'http') . '://api.voicerss.org/';
    	$ch = curl_init($url);
		
		curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
		curl_setopt($ch, CURLOPT_BINARYTRANSFER, (isset($settings['b64']) && $settings['b64']) ? 0 : 1);
		curl_setopt($ch, CURLOPT_HTTPHEADER, array('Content-Type: application/x-www-form-urlencoded; charset=UTF-8'));
		curl_setopt($ch, CURLOPT_POST, 1);
		curl_setopt($ch, CURLOPT_POSTFIELDS, $this->_buildRequest($settings));

		$resp = curl_exec($ch);

		curl_close($ch);
		
		$is_error = strpos($resp, 'ERROR') === 0;
	    
    	return array(
    		'error' => ($is_error) ? $resp : null,
    		'response' => (!$is_error) ? $resp: null);
	}

	private function _buildRequest($settings) {
	    return http_build_query(array(
	        'key' => isset($settings['key']) ? $settings['key'] : '',
	        'src' => isset($settings['src']) ? $settings['src'] : '',
	        'hl' => isset($settings['hl']) ? $settings['hl'] : '',
	        'v' => isset($settings['v']) ? $settings['v'] : '',
	        'r' => isset($settings['r']) ? $settings['r'] : '',
	        'c' => isset($settings['c']) ? $settings['c'] : '',
	        'f' => isset($settings['f']) ? $settings['f'] : '',
	        'ssml' => isset($settings['ssml']) ? $settings['ssml'] : '',
	        'b64' => isset($settings['b64']) ? $settings['b64'] : ''
	    ));
	}
}
?>
```
* In blade file use this code
```php
<div class="col-md-6 pl-0 text-right">
  <div class="micro">
     <i class="fa fa-microphone" onclick="wordSpeech({{ $word->id }})"></i>
  </div>
  <div class="text-center" id="speech-function{{ $word->id }}"></div>
</div>

function wordSpeech(id)
   {
      $.ajax({
      type : 'GET',
      url : '/admin/wordbank/definition/'+id,
      beforeSend: function() {
         $("#wait").css("display", "block");
      },
      success: function(data){
         var downloadFile;      
         $("#wait").css("display", "none");          
         $("#speech-function").html('');                          
         $("#error-message").hide();                          
         $("#download-speech").hide();
         if( data["status"] == 200) {                                                                                                         
            downloadFile = data.responseText["download-file"];
            $("#download-speech").show(); 
            $("#speech-function"+id).html('<audio controls><source src="'+ data.responseText["play-url"] +'" type="audio/mpeg"> Your browser does not support the audio element.</audio> ');
         } 
        }
     });
   }
``` 
* On route web.php add this url or your url which is coming from jquery ajax
```php
Route::get('/admin/wordbank/definition/{definition}', 'Admin\WordBankController@wrodDefinition');
```
* On controller
```php
use App\Library\VoiceRSS;  //For Text to speech

public function wrodDefinition($id)
    {
        $data = Wordbank::find($id);
        // $voice = new COM("SAPI.SpVoice");
        // $voice->Speak($data->definition);
        try {		
			$tts = new VoiceRSS;
			$voice = $tts->speech([
			    'key' => env('VOICE_RSS_API_KEY'),
			    'hl' =>  'en-us',
			    'src' => $data->definition,
			    'r' => '0',
			    'c' => 'mp3',
			    'f' => '44khz_16bit_stereo',
			    'ssml' => 'false',
			    'b64' => 'false'
			]);	
            
            $filename = Str::uuid().'.mp3';
			if( empty($voice["error"]) ) {		
				$rawData = $voice["response"];					
				if (!File::exists(storage_path('app/public/speeches')))
				{ 
					Storage::makeDirectory(public_path('storage/speeches'));
				}

				Storage::disk('speeches')->put($filename, $rawData);
				$speechFilelink =  asset('storage/speeches/'.$filename);							   		                 
			   	$urls["play-url"] = $speechFilelink;		   	
			   	$urls["download-file"] = $filename;			   
			   	$data = array('status' => 200, 'responseText' => $urls);
	          			return response()->json($data);		
			}

	   		$data = array('status' => 400, 'responseText' => "Please try again!");
            return response()->json($data);     

		}catch(\Exception $e) {
            DB::rollback();
            Log::error($e->getMessage());
            return redirect()->back()->with('error', 'Error occurs! Please try again!');
        }
    }
 ```
 
 
