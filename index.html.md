---
title: API Reference


toc_footers:
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

---

# Introduction

Welcome to the ShootQ developer documentation. 

Comments, questions, or suggestions? Head on over to our github account: https://github.com/ShootQDev/developers.shootq.com

# Contact Form API

You can use javascript or any other programming language to submit a lead 
to one of your contact forms.   

A sample JavaScript implementation is provided later on in this document.

General workflow: 

  * All API URLs have the following prefix: `https://api.shootq.com/api/v1/`
  * Make note of your contact form template identifier in the Contact Form Editor.
  * You will need to get information about the contact form from the ShootQ API using the above contact form template identifier.
  * You may need to get some information about the kinds of values you can submit before posting to that form, such as for Job Types, Job Roles, and Referral Types.
  * Some fields require specific formatting such as the Event Date, URL, and Address.  This documentation will describe each field in detail.
  * You will create a JSON dictionary with all required fields and then submit a POST request. 
  * The API endpoint will return either a 200 HTTP response or some other response with details about what went wrong.


## Get Contact Form Information

You will use the contact form template identifier `form_template_id` to ask for additional form information which you will use to submit the lead.

Make a GET request to the following URL:

### HTTP Request

`contactform/contactformtemplate/<form_template_id>/view/`

The response is a JSON-formatted dictionary with information you will need in a subsequent request. 


```shell
curl "https://api.shootq.com/api/v1/contactform/contactformtemplate/<form_template_id>/view/"
```

> Here is an example response:

```json
{
  "id": 616,
  "additional_fields_options": { ... },
  "form_link": "https://dev.shootq.io/contactform/8ce6984f946b",
  "account_details": {...},
  "job_types_details": [
    {
      "label": "Engagement",
      "value": 27546
    },
    ...
  ],
  "referral_types_details": [
    {
      "label": "Website",
      "value": 7657
    },
    ...
  ],
  "job_roles_details": [
    {
      "label": "Senior",
      "value": 42657
    },
    ...
  ],
  "theme_details": { ... },
  "timezone": "America/Chicago",
  "link_code": "8ce6984f946b",
  ... // Additional information removed
}
```

Below is a table with the information you will require.  You can ignore the other fields because they are irrelevant to submitting the form.

Field Name  |  Type  |  Description
----------- | ------ | -------------
id | Integer | This is the identifier you will use in the next step, where you submit the lead via a POST request.
job_roles_details | array of dictionaries | This array contains the identifier and the human-readable text of the Job Roles that can be submitted in a the job_role field.<br>The dictionary has the following form:<br>`{ value: <integer>, label: "Human Readable Text" }`
job_types_details | array of dictionaries | This is an array of dictionary items that contains the different job types that are accepted in the `job_type` field.  The format is the same as the `job_roles_details`.
referral_types_details | array of dictionaries | This is an array of dictionary items that contains the different referral types that are accepted in the referral_type field.  The format is the same as the job_roles_details.


The rest of the information returned can be ignored, but you can inspect it if you'd like to provide more complex support for your forms.  


## Add a Lead

You will create a JSON-formatted dictionary using some of the information you got from the step above and some of the information gathered from your user.  

### HTTP Request

`/api/v1/contactform/contactformtemplate/<form_id>/submit/`

<aside class="success">
   The `form_id` path element is the "id" field returned in the first step above, when you got the information from the form.
</aside>

Here is an example dictionary.  Below that is a table with the description of the different fields:

```json

var data = {
  "first_name": "Robert",
  "last_name": "Cadena",
  "email": "robert.caden@shootq.com",
  "phone": "310-555-5555",
  "additional_comments": "Thanks!",
  "company_name": "",
  "event_date": "2019-02-05T05:44-07:00",
  "event_location": "The Centennial",
  "address": {
    "address1": "2227 Wilshire Blvd, Santa Monica, CA 90403, USA",
    "city": "Santa Monica",
    "country": "US",
    "state": "CA",
    "street": "2227 Wilshire Blvd",
    "zip": "90403"
  },
  "url": "http://www.shootq.com/",
  "job_role": 42657,
  "job_type": 27546,
  "referral_type": 7657
}
```

<aside class="success">
  Please note, some fields are always required and some fields may be required if you configured your contact form to require them.  You can either look in your Contact Form Editor or you can look in the `additional_field_options` field returned in the previous step.
</aside>

