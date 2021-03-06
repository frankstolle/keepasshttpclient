# KeePassHTTPClient

## Requirements
 - php >= 5.5
 - [guzzlehttp/guzzle](https://github.com/guzzle/guzzle) >= 6
 
## Install

Best way is to install with [composer](https://getcomposer.org/)

```sh
$ composer require valicek1/keepasshttpclient
```

## Sample code

```php
# load key and it's id
$secret = loadKey(); // you need to implement this by yourself

# create class
# $secret[0] is 256-bit (32 characters) long key
# $secret[1] contains key id
$kpx = new KeePassHTTPClient($secret[1], $secret[0]);

# is the key associated with DB?
if (!$kpx->testAssociated()) {
	# if not, try to authorize
	$label = $kpx->authorizeKey();
	# and save key + label for next use
	saveKey($label, $secret[1]); // you need to implement this yourself
}

if ($kpx->testAssociated()) {
  // you are welcome to do something in database
  $url = "https://skype.com";
  
  // get logins
  $logins = $kpx->getLogins($url, $url); // first URL is page URL, second one is Submit url for "form"
  
  // or just their count
  $count = $kpx->getLoginsCount($url, $url)
   
  // or create new pairs in database
  $kpx->setLogin($url, $url, "username", "realPassword") 
 
}
```


## Technological process

### Pairing with KeePass
1. `testAssociated` - optional
2. `authorizeLey`, save key id returned - for future request
3. `testAssociated` - if succeeds, you can continue by sending requests

### Requests
Server requires `testAssociated` at the beginning of every new session, or after failure. There is written you have to do it, but personally, I haven't seen any difference in KeepassHTTP's behaviour without `testAssociated` 

1. `testAssociated` - before first request
2. `getLoginsCount`, `getLogins` or `setLogin` - work with data

## Documentation

### Constructor

```php
/**
 * KeePassHTTPClient constructor.
 * @param string $key Encryption key, 256 bits
 * @param string $label Label (product of association)
 * @param string $address KeepassHTTP listening address
 * @param int    $port KeepassHTTP listening port
 */
public function __construct($key, $label = "", $address = "localhost", $port = 19455){}
```

### Getters

```php
public function getAddress(){} // address of server
public function getPort(){} // port of server
public function isAssociated(){} // state of last testAssociated
public function getClient(){} // return guzzle client
```

### Functions to work with KeePass
```php
public function isServerListening(){} // check, if port is opened
public function sendRequest($json){} // json encode $json and send it as request. Return json object of response
public function testAssociated($empty = FALSE){} // test if key is associated or server responding (empty = TRUE)
public function authorizeKey(){} // authorize key (from constructor)
public function getLogins($url, $redirectUrl){} // find logins/passwords for entered criteria.
public function getLoginsCount($url, $redirectUrl){} // count number of items for entered criteria
public function setLogin($url, $redirectURL, $login, $password){} // save login to KeePass DB
```

It's not much pretty, but enough for using. You can still look into [source](src/KeePassHTTPClient.php)..
