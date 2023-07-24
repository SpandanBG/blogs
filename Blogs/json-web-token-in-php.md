# JSON Web Token (JWT) in PHP 

---

---

**JSON Web Token (JWT)** is a token based client authentication architecture. It is a compact and URL safe means of authenticating clients. The working of a token works is as such; the client gives it's credentials to the server to authenticate itself, the server then verifies the credentials and authorizes a token (usually with an expiration date) and sends it back to the client. Now, when the client wants to make any subsequent requests for the protected data, all it has to do is send the request along with the token to the server. The server then authenticates the token and responds to the request.

The JWT is stateless, that means, there is no instance of the token stored in the server. When a token is received, it verifies it's authentication and responds appropriately. This is a reason why JWT is used along with the RESTfull architecture.

**Structure of JWT** is quite simple and elegant. It has three parts header, payload and signature, each separated by a dot. Example

```jwt
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```

Each of the part is base64_encoded (base64 encoding is optional to the signature), thus allowing each of them to be decoded easily to show the content. This is a reason why any sensitive data should be avoided in the token. The above in base64_decoded as follows:
```json
//Header: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
{
  "alg": "HS256",
  "typ": "JWT"
}
```

```json
//Payload: eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```

The signature `TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ` is not base64_encoded, it's in its true self.

### What are these?

- **Header**The header consists of two parts, <code>typ</code> and <code>alg</code>. The <em>typ</em> defines the type of the token; here it is <code>JWT</code>. The <em>alg</em> defines the algorithm used to sign the token; here it is <code>HMAC SHA256</code>. The header array,
<pre class="EnlighterJSRAW" data-enlighter-language="json" data-enlighter-group="jwt_12" data-enlighter-title="header array">{ 'typ':'JWT', 'alg':'HS256' }</pre>
, is base64_encoded to get the header string.
- **Payload**The payload consists of claims, which are statements about the client. There are three types of claims: reserved claim, public claim and private claim. The reserved claims consists of the JWT information, like expiration date, issued server name, etc. The public claim is given by the JWTs users. The private claim consists of the information to be shared by the two parties. The payload array,
<pre class="EnlighterJSRAW" data-enlighter-language="json" data-enlighter-group="jwt_13" data-enlighter-title="payload array">{ 'sub': '1234567890', 'name': 'John Doe', 'admin': true }</pre>
, is base64_encoded to get the payload string.
- **Signature**The signature is generated with <em>hash based message authentication code (hmac)</em> taking the header, payload and a secret that is known only by the owner of the server. This is the validation of the header and payload has not been tampered with.

## Code Work
To implement JWT in PHP I cooked up a simple php class. It consists of the following public functions:

- **Generate Token**This function takes the public claim array in the parameter and adds it to the payload array. This function calls a getsignature() function that returns the value generated by the hash_hmac() function.
```php
/*
	group="jwt_php_1"
	title="gen_token($claims)"
*/

public function gen_token($claims){
	try{
		// Loading header
		$this->head = $this->base64url_encode(json_encode(array('typ'=>'JWT', 'alg'=>'HS512')));
		
		// Loading Payload
		$exp = time() + (7*24*3600);
		$iat = time();
		$reserved = array('iss'=>'sudocoding', 'exp'=>$exp, 'nbf'=>$iat, 'iat'=>$iat);
		$this->payload = $this->base64url_encode(json_encode(array('res'=>$reserved, 'clm'=>$claims)));
		
		// Loading Signature
		$this->signanture = $this->base64url_encode($this->getsignature());
		
		return $this->head.".".$this->payload.".".$this->signanture;
	} catch (Exception $err) {
		return null;
	}
}
```

```php
/*
	group="jwt_php_1"
	title="getsignature()"
*/

private function getsignature(){
	try{
		$secret = 'zuUo 0ykU UgsU u2Uo 0zUt lzUh kUrk tUyn U2nk loxk UutU U2gy uxrj s4U2';
		return hash_hmac('sha512', "$this->head.$this->payload", $secret, false);
	} catch (Exception $err) {
		return null;
	}
}
```

```php
/*
	group="jwt_php_1"
	title="base64url_encode($data)"
*/
private function base64url_encode($data) {
	return rtrim(strtr(base64_encode($data), '+/', '-_'), '=');
}
```

