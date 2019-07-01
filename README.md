# How to embed Froont Instant popup and it' s configuration options

## Froont Instant embed JavaScript:

To embed Froont Instant popup, use this snippet. It will create the necessary `<script>` and container elements for Froont Instant popup to occupy.

To use the snippet, put it in a `<script>` tag at the end of your `<body>` element (right before the `</body>`).

Also create a `<div>` element with a class `polar-popup-placeholder-js`, where you want the code to be loaded (it must be before the snippet).

```javascript
(function() {
  function loadFrPolar(e,o,t){var r,a="http://froont.com/-polar/embed.js",d=defaultNodeSelector=".fr-polar-popup-placeholder-js";if(e.FrPolar&&e.FrPolar.polarLoaded)return void(window.location.hash="/magic/edit/"+t.project);"embedUrl"in t&&t.embedUrl&&(a=t.embedUrl,delete t.embedUrl),"nodeSelector"in t&&t.nodeSelector&&(d=t.nodeSelector,delete t.nodeSelector),e.FrPolar={initConfig:{}};for(var l in t)e.FrPolar.initConfig[l]=t[l];a+=(a.search(/\?/)>-1?"&":"?")+"api_token="+t.token,r=/^#[A-z0-9_-]+$/.test(d)?document.selectElementById(d.substr(1)):/^\.[A-z0-9_-]+$/.test(r)?document.getElementsByClassName(d.substr(1))[0]:document.querySelector(d),r||(d=defaultNodeSelector,r=document.createElement("div"),r.className=d.substr(1),document.body.appendChild(r));var n=document.createElement("script");n.type="text/javascript",n.src=a,e.FrPolar.polarLoaded=!0,n.addEventListener("error",function(e){console.error("Failed embed Polar popup: "+e.target.src+".\n Please make sure you passed valid athentication token.")});try{r.appendChild(n)}catch(e){console.error("Failed to add script tag to the dom.")}}

  // Configuration:
  var config = {
    // Your configuration goes here
  };

  loadFrPolar(window, document, config);
})(window, document)
```


## Example document:
```html
<!DOCTYPE html>
<html>
  <head><!-- Your document meta data (title, favicon, etc...) --></head>
  <body>
  <!-- your document content (images, text, etc...) -->
  <div class="polar-popup-placeholder-js"><!-- this is where the code will go once it's loaded --></div>
  <script>
    // The snippet goes here
  </script>
  </body>
</html>
```


## Config:
Configuration is defined in TypeScript to illustrate types of each property.

Note:
- Object keys are the keys of the configuration object, values are their types.
- Question mark (`?`) after a key (e.g. `previewURL?`) indicates that it's not required for the popup to work.


### All available properties:
```typescript
interface FroontInstantConfig {
  embedUrl: string;  // URL to Froont Instant embed code
  app: string;  // Your Froont user name
  token: string;  // Authentication token
  project: string;  // Froont project slug or `_new_` if new project is needed
  variables: {[variable: string]: string};  // JS Object containing slugs and values of the variables
  stepLabels?: {[slug: string]: string};  // JS Object containing app step slugs and label values
  css?: string;  // Froont Instant generated CSS of the page in string format
  html?: boolean;  // Should we return HTML
  cssSelectorPrefix?: string;  // CSS selector to prefix each style selector
  previewURL?: string;  // URL to the page to be show in the preview iFrame (defaults to Froont project URL), `.b-content` must be present in preview page
  fileUploadURL?: string;  // URL to use for file upload (defaults to Froont file upload URL)
  resultCallback?: Function|null;  // Function to be called after user has done editing (defaults to null)
  domain?: string;  // Domain in relation to which popup will construct URLs (defaults to `froont.com`)
  upgradeTitle?: string;  // Upgrade title shown above upgrade button
  upgradeButtonText?: string;  // Upgrade button text
  upgradeCallback?: Function|null;  // Function to be called when user clicks upgrade button (defaults to null)
  onEditorOpenCallback?: Function|null;  // Function to be called when editor opens (defaults to null)
  onEditorCloseCallback?: Function|null;  // Function to be called when editor closes (defaults to null)
  onEditorReadyCallback?: Function|null;  // Function to be called when editor is ready, project is loaded (defaults to null)
  imageLibrary?: string|Array<{title: string, url: string, tags: string[]}>;  // An array of Image library image objects or URL (GET) returning such array in JSON
  useProjectDoneCallback?: boolean;  // Whether to return a callback function when project is saved
}
```


