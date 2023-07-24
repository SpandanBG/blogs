# RESTfull Implementation using PHP 

---

---

**Representational state transfer (REST) **is a way to provide web service to different client apps. It was defined by Roy Fielding in his 2000 PhD  dissertation "Architectural Styles and the Design of Network-based Software Architectures" at UC Irvine.

![image](https://drive.google.com/uc?export=view&id=0B0EaugnQp5mWWE1ldDk0LWx6bm8)

The architectural properties of the REST architecture are, the increase in performance, scalability, simplicity, modifiablity, visibility, portability and reliability. The idea behind the REST architecture is that all requests are to be responded with a common data format, either JSON or XML. This allows different systems to communicate with each other over the HTTP protocol. With this architectural design it is easy to scale the web service with newer interacting components.

The REST architecture has six constraints to be defined as a RESTful system.

- Client-Sever Model: Separation of user interface and the data, thus improving the portability of the user interface system.
- Stateless: The client-server communication is not  constrained by any client context being stored on the server between the requests.
- Cacheable: The responses from the service should be marked as cacheable or not in order to prevent redundant requests.
- Layered System: Intermediate systems in the web service can improve the scalability and applying security of different tiers.
- Code on demand: Ability to send executable codes to the client side on demand, like javascripts, applets, etc.
- Uniform interface: Decoupling the architecture to enable each part to evolve independently.

The REST architecture can be written in any language, but here I'll be writing it in PHP. The following source code is the REST class. The main task of the object of this class is to break down the HTTP URL and the sent along GET or POST form data; and send a response with required http code and JSON data.

```php
/*
    group="rest_1"
    title="rest.php"
*/
<?php
/* @author: Spandan Buragohain REST class to handle all the requests and response */
class REST {
    private $_inpst = []; // array of received inputs private
    private $_func, $_http, $_http_scheme; // $_func -> page tried to access, $_http -> the http request method, $_http_scheme -> http/https

    // Constructor to grab the http method, http scheme, the page access and the form data
    public function __construct() {
        $this->_http = $_SERVER["REQUEST_METHOD"];
        $this->_http_scheme = $_SERVER["REQUEST_SCHEME"];
        $this->inprc();
    }

    // This method returns the page tried to access
    public function get_function() {
        return $this->_func;
    }

    // This method returns the set of form data
    public function get_data() {
        return $this->_inpst;
    }

    // This method returns the http request method
    public function get_http() {
        return $this->_http;
    }

    // This method is responsible to return the http scheme
    public function get_http_scheme() {
        return $this->_http_scheme;
    }

    /* The method returns the json object of the dataset Also sets the appropriate header */
    public function respond($dataset, $_code) {
        header("HTTP/1.1 " . $_code . " " . $this->gstat_msg($_code));
        header("Content-Type: application/json");
        echo json_encode($dataset);
    }

    // This method is responsible to extract the form data into an array
    private function inprc() {
        $this->_func = basename($this->validate(@$_REQUEST["rquest"]), ".php");
        if ($this->_func === null) {
            $this->respond(null, 500);
        } else {
            switch ($this->get_http()) {
                case "POST":
                    $this->_inpst = $this->scrapinp(@$_POST);
                    break;
                case "GET":
                    $this->_inpst = $this->scrapinp(@$_GET);
                    break;
                default:
                    $this->respond(null, 500);
            }
        }
    }

    // This method words in coordination with the inprc() method
    private function scrapinp($dataset) {
        $inps = [];
        if (is_array($dataset)) {
            foreach ($dataset as $key => $val) {
                if ($key !== "rquest") {
                    $inps[$key] = $this->scrapinp($val);
                }
            }
            return $inps;
        } else {
            return $this->validate($dataset);
        }
    }

    // This method strip the data of html chars
    private function validate($data) {
        if (is_array($data)) {
            return null;
        }
        $data = stripslashes($data);
        $data = trim($data);
        $data = htmlspecialchars($data);
        return $data;
    }

    // This method returns the string of the response code
    private function gstat_msg($_code) {
        $status = [
            100 => "Continue",
            101 => "Switching Protocols",
            200 => "OK",
            201 => "Created",
            202 => "Accepted",
            203 => "Non-Authoritative Information",
            204 => "No Content",
            205 => "Reset Content",
            206 => "Partial Content",
            300 => "Multiple Choices",
            301 => "Moved Permanently",
            302 => "Found",
            303 => "See Other",
            304 => "Not Modified",
            305 => "Use Proxy",
            306 => "(Unused)",
            307 => "Temporary Redirect",
            400 => "Bad Request",
            401 => "Unauthorized",
            402 => "Payment Required",
            403 => "Forbidden",
            404 => "Not Found",
            405 => "Method Not Allowed",
            406 => "Not Acceptable",
            407 => "Proxy Authentication Required",
            408 => "Request Timeout",
            409 => "Conflict",
            410 => "Gone",
            411 => "Length Required",
            412 => "Precondition Failed",
            413 => "Request Entity Too Large",
            414 => "Request-URI Too Long",
            415 => "Unsupported Media Type",
            416 => "Requested Range Not Satisfiable",
            417 => "Expectation Failed",
            500 => "Internal Server Error",
            501 => "Not Implemented",
            502 => "Bad Gateway",
            503 => "Service Unavailable",
            504 => "Gateway Timeout",
            505 => "HTTP Version Not Supported",
        ];
        return $status[$_code] ? $status[$_code] : $status[500];
    }
} ?>
```

The following code is the REST handle class, that incorporates with the REST class to run the requested function and the send back appropriate response.

```php
/*
    group="rest_2"
    title="h_rest.php"
*/
<?php require_once "rest.php";
require_once "getdb.php"; /* REST Handler class */
class Handler {
    // REST object
    private $rest;

    public function __construct() {
        $this->rest = new REST();
    }

    /* The method checks if a function exists and gets the json dataset to be sent */

    public function run() {
        $_func = $this->rest->get_function();
        $_dataset = $this->rest->get_data();
        if (function_exists($_func)) {
            $this->rest->respond(
                $_func(
                    $this->rest->get_http(),
                    $_dataset,
                    $this->rest->get_http_scheme()
                ),
                200
            );
        } else {
            $this->rest->respond(["dataset" => "$_func"], 404);
        }
    }
}

// DEMO method to retrive blog dataset
function blog($http, $ds, $https) {
    if ($http === "POST" and $https === "https") {
        $bname = @$ds["bname"];
        if (!empty($bname)) {
            $con = new MongoClient(); // new MongoClient("mongodb://Username:Password@localhost");
            $db = $con->sudocoding;
            $col = $db->selectCollection("blogs");
            $cur = $col->find(["title" => "$bname"]);
            return $cur->getNext();
        } else {
            return null;
        }
    } else {
        return null;
    }
}

// Starting the handler $rest = new Handler(); $rest->run();
?>
```

In the above script, I have defined a demo function that access a blog from the database and sends back JSON object of the data. For the database I have used MongoDB. The document in the database is:

![image](https://drive.google.com/uc?export=view&id=0B0EaugnQp5mWdTA4eGR2cGlSY0E)

The URL redirection is done with the following .htaccess file:
```htaccess
# group="rest_3"
# title=".htaccess"

RewriteEngine On

RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-s
RewriteRule ^(.*)$ h_rest.php?rquest=$1 [QSA,NC,L]

RewriteCond %{REQUEST_FILENAME} -d
RewriteRule ^(.*)$ h_rest.php [QSA,NC,L]

RewriteCond %{REQUEST_FILENAME} -s
RewriteRule ^(.*)$ h_rest.php [QSA,NC,L] 
```

To test the REST architecture I am using the [RESTClient add-on on Firefox](https://addons.mozilla.org/en-US/firefox/addon/restclient/). To it I send the following HTTP post URL request `localhost/blog.php` with the parameter as `bname=Blog 1`

![image](https://drive.google.com/uc?export=view&id=0B0EaugnQp5mWNEM4TUJtaWRFREE)

The response is the following JSON object of the request blog:

```json
/*
group="rest_4"
title="json"
*/ 
{
  "_id": {
    "$id": "592d21412b07ee0d100bb7c8"
  },
  "title": "Blog 1",
  "description": "This is the first blog",
  "author": "sudokid",
  "likes": "100",
  "comments": {
    "usr1": {
      "name": "user name 1",
      "comment": "this is a comment"
    },
    "usr2": {
      "name": "user name 2",
      "comment": "this is the second comment"
    }
  }
}
```

This project codes are available in my [GitHub repository](https://github.com/SpandanBG/REST-api-in-PHP), feel free to check it out.

---

Spandan Buragohain,
2017-06-01 07:09:55