field | type | description
----- | ---- | -----------
first_name | string | The lead's first name 
last_name | string | The lead's last name  
email | string | The lead's email  
phone | string | Lead's phone number 
additional_comments | string | Any additional information to add to the lead 
company_name | string | A company name to add to the lead's contact entry.  
event_date | string | The date to set the main event. The format is `YYYY-MM-DDTHH:mm-HH:mmZ`<br>If no timezone information is set (+-HH:mmZ) then the timezone is UTC.
event_location | string | A free form event location string.  
address | dictionary | A location dictionary with specific information about the event date.  This uses Google Maps API location dictionary format as follows:<br>```{"address1": "2227 Wilshire Blvd, Santa Monica, CA 90403, USA", "city": "Santa Monica", "country": "US", "state": "CA", "street": "2227 Wilshire Blvd", "zip": "90403" }```<br>If you are supplying the address field then all of the above fields in the address dictionary must be present.  Otherwise you will receive a 400 Bad Request Error.
url | string | A url.  It must begin with either `http://` or `https://`.  It cannot be a website address on its own such as www.foo.com.  It must be prefixed by the transport.
job_role | integer | The value field of the appropriate entry from the `job_roles_details`  dictionary. For example, if the lead selected "Senior", then the value here must be "42657" or whatever identifier is associated with the account's job role for Senior. 
job_type | integer | The value field of the appropriate entry from the `job_types_details` dictionary. 
referral_type | integer | The value field of the appropriate entry from the `referral_types_details` dictionary.

Once you have created the JSON data structure, you submit it via POST request to the above URL.

You will receive an HTTP 200 response on success.
Otherwise, you will receive a dictionary with error messages explanining what went wrong.

## Example Code

Below is some example code to get you started. The snippet below assumes you are using jQuery.  As soon as it loads, it will request the form information and then submit a lead using hard-coded data.  Modify the code to point to your own form for testing.

```javascript
$(function($) {
    console.log('init', $);
 
    var FORM_ID = '8ce6984f946b';
    var config = {
        api_root: 'https://dev-api.shootq.io/api/v1/'
    };
 
 
    var ContactForm = function(id) {
        this.id = id;
        this.job_roles = [];
        this.job_types = [];
        this.referral_types = [];
    }
 
    var ContactFormParser = function() {}
 
    ContactFormParser.prototype.parse = function(data) {
        var id = data['id'];
        var form = new ContactForm(id);
 
        form.job_roles = this._parseTuples(data.job_roles_details);
        form.job_types = this._parseTuples(data.job_types_details);
        form.referral_types = this._parseTuples(data.referral_types_details);
 
        return form;
    }
 
    ContactFormParser.prototype._parseTuples = function(field) {
        var tuples = [];
 
        for (var i = 0; i < field.length; i++) {
            tuples.push(field[i]);
        }
 
        return tuples;
    }
     
    function getURL(path) {
        return config.api_root + path;
    }
 
    function getForm(formId, completion) {
        $.ajax(getURL('contactform/contactformtemplate/'+formId+'/view/'))
        .done(function(data) {
            console.log('got', data);
            var parser = new ContactFormParser();
 
            var form = parser.parse(data)
            console.log('got form', form);
 
            if (completion) {
                completion(true, form);
            }
        })
        .fail(function(jqXHR, textStatus, errorThrown) {
            console.error(textStatus)
            console.error(errorThrown)
 
            if (completion) {
                completion(false, null);
            }
        })
    }
 
    function submitForm(formId, data, completion) {
        console.log('submitting form', formId, data);
        var data = JSON.stringify(data);
 
        $.ajax({
            url: getURL('contactform/contactformtemplate/8ce6984f946b/submit/'),
            type: 'POST',
            data: data,
            dataType: 'json',
            contentType: "application/json; charset=utf-8",
        })
            .done(function(data, textStatus) {
                console.log('submit done', textStatus);
 
                if (completion) {
                    completion(true);
                }
            })
            .fail(function(jqXHR, textStatus, errorThrown) {
                console.error('submit error', textStatus);
                console.error(errorThrown);
 
                if (completion) {
                    completion(false);
                }
            });
    }
 
 
    function init() {
        getForm(FORM_ID, function(success, form) {
            console.log('getForm', success, form);
 
            if (!success) {
                return;
            }
 
            var newData = {
                "first_name": "Robert",
                "last_name": "Cadena",
                "email": "robert.cadena+test@shootq.com",
                "phone": "310-333-3333",
                "additional_comments": "Thanks",
                "company_name": "Sample Company Name",
                "event_date": "2019-02-05T05:44-07:00",
                "event_location": "The Centennial",
                "address": {
                    "address1": "2227 Wilshire Blvd, Santa Monica, CA 90403, USA",
                    "city": "Santa Monica",
                    "country": "US",
                    "state": "CA",
                    "street": "2227 Wilshire Blvd",
                    "zip": "90403",
                },
                "url": "http://www.robert-cadena-tester.com"
            };
 
            // Now we need to set the data stuff
            newData['job_role'] = form.job_roles[0].value;
            newData['job_type'] = form.job_types[0].value;
            newData['referral_type'] = form.referral_types[0].value;
 
 
            submitForm(FORM_ID, newData, function(success) {
                console.log('submit', success);
            });
        });
    }
 
    init();
});
```