### Example configuration object:
```javascript
var config = {
  embedUrl: 'https://froont.com/-polar/embed.js',
  app: 'darth-v',
  token: '24fbe8fd-890f-41ff-9cca-201m2a7cgd3b';
  project: 'death-star-plans';
  variables: {
    userName: 'Darth Vaider',
    profileImage: 'https://cdn.com/mask.jpeg'
  },
  stepLabels: {
    'select-page': 'Choose Page',
    'design': 'Edit Style'
  },
  css: '.fr_header {background-color: #fff;}',
  html: false,
  cssSelectorPrefix: '.star-wars',
  previewURL: 'https://empire.com/death-star-plans',
  fileUploadURL: 'https://empire.com/api/file-upload',
  useProjectDoneCallback: false,
  resultCallback: function (resultObject) {
    $.post('/api/save-result', function () {
      if (resultObject.projectDoneCallback) {
        resultObject.projectDoneCallback();
      }
    });
  },
  upgradeTitle: 'Upgrade required to enable this feature',
  upgradeButtonText: 'Upgrade now',
  upgradeCallback: function upgradeCallback(stepSlug, token) {
    console.log('Feature', stepSlug);
    console.log('Token', token);
  },
  onEditorOpenCallback: function () {
    // Called when editor opens
  },
  onEditorCloseCallback: function () {
    // Called when editor closes
  },
  onEditorReadyCallback: function () {
    // Called when editor is ready, project is loaded
  }
}
```


### Embed URL:
```typescript
embedUrl: string
```
This indicates where the logic for constructing popup should be loaded from on Froont server.
It's used to switch between production and development logics as well as to control Froont Instant version.

It will be a variation of this URL:

```
polarUrl: 'https://froont.com/-polar/embed.js'
```

The URL may contain a version for cache busting or to load a specific embed version.
```
https://froont.com/-polar/embed.<version>.js
```

### App identifier:
```typescript
app: string
```
This identifies the user whose projects will be used to generate your pages.
To protect your production pages, it will most likely differ between production and development.
Something like – production: `app: 'darth-v'`, development: `app: 'darth-dev'`.


### Session token:
```typescript
token: string
```
You will request this token from our server each time you want to construct the Froont Instant popup.
It will allow you to control the user access to features and projects in your possession.

To get the token, execute `POST` request to this endpoint:
```
https://froont.com/-api/get-api-token
```

with request body:
```typescript
interface TokenRequestBody {
  api_key: string;  // Your secret API key
  options: {  // Options that will limit the use of token
    project: string  // Project slug, to restrict the token to the single project
    available_steps: string[]  // List of available steps for user (step slug is visible in browser address bar)
    [option: string]: any  // Any other options
  }
}
```

- For testing, the domain may also be changed to `stage.froont.com`. Also, API token may be moved to request body.
- The request content may be FormData like or JSON. If sending JSON type, also pass the `Content-Type: application/json` header.
- Response will always be JSON in form of:
```typescript
interface APITokenResponse {
  message: string;
  data: {
    api_token: string,  // API token
    expires_at: string,  // ISO formatted token expiration date and time
    project?: string  // Project slug, returned only if requested in request options
  };
}
```
- Responses use HTTP response codes to identify if they're successful.
- If request is not successful the `message` field will contain error message.
- Response returns `project: '_new_'` with status code `404` if specific project is requested but not found.
- Successful response Example:
```JSON
{
  "message": "ok",
  "data": {
    "api_token": "24fbe8fd-890f-41ff-9cca-201m2a7cgd3b",
    "expires_at": "2017-08-24T10:02:23.226427",
    "project": "empire-page-template"
  }
}
```


### Project identifier:
```typescript
project: string
```

This is a short unique identifier of the project you would like to edit.
- If the project doesn't exist yet for the user, you will pass new project identifier (`project: '_new_'`) or the identifier of your base project (e.g. `project: 'empire-page-template'`).
- It will create a new project from the given base project.
- Your base projects are marked as `example` on Froont side, so you don't need to pass any additional options.