- **Verify Token**This function is responsible to verify if the token is valid or not. It grabs the header and payload and generates a signature with the secret and checks it against the signature that came along with the token. This function also verifies if a token has expired by calling a private function $this->ver_exp().
```php
/*
	group="jwt_php_2" 
	title="ver_token($tok)"
*/
public function ver_token($tok){
	try{
		$arr = explode('.',$tok);
		$this->head = @$arr[0];
		$this->payload = @$arr[1];
		$this->signanture = base64_decode(@$arr[2]);
		$sign = $this->getsignature();
		if($sign === $this->signanture and $this->ver_exp()){
			return base64_decode($this->payload);
		} else {
			return null;
		}
	} catch (Exception $err) {
		return null;
	}
}
```

```php
/*
	group="jwt_php_2"
	title="ver_exp()"
*/
private function ver_exp(){
	try{
		$res = base64_decode($this->payload);
		$res = json_decode($res,true);
		$res = $res['res'];
		$ct = time();
		if($ct<=$res['exp'] and $ct>=$res['nbf']){
			return true;
		} else {
			return false;
		}
	} catch (Exception $err){
		return false;
	}
}
```

- **Refresh Token**This function is responsible to refresh a token, that is, push the expiration date further, if the token is still valid. It uses the existing token and updates the $exp in the payload and generates a new token to be sent back.
```php
/*
	group="jwt_php_3"
	title="rfs_token($tok)"
*/

public function rfs_token($tok){
	try{
		// Verifying token
		if($this->var_token($tok)!==null){
			// Refreshing exp time and iat time
			$res = json_decode(base64_decode($this->payload), true);
			$exp = time() + (7*24*3600);
			$iat = time();
			$res['res']['exp'] = $exp;
			$res['res']['iat'] = $iat;
			$this->payload = $this->base64url_encode(json_encode($res));
			
			$this->signanture = $this->base64url_encode($this->getsignature());
			return $this->head.'.'.$this->payload.'.'.$this->signanture;
		} else {
			return null;
		}
	} catch (Exception $err) {
		return null;
	}
}
```

## The Working
Lets have a idea of it's working. For this I will create three scripts, namely, login.php, profile.php and settings.php. When a user tries to access profile.php or settings.php without loggin in they will receive a json object stating an error. After login they will have access to only profile.php and not to settings.php.

- **Login.php**This script accepts two post parameters, usr and pss. After processing which it generates the JWT with the public claim as <em>{'usr':'user', 'scope':['profile']}</em>. This token is then sent to the client and upon further request this token is to be used along with the header.

```php
/*
	group="php_1"
	title="login.php"
*/
<?php require_once 'enigma.php'; if($_SERVER['REQUEST_METHOD']=='POST'){ if(@$_POST['usr']==='user' and @$_POST['pss']==='user123'){ $jwt = new ENIGMA(); $token = $jwt->gen_token(array('usr'=>@$_POST['usr'], 'scope'=>array('profile')));
		echo json_encode(array('auth'=>$token));
	} else {
		echo json_encode(array('auth'=>'failed'));
	}
} else {
	echo json_encode(array('auth'=>'failed'));
}

?>
```

- **Profile.php**This scripts extract the headers and check for <em>Authenticate</em> header from which it extracts the JWT. The token is then verified and the scope parameter is checked to verify if the client has access to this page.

```php
/*
	group="php_2"
	title="profile.php"
*/
<?php require_once 'enigma.php'; $auth = getallheaders(); if(!empty($auth['Authenticate'])){ $jwt = new ENIGMA(); $auth = @$auth['Authenticate']; $res = $jwt->ver_token($auth);
	if(strpos($res, 'profile')!==False){
		echo json_encode(array('usr'=>'user'));
	} else {
		echo json_encode(array('auth'=>'err2'));
	}
}else {
	echo json_encode(array('auth'=>'err1'));
}
?>
```

- **Settings.php**This script is similar to the profile.php script. The only difference is that this one is bound to be inaccessible since the scope parameter in the login.php script is only set to profile.php.

```php
/*
	group="php_3"
	title="settings.php"
*/
<?php require_once 'enigma.php'; $auth = getallheaders(); if(!empty($auth['Authenticate'])){ $jwt = new ENIGMA(); $auth = @$auth['Authenticate']; $res = $jwt->ver_token($auth);
	if(strpos($res, 'settings')!==False){
		echo json_encode(array('settings'=>'user'));
	} else {
		echo json_encode(array('auth'=>'err2'));
	}
}else {
	echo json_encode(array('auth'=>'err1'));
}
?>
```

JWT is really an elegant solution to stateless authentication, also unlike cookies, it is free from CSRF or XSRF exploits, thus making it safer. You can check out the JWT libraries for different languages from [here](https://jwt.io/#libraries-io), or create your own.

---

Spandan Buragohain,
2017-06-03 05:30:24