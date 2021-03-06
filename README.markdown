PHP REST Client
===============
(c) 2013 Travis Dent <tcdent@gmail.com>

Basic Usage
-----------

    $api = new RestClient(array(
        'base_url' => "https://api.twitter.com/1.1", 
        'format' => "json", 
         // https://dev.twitter.com/docs/auth/application-only-auth
        'headers' => array('Authorization' => 'Bearer '.OAUTH_BEARER), 
    ));
    $result = $api->get("search/tweets", array('q' => "#php"));
    // GET http://api.twitter.com/1.1/search/tweets.json?q=%23php
    if($result->info->http_code == 200)
        var_dump($result->decode_response());


Configurable Options
--------------------
`headers` - An associative array of HTTP headers and values to be included in every request.  
`parameters` - An associative array of parameters to be merged with individual request parameters in every request.  
`curl_options` - cURL options to apply to every request. These will override any automatically generated values.  
`user_agent` - User agent string.  
`base_url` - URL to use for the base of each request.  
`format` - Format to append to resource and support decoding.  
`format_regex` - Pattern to extract format from response Content-Type header.  
`decoders` - Associative array of format decoders, see documentation below.  
`username` - Username to use for basic authentication. Requires `password`.  
`password` - Password to use for basic authentication. Requires `username`.  

Options can be set upon instantiation, or individually afterword:

    $api = new RestClient(array(
        'format' => "json", 
        'user_agent' => "my-application/0.1"
    ));

-or-

    $api = new RestClient;
    $api->set_option('format', "json");
    $api->set_option('user_agent', "my-application/0.1");


Direct Iteration and Response Decoding
--------------------------------------
If the the response data format is supported, the response will be decoded 
and accessible by iterating over the returned instance. When the `format` 
option is set, it will be used to select the decoder. If no `format` option 
is provided, an attempt is made to extract it from the response Content-Type 
header. This pattern is configurable with the `format_regex` option.

    $api = new RestClient(array(
        'base_url' => "http://vimeo.com/api/v2", 
        'format' => "php"
    ));
    $result = $api->get("tcdent/info");
    foreach($result as $key => $value)
        var_dump($value);

Reading via ArrayAccess has been implemented, too:

    var_dump($result['id']);

To access the decoded response as an array, call `decode_response()`.

'json' and 'php' formats are configured to use the built-in `json_decode` 
and `unserialize` functions, respectively. Overrides and additional 
decoders can be specified upon instantiation, or individually afterword. 
Decoder functions take one argument: the raw request body. Functions 
created with `create_function` work, too. 

    function my_xml_decoder($data){
        new SimpleXMLElement($data);
    }

    $api = new RestClient(array(
        'format' => "xml", 
        'decoders' => array('xml' => "my_xml_handler")
    ));

-or-

    $api = new RestClient;
    $api->set_option('format', "xml");
    $api->register_decoder('xml', "my_xml_decoder");

Or, without polluting the global scope with a runtime function. This 
particular example allows you to receive decoded JSON data as an array.

    $api->register_decoder('json', 
        create_function('$a', "return json_decode(\$a, TRUE);"));


