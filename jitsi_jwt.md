# Example Token

```Headers (using RS256 public key validation)
{
  "kid": "jitsi/custom_key_name",
  "typ": "JWT",
  "alg": "RS256"
}
```
# Payload
```
{
  "context": {
    "user": {
      "avatar": "https:/gravatar.com/avatar/abc123",
      "name": "John Doe",
      "email": "jdoe@example.com",
      "id": "abcd:a1b2c3-d4e5f6-0abc1-23de-abcdef01fedcba"
    },
    "group": "a123-123-456-789"
  },
  "aud": "jitsi",
  "iss": "my_client",
  "sub": "meet.jit.si",
  "room": "room_id",
  "exp": 1500006923
}
```

# Check token

To check of token is valid you need to use https://jwt.io/

# Important parames we set in the config

we set jwt credentials in the prosody config, you can set them or update on following path :
```
/etc/prosody/conf.d/cubi.app.cfg.lua
```
These are the parameters :

app_secret : This is the secret we use to create token. Its used to verify token on by jiti, we fill it in VERIFY SIGNATURE part on https://jwt.io/
```
asap_accepted_issuers = { "jitsi", "smash" }
asap_accepted_audiences = { "jitsi", "smash" }
```
This is used for folloing part :
iss : 
sub :

# Example code snippet to generate token in php.
```
function generate_token($meeting_id){

// Create token header as a JSON string
$header = json_encode(['typ' => 'JWT', 'alg' => 'HS256']);

$time = strtotime(' +1 day');
// Create token payload as a JSON string
$payload = json_encode(['aud' => $AUD,"iss"=>$ISS,"sub"=>$HOST,"room"=>$meeting_id,"exp"=>$time]);

// Encode Header to Base64Url String
$base64UrlHeader = str_replace(['+', '/', '='], ['-', '_', ''], base64_encode($header));

// Encode Payload to Base64Url String
$base64UrlPayload = str_replace(['+', '/', '='], ['-', '_', ''], base64_encode($payload));

// Create Signature Hash
$signature = hash_hmac('sha256', $base64UrlHeader . "." . $base64UrlPayload, $SECRET, true);

// Encode Signature to Base64Url String
$base64UrlSignature = str_replace(['+', '/', '='], ['-', '_', ''], base64_encode($signature));

// Create JWT
$jwt = $base64UrlHeader . "." . $base64UrlPayload . "." . $base64UrlSignature;

return $jwt;
}
```