### Variables:
```TypeScript
variables: {[variable: string]: string}
```
Variables is a plain JS object that contains the identifiers and the values of the variables in the given project.

The available variables will be specified once the projects are finalized,
but here are some that are available for the test page.
```JavaScript
{
    "profile-image": "https://cdn.froont.com/elza-karklina/8vmklx/_static-uploads/images/thumbnail/deffault-img-900x900.png_900x900.png",
    "social-links": "",
    "facebook": "",
    "instagram": "",
    "twitter": "",
    "vimeo": "",
    "soundcloud": ""
}
```


### Step Labels (optional):
```typescript
stepLabels: {[slug: string]: string}
```
Step Labels is a plain JS object that contains step slug and it's display name (label) in the app UI. Step labels are visible as the final part of each step URL. For instance, the slug `design` is the final part of URL for Froont Instant popup design step - `#/magic/edit/_new_/design`.


### CSS string (optional):
```typescript
css: string
```
The CSS generated by the Froont Instant popup. This may be left empty.


### HTML (optional):
```typescript
html: boolean
```
This defines wether the HTML of generated page will be returned in result or not.


### CSS selector prefix (optional):
```TypeScript
cssSelectorPrefix: string
```
A CSS selector string that will be prefixed to all style selectors generated by the Froont Instant app. This doesn't include the CSS passed through the CSS parameter. The CSS selectors in CSS code passed through CSS parameter should already be prefixed if prefixes are required.


### Preview URL (optional):
```typescript
previewURL: string
```
This URL is used to show preview in the popup.
The page given in the URL will be loaded in the iFrame and Froont Instant CSS will be injected at the end of it's `<head>` section;

Element with class `b-content` must be present in the preview page. We measure if preview has loaded according to it.

If not specified, will default to the URL of Froont project being edited (e.g.: `https://froont.com/vaider-v/deth-star-plans`).


### File upload URL (optional):
```typescript
fileUploadURL: string
```
URL to which files used by user will be uploaded to. If the user specifies new profile image, cover image, audio file for playback or any other type of file for download the files will be `POST`ed to the given URL one by one with field name `user_file`.

This is equivalent to file input with name of "user_file":
```html
<input type="file" name="user_file">
```

**Other form fields**

- _deprecated_: `image`(`<input type="file" name="image">`) - contains user file for backwards compatibility
- `variable` - name of variable (if uploaded directly thorugh variable input field)
- `type` - type of file that is being uploaded (`image`, `audio`, `any`). The type depends on input type used for file selection

Request example
```http
------WebKitFormBoundary8BSs1WNpAksN5Aex
Content-Disposition: form-data; name="image"; filename="some-image.png"
Content-Type: image/png


------WebKitFormBoundary8BSs1WNpAksN5Aex
Content-Disposition: form-data; name="user_file"; filename="some-image.png"
Content-Type: image/png


------WebKitFormBoundary8BSs1WNpAksN5Aex
Content-Disposition: form-data; name="variable"


------WebKitFormBoundary8BSs1WNpAksN5Aex
Content-Disposition: form-data; name="file_type"

image
------WebKitFormBoundary8BSs1WNpAksN5Aex--
```

This will default to Froont image upload URL. (Froont doesn not support "any" file upload to Froont server.)

**Required Response**

```typescript
interface FileUploadResponse {
  message: string;  // Any kind of status message, this will be displayed if the response is error
  data: {
    url: string;  // API token
  };
}
```


### Result callback (optional):
```typescript
resultCallback: Function|null
```
This is how you get the result from the popup. The given method will be called with the user generated results from the popup.
The result object is almost identical to the config object:

```typescript
interface FroontInstantResult {
  app: string;  // Your Froont user name
  token: string;  // Authentication token you passed in
  project: string;  // Project slug you passed in or new generated slug if project was created
  variables: {[variable: string]: string};  // Variables you passed in, updated with the user changed values
  css: string;  // CSS generated by the popup
  html?: string;  // HTML from generated page
  projectDoneCallback?: () => void;  // Callback function to be called when you're finished with result processing and wish to notify the user
}
```


