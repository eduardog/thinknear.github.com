---
layout: post
title: "Reverse Engineering Honeybadger Incoming Error Request"
date: 2015-05-01 15:25:00 -0700
author: Tassos Livogiannis
comments: true
categories:
---

Our systems at Thinknear run on a 24/7 basis and monitoring them for errors is essential to prevent production issues that can have a negative impact on our business.
One of the tools we use for error reporting is [Honeybadger](https://www.honeybadger.io).
When one of our systems sends an error (usually a Ruby or Java exception) it gets posted to the Honeybadger website, engineers get an email notification and they can log into Honeybadger to see the details of the exception.
It is important for debugging to see all the relevant error information in Honeybadger, grouped in a useful way.

Below is the summary portion of an error reported to Honeybadger.
{% img center /images/honeybadger-post/error_summary.png 945 457 'image' 'images' %}

Recently, we wanted to add information about the incoming request (both headers and body) and we had to look deeper into Honeybadger's [incoming error API](https://www.honeybadger.io/pages/collector) to identify where things show up in a Honeybadger error page.
The [payload tester](https://app.honeybadger.io/notice/test) tool provided is great to test sample JSON requests and see how the error will be reported on the page.
However, the sample page it shows does not contain the Context and App Environment sections.
Furthermore, the documentation does not provide a mapping of JSON keys and values to elements on the resulting error page so we had to reverse engineer the format of the page, based on test requests.

A JSON request to the Honeybadger incoming error API should contain the following top level elements:

* notifier
* error
* request
* server

The Honeybadger error page is broken up into 4 major sections:

1. Summary
1. Full backtrace
1. Context
1. App Environment

## Summary

Summary is the top most element in an error page and it contains basic information about the error.
In this section of the page, the following can be displayed:

* Message: this is where the string inside the ```error.message``` element shows.
* Backtrace: this is where the first element of the ```error.backtrace``` array shows.
* URL: this is where the request url that can be found under the ```request.url``` element shows.
* Location: this is where the string under the ```server.environment_name``` element shows.
* Tags: this is where the strings under the ```error.tags``` array show.

For example, the following (partial) JSON

```
"error":{
"class":"RuntimeError",
"message":"RuntimeError: This is a runtime error",
"tags":["error"],
"backtrace":[
  {
    "number":"4",
    "file":"/Users/anastasiosl/code/demoapp/app/controllers/pages_controller.rb",
    "method":"runtime_error"
  }
]
},
"request":{
"url":"http://demoapp.dev/pages/runtime_error"
},
"server":{
"environment_name":"development"
}
```

will yield the page of this image:
{% img center /images/honeybadger-post/summary_message.png 577 494 'image' 'images' %}

## Full Backtrace

Here, Honeybadger displays a backtrace (stacktrace) that is built by the information under ```error.source``` and ```error.backtrace``` elements.
The ```error.source``` element contains line number and source code information of the piece of code that threw the exception or error.
The ```error.backtrace``` contains information of the call hierarchy up to the method that failed.
Certain Honeybadger integration libraries, such as [Honeybadger-Ruby](https://github.com/honeybadger-io/honeybadger-ruby) can build this elements for you, given an actual programming language exception (or ```Error``` in the case of Ruby).

To illustrate how ```error.source``` and ```error.backtrace``` work, the following (partial) JSON
```
"source": {
  "2": "",
  "3": "  def runtime_error",
  "4": "    raise RuntimeError.new(\"This is a runtime error\")",
  "5": "  end",
  "6": ""
},
"backtrace": [
  {
    "number":"4",
    "file":"/Users/anastasiosl/code/demoapp/app/controllers/pages_controller.rb",
    "method":"runtime_error"
  },
  {
    "number": "15",
    "file": "/Users/anastasiosl/code/demoapp/app/controllers/pages_controller.rb",
    "method": "throw_runtime_error"
  }
]
```
will yield the page of this image: 
{% img center /images/honeybadger-post/backtrace_message.png 903 291 'image' 'images' %}

## Context

In this section, Honeybadger will display everything that is under the ```request.context``` JSON object.
Please, note that Honeybadger error page ignores any other element object under ```request```, with the exception of ```request.url``` which appears in the Summary section, as shown above.
As an example, the error page will not display any values under the ```request.cgi_data``` object.
To ensure request data (i.e. request body, request headers etc.) appear in the error page, they need to be added to the ```request.context``` object.

To illustrate this, the following (partial) JSON

```
"request": {
    "url":"http://demoapp.dev/pages/runtime_error",
    "context": {
        "request_body": { "test-key": "test-value" },
        "request_headers": { "x-openrtb-version": "2.0", "Content-Type": "application/json" },
        "user_id": "123",
        "user_email": "test@example.com",
        "made_up_key": "made_up_value"
    }
}
```

will yield a page that looks like this:
{% img center /images/honeybadger-post/context_message.png 394 285 'image' 'images' %}

Note, the ```Email user``` button that appears under context when a ```user_email``` value is present under ```request.context```.

## App Environment

In this section, Honeybadger will display everything that is under the top level ```server``` element.
Custom keys will be read and displayed along with the ```environment_name``` key that also appears in the summary section, as explained above.

For example, the following (partial) JSON

```
"server": {
    "environment_name": "development",
	"hostname": "example.com",
	"made_up_key": "made_up_value"
}
```

will yield a page that looks like this.
{% img center /images/honeybadger-post/env_message.png 319 157 'image' 'images' %}

## Conclusion

Honeybadger's [payload tester](https://www.honeybadger.io/pages/collector) only shows how the Summary and Full Backtrace will appear based on test requests.
Hopefully, our findings presented above shed some light to those areas not covered by the payload tester.
As a reference, the full example JSON request presented in this blog post can be found in [Github](https://gist.github.com/anastasiosl/90fbc4a5b20c305d6929).
