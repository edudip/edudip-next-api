# edudip next API

**Version 2020-08-24** 

## Use case

The edudip next API can be used for program control of the edudip next functions. In this way, webinars that have been created can be read out and changed for example. This facilitates integration of edudip next webinars in your own website or application.

## General structure

edudip next API is a REST API (https://en.wikipedia.org/wiki/Representational_state_transfer). JSON is used as the format for data exchange.

Each function of the API is represented by a so-called endpoint. Each endpoint consists of a URL, the corresponding HTTP verb (GET, POST, PUT, DELETE), and optionally a list of required parameters.

In this document we will only specify the path of the endpoint. Each endpoint must therefore always be preceded by ```https://api.edudip-next.com```.

If an endpoint requires a list of parameters, these parameters must be encoded as multipart/form-data (https://www.w3.org/TR/html5/sec-forms.html#multipart-form-data) or application/x-www-form-urlencoded (https://www.w3.org/TR/html5/sec-forms.html#urlencoded-form-data).

Each API request should also contain the HTTP header ```Accept``` with the value ```application/json```.

Example of an implementation of a POST request using PHP and cURL:

```
$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, 'https://api.edudip-next.com/api/webinars');
$httpHeaders = [
    'Accept: application/json',
    'Authorization: Bearer API-Token',
];
curl_setopt($ch, CURLOPT_HTTPHEADER, $httpHeaders);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, 'parameter1=value1&parameter2=value2');
$response = curl_exec($ch);
curl_close($ch);
var_dump($response);

```

For all API requests that could be successfully processed, the HTTP status code 200 OK is returned. If an API request fails, an appropriate HTTP status code is returned.

If you are missing functions in the API for integration in your application, please contact us and we will be happy to discuss with you whether this functionality can be added to the API.

## Reference implementation

A sample implementation of the edudip next API can be downloaded from https://github.com/edudip/next-api-client. This implementation is based on PHP and cURL and can be easily integrated in your project using composer.

## Authentication

### Authenticated requests with API token

All API requests described in this document must be authenticated. So-called API tokens are used to authenticate requests. An API token is an ASCII string with a length of 60 characters. Each API token is assigned to a user in edudip next. Each action that is performed via the API is assigned to a team member via the API token and therefore has the same access rights as this team member.

The API token must be included in every API request as an HTTP header named ```Authorization```:

    Authorization: Bearer [API-TOKEN]

Replace [API-TOKEN] with your generated API token.

API tokens must not be passed on to third parties and should be kept safe. To ensure full security, all API requests must take place via HTTPS.


### Creating API tokens

To generate an API token, please log in to edudip next with your email address and password. Now click on the menu item "Settings" on the left side. Now scroll down until you reach the item "API-Tokens". Now click on the button "Generate another API-Token". The system will now ask you to specify which team member this API token should be assigned to. Select the desired team member and then click on generate “API-Token".

Note: If the item "API-Token" does not appear or you do not have the option of generating an API token, the API has not been activated for your account. In this case please contact sales@edudip.com.

## Webinars

### List all webinars

**Endpunkt:** GET /api/webinars

Returns a list of all existing webinars.

Return value:

     {
         "success": Boolean,
         "total": Uint,
         "webinars": [
         		// A list of webinar objects
         ]
     }

The property "success” is set to "true" if the project is successful. The property "total" specifies the total number of existing webinars. The property "webinars" contains an array of webinar objects. A webinar object is encoded as a JSON object and has the following properties:

|Property|Data type|Description|
|----|------|------|
|id|Uint|Primary, unique identifier of the webinar|
|users_id|Uint|User ID of the team member who created this webinar|
|title|String|Title or name of the webinar|
|access|Enum/String ("all" oder "invitation")|Describes who can book the webinar. "all" means that anyone can book the webinar. "invitation" means that the participant may only register for the webinar with an invitation.|
|registration_type |Enum/String|Beschreibt, ob sich Teilnehmer für einen Termin des Webinars oder immer für alle Termine des Webinars registrieren (Einzel- oder Serientermin)|
|dates|Array|A list with dates of the webinar|
|next_date|Object|Includes the next scheduled date of the webinar. Is ```null``` if no upcoming event exists.|
|max_participants|Uint|Maximum number of webinar participants.|
|moderators|Array|A list of (co)moderators of the webinar. The creator of the webinar is always entered as the main moderator.|
|participants_count|Uint|Number of participants already registered for this webinar|
|landingpage|Array|Contains the relevant landing page information. Among this are the properties ```url``` for the URL of the landing page, ```image``` for an object with information about the deposited image or YouTube video, ```description_short``` and ```description``` for the short (limited to 120 characters) and long description of the webinar.|
|created_at|String|Time of creation of the webinar in the form ```YYYY-MM-DD HH:ii:ss```|
|updated_at|String|Time of last modification of the webinar in the form ```YYYY-MM-DD HH:ii:ss```|


### Create a new webinar

**Endpoint:** POST /api/webinars

This API endpoint can be used to create new webinars. The following parameters must be transferred as an HTTP POST field when using the endpoint:

|Parameter|Data type|Required|Description|
|----|------|:---:|------|
|title|String|✓|The title/name of the webinar. Must be a minimum of 5 and a maximum of 190 characters.|
|max_participants|Uint|✓|Maximum number of participants in the webinar. Must be set to at least 1. The maximum value depends on your booked edudip next subscription.|
|recording|Uint|✓|Should a video clip of the webinar be recorded? 1 = The webinar will be recorded; 0 = Do not record the webinar|
|registration_type|String|✓|Can accept the values "series" or "date". “series" = appointment series: Participants register for all appointments at the same time; "date" = alternative appointments: Participants register individually for each appointment.|
|access|String|✓|Can accept the values ```all``` or ```invitation```. ```all``` = anyone may register, ```invitation``` = only invited participants may register|
|dates|String|✓|JSON-encoded array with individual date objects on which the webinar should take place. Each date object must have two properties: "date" with the date string in the form ```YYYY-MM-DD HH:MM:SS```, on which the appointment should take place, and the property "duration", which specifies in minutes how long the appointment should last. Example: ```[{"date":"2018-01-20 12:00:00","duration":20}]```|
|users_id|Uint|✕|Defines which team member should be the owner (main moderator) of the webinar. This team member requires a moderator license. If the parameter is not provided, the user to which the API token belongs is entered as the owner|
|language|String|✕|The language of the webinar. Allowed values: ```de``` or ```en```|

**Please note**, that the creator of the webinar (main moderator) must be present in the webinar in order for it to start or automatically start.

### Read out a single webinar

** Endpoint:** GET /api/webinars/[Webinar-Id]

This endpoint can be used to specifically read out the values of an individual webinar. More values of the webinar are returned here compared to the endpoint ```GET /api/webinars```.

If successful, the return delivers the following JSON return:

    {
    	"success": true,
    	"webinar": {
    		// Webinar object
    	},
    	"stats": {
    		"views_total": Uint,
    		"registrations_total": Uint
    	}
    }

The property ```webinar``` contains an object that contains all values of the requested webinar. In addition, the property stats contains the values ```views_total``` with the number of landing page views that belongs to this webinar. In addition, the ```registrations_total``` value specifies the number of existing registrations for this webinar.

The returned webinar object is largely identical to the webinar object that is delivered via the ```GET /api/webinars``` endpoint, although the following values are also delivered:

|Property|Data type|Description|
|----|------|------|
|registration\_type\_editable|Boolean|If this value is set to ```true```, the registration type (alternative dates/appointment schedule, property registration_type) can no longer be changed.|
|recording|Uint|1 = the webinar will be recorded; 0 = the webinar will not be recorded|
|slug|String|The SEO Slug used for the webinar landing page URL|
|participants|Array|A list of registered participants for this webinar. A description of an individual participant object can be found below this table|
|dialin_enabled|Uint|**This property is currently without function** 0 = phone dial-in not available; 1 = phone dial-in is available for this webinar|
|recordings|Array|A list of recordings produced for this webinar.|

A participant object has the following properties:

|Property|Data type|Description|
|----|------|------|
|firstname|String|Forename of the participant|
|lastname|String|Surname of the participant|
|auth_key|String|Authentication key. Required to enter the webinar room and log out of webinars|
|email|String|Participant's email address|
|created_at|String|Time of registration|
|updated_at|String|Time of the last modification of the data record|

### Delete webinar

**Endpoint:** DELETE /api/webinars/[Webinar-Id]

### Modify an existing webinar

** Endpoint:** PUT /api/webinars/[Webinar-Id]

Changes the values/settings of an existing webinar. The following parameters can be transferred when calling the endpoint (analogous to HTTP POST requests). It is not necessary to transfer all parameters every time, it is also possible to transfer a subset only:

|Parameter|
|---------|
|title|
|max_participants|
|recording|
|registration_type|
|access|

The meaning and value range of the individual parameters can be found in the description of the endpoint ```GET /api/webinars``` and ```GET /api/webinars/[Webinar-Id]```.

In case of success a JSON object is returned, with the property ```success``` with the value ```true```, as well as the property ```webinar``` which contains the modified and updated webinar object.

### Add a new appointment to a webinar

**Endpoint:** POST /api/webinars/[Webinar-Id]/add-date

|Property|Data type|Description|
|----|------|------|
|date|String|Date and time when the appointment should take place. Format: ```YYYY-MM-DD HH:MM:SS``` (e.g. 2019-12-01 12:30:00)|
|duration|Uint|Duration of the appointment in minutes.|


Return in case of success:

    {
         "success": true,
         "date": [Webinar-Date Object]
    }

### Delete existing date of a webinar

**Endpoint:** DELETE /api/webinars/[Webinar-Id]/dates/[Webinar-Date-Id]

**Please note** that the last date of a webinar cannot be deleted, i.e. at least one date must exist for each webinar. Furthermore, when a date is deleted, all associated data is also deleted. This includes all recordings and registration data of participants for this date. We therefore recommend that you save all recordings and relevant data before deleting an appointment.

### Register a participant for a webinar

**Endpoint:** POST /api/webinars/[Webinar-Id]/register-participant

Registers a participant for the specified webinar. The following POST parameters can be specified when calling the endpoint:


|Parameter|Data type|Required|Description|
|----|----|----|------|
|email|String|✓|Participant's email address|
|firstname|String|✓|Forename of the participant|
|lastname|String|✓|Surname of the participant|
|webinar_date|String|✕|Date in the format ```YYYY-MM-DD HH:MM:SS```, if you wish to register for one date only (if property ```registration_type``` of the webinar is set to ```date`` (single appointment)).|

**Return in case of error:**

     {
        "success": false,
        "error": {
            "message": String, Fehlermeldung
            "type": Error code (see below)
            "fields": [
                // Optional, only if "type" has value "form_validation". Contains an array with fieldnames, that contain invlid or missing values
            ]
        }
     }

**Rückgabe im Erfolgsfall:**

    {
        "success": true,
        "participant": {
            "auth_key": String,
            "firstname": String,
            "lastname": String,
            "updated_at": String, format: YYYY-MM-DD HH:ii:ss,
            "created_at": String, format: YYYY-MM-DD HH:ii:ss
        },
        "registeredDates": [
            {
                "date": String, format YYYY-MM-DD HH:ii:ss,
                "key": String,
                "room_link": String
            }
        ]
    }

The ```auth_key``` field contains the personal authorisation key with which a participant in the room can identify themselves. This key is already inserted in the webinar room link in the ```room_link``` property. The participant accesses the webinar room for the relevant date directly using the ```room_link``` property. Each webinar appointment has its own link. The link is personalised and only intended for this participant.

**Error codes:**

|Code|Meaning|
|----|---------|
|form_validation|One or more registration fields were not filled in or were filled in incorrectly|
|date_missing|Single appointment was selected as registration type, but no appointment was specified for which the participant should be registered|
|date_not_bookable|The specified webinar date (Post parameter "webinar_date") can no longer be booked or is in the past|
|participant_exists|A participant with the specified email address has already been registered|

### De-register a participant from a webinar

**Endpoint** POST /api/webinars/[Webinar-Id]/cancelRegistration

|Parameter|Data type|Description|
|----|----|-------|
|email|String|Email address of the participant|
|auth_key|String|DThe participant's authorisation key (see "Registering a participant for a webinar").|

#### Upload participant avatar

**Endpoint** POST /participants-management/participant/[ParticipantEmail]/avatar

|Parameter|Data type|Description|
|----|----|-------|
|avatar|multipart/form-data|Image file which becomes the new avatar of the participant.|

**Please note**, that the uploaded image is provided in the resolutions ```600x800px``` and ```64x64px```. To ensure this, the image will be automatically adjusted and cropped by us.

#### Delete participant avatar

**Endpoint** DELETE /participants-management/participant/[ParticipantEmail]/avatar

### Deleting a participant

**Endpoint:** DELETE /api/participants/[E-Mail-Adresse des Teilnehmers]

Deletes all participant data from edudip next. Can be used, for example, to comply with requests in accordance with Art. 17 GDPR regarding the right to deletion. The difference to the endpoint ```POST /api/webinars/[Webinar-Id]/cancelRegistration``` is that after de-registration, the participant's data remain stored in the system for further processing.

### Add a moderator to a webinar

**Endpoint:** POST /api/webinars/[Webinar-Id]/moderators/add

Adds a new (co)moderator to an existing webinar. Up to three (co)moderators can be added (in addition to the webinar owner). The following parameters must be passed to this endpoint.

|Parameter|Data type|Description|
|----|----|-------|
|email|String|Email address of the new moderator|
|firstname|String|Forename of the moderator (displayed in the webinar room)|
|lastname|String|Surname of the moderator (displayed in the webinar room)|

**Please not**, that the main owner of the webinar must be present in the webinar room to start the webinar.

### Remove an existing moderator from a webinar

**Endpoint:** DELETE /api/webinars/[Webinar-Id]/moderators/[Moderator-Email]

Removes a (co)moderator from a webinar.

### Read the attendance time of the attending participants of a webinar appointment

**Endpoint:** GET /api/webinars/[Webinar-Id]/[Webinar-Date-Id]/participant-attendance

The return of the attendance time is delivered in the form of a JSON object, where the email address of the participant is set as the property and the attendance time in minutes as the value.

The webinar appointment ID of the appointment for which the attendance time should be read out can be read out via the API endpoint ```read out single webinar```.

**Example of a return in case of success**

    {
        "success": true,
        "attendance": {
        	"max.mustermann@example.com": 12,
	     	"john.doe@example.com": 31,
 			  ...
        }
    }


## Recordings

### List all recordings

**Endpoint** GET /api/recordings

The following GET parameters can be passed to the API endpoint to paginate the results:

|Parameter|Data type|Required|Description|
|----|----|-------|--------|
|offset|Uint|✕|Index of the first element to be output (0 corresponds to the first element of the list)|
|limit|Uint|✕|Number of output elements|


**Please note** that recording takes time and recordings are not available immediately after the webinar. It usually takes about half the time that was taken by the webinar. For this reason, you may need to implement a polling mechanism in your application - depending on the application case - to check for the presence of recordings of a webinar. We recommend a time interval of 10 minutes.

### Output the download URL of a single recording

**Endpoint** GET /api/recordings/[Recording-Id]/download-url

Returns an URL, via which it is possible to download the recording as an MP4 file. The Recording-Id can be read via the endpoint GET /api/records. Please note that the download URLs are only valid for a limited time. Usually the URLs are valid for about 2 hours.

### List recordings of a single webinar

**Endpunkt** GET /api/webinars/[Webinar-Id]/recordings

Reads out all recordings of the specified webinar.