### Upgrade title (optional):
```typescript
upgradeTitle: string
```
Upgrade title shown above upgrade button when requested step is not available for user.
If not specified, system default upgrade title will be shown.


### Upgrade button text (optional):
```typescript
upgradeButtonText: string
```
Upgrade button text. Button is visible only if requested step is not available for user.
If not specified, system default upgrade button text will be shown.


### Upgrade callback (optional):
```typescript
upgradeCallback: Function|null
```
The given method will be called when user clicks upgrade button.
Method parameters:
`stepSlug` returns requested step slug,
`token` user token.


### Editor open callback (optional):
```typescript
onEditorOpenCallback: Function|null
```
The given method will be called when editor opens.


### Editor close callback (optional):
```typescript
onEditorCloseCallback: Function|null
```
The given method will be called when editor closes.


### Editor ready callback (optional):
```typescript
onEditorReadyCallback: Function|null
```
The given method will be called when editor is ready, project is loaded.


### Image Library (optional, Image Library access required):
```typescript
imageLibrary: string|Array<{
  title: string;
  url: string;
  tags: string[];
  type?: 'image'|'audio'|'any';
}>
```
An array of Image library image objects or URL (GET) returning such array in JSON.
To use this option, Image Library access is required.
If specified, image input will open Image Library instead of browser's file select window.
Each image object must contain image title, URL to the image file and array of tags.
Additionally item may contain `type` field. By default all items are considered images, but other types of items can be passed (`audio`, `any`) and they will be made available to user in inputs that require such items.



### Use project done callback (optional):
```typescript
useProjectDoneCallback: boolean
```
Whether to return a callback function when project is saved. When this option is set to `true` a function with name `projectDoneCallback` will be returned as a property of result object. And the popup will continue to show user the lading screen until the returned function is called.
This is useful when you have some computation to be done on the frontend once you receive the data. This may prevent user from leaving the site too soon.


## Webhook:
All the project data is sent as JSON to the webhook endpoint using `POST` method every time the project is saved.

```typescript
interface WebhookRequest {
  app: string;  // Your Froont user name
  token: string;  // Authentication token you passed in
  project: string;  // Project slug you passed in or new generated slug if project was created
  variables: {[variable: string]: string};  // Variables you passed in, updated with the user changed values
  css: string;  // CSS generated by the popup
  html: string;  // URL of the project HTML export API endpoint
}
```


## Project cloning (duplication) API:
To clone an existing Froont Instant project, execute `POST` request to the project **clone** endpoint:

```
https://froont.com/-api/[project]/-api/clone
```

Pass your API key as `api_key`  field in the body:

```typescript
interface CloneRequestBody {
  api_key: string;  // Your secret API key
  options: {  // Options that will limit the use of token
    available_steps: string[]  // List of available steps for user (used for token generation)
  }
}
```

- For testing, the domain may also be changed to `stage.froont.com`.
- The request content may be FormData like or JSON. If sending JSON type, also pass the `Content-Type: application/json` header.
- Response will always be JSON in form of:
```typescript
interface APICloneResponse {
  message: string;
  data: {
    author: string,  // Name of application that cloned the project (the `app` field in Instant config)
    description: string|null,
    is_private: boolean,
    slug: string,  // The slug of the new project
    tags: string[],  // List of tags the project has
    title: string,
    url: string,  // URL to the new project
    token: {
      api_token: string,  // API token
      expires_at: string,  // ISO formatted token expiration date and time
    }
  };
}
```
- Responses use HTTP response codes to identify if they're successful.
- If request is not successful the `message` field will contain error message.
- If cloning the project is successfull, but creating the token isn't then project data will be returned in the error response. With the error message.
- Successful response Example:
```JSON
{
  "data": {
    "author": "darth-v", 
    "description": null, 
    "is_private": false, 
    "slug": "death-star-plans-kxx4z3e", 
    "tags": [], 
    "title": "The death star plans", 
    "token": {
      "api_token": "616f3c80-f8b2-4380-9d42-891a845d0a4c", 
      "expires_at": "2019-06-27T06:10:54.488694"
    }, 
    "url": "http://froont.com/darth-v/death-star-plans-kxx4z3e"
  }, 
  "message": "Project cloned successfully!"
}
```


