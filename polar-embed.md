# How to embed Polar popup and it' s configuration options

## Polar embed JavaScript:

To embed Polar popup, use this snippet. It will create the necessary `<script>` and container elements for Polar popup to exist.
You can configure where in your page this code will be loaded by creating an element with `fr-polar-popup-placeholder-js` class.
If an element doesn't exist, it will be created and appended at the end of the `<body>`;

```JavaScript
(function() {
  function loadFrPolar(a,b,c){var d="http://froont.com/-polar/embed.js",e=defaultNodeSelector=".fr-polar-popup-placeholder-js";"embedUrl"in c&&c.embedUrl&&(d=c.embedUrl,delete c.embedUrl),"nodeSelector"in c&&c.nodeSelector&&(e=c.nodeSelector,delete c.nodeSelector),a.FrPolar={initConfig:c},d+=(d.search(/\?/)>-1?"&":"?")+"api_token="+c.token;var f=document.querySelector(e);f||(e=defaultNodeSelector,f=document.createElement("div"),f.className=e.substr(1),document.body.appendChild(f));var g=document.createElement("script");g.type="text/javascript",g.src=d,g.addEventListener("error",function(b){console.error("Failed embed Polar popup: "+b.target.src+".\n Please make sure you passed valid athentication token.")});try{f.appendChild(g)}catch(a){console.error("Failed to add script tag to the dom.")}}

  // Configuration:
  var config = {
    // Your configuration goes here
  };

  loadFrPolar(window, document, config);
})(window, document)
```

## Config:
Configuration is defined in TypeScript to illustrate types of each property.

Note:
- Object keys are the keys of the configuration object, values are their types.
- Question mark (`?`) after a key (e.g. `previewURL?`) indicates that it's not required for the popup to work.

### All available properties:
```TypeScript
interface PolarConfig {
  embedUrl: string;  // URL to Polar embed code
  app: string;  // Your Froont user name
  token: string;  // Authentication token
  project: string;  // Froont project slug or `_new_` if new project is needed
  variables: {[variable: string]: string};  // JS Object containing slugs and values of the variables
  css: string;  // Polar generated CSS of the page in string format
  html: boolean;  // Should we return HTML
  previewURL?: string;  // URL to the page to be show in the preview iFrame (defaults to Froont project URL), `.b-content` must be present in preview page
  fileUploadURL?: string;  // URL to use for file upload (defaults to Froont file upload URL)
  resultCallback?: Function|null;  // Function to be called after user has done editing (defaults to null)
  domain?: string;  // Domain in relation to which popup will construct URLs (defaults to `froont.com`)
}
```

### Example configuration object:
```JavaScript
var config = {
  embedUrl: 'http://froont.com/-polar/embed.js',
  app: 'darth-v',
  token: '24fbe8fd-890f-41ff-9cca-201m2a7cgd3b';
  project: 'death-star-plans';
  variables: {
    userName: 'Darth Vaider',
    profileImage: 'https://cdn.com/mask.jpeg'
  },
  css: '.fr_header {background-color: #fff;}',
  html: false,
  previewURL: 'https://empire.com/death-star-plans',
  fileUploadURL: 'https://empire.com/api/file-upload',
  resultCallback: function (resultObject) {
    $.post('/api/save-result', resultObject);
  }
}
```

### Embed URL:
```TypeScript
embedUrl: string
```
This indicates where the logic for constructing popup should be loaded from on Froont server.
It's used to switch between production and development logics as well as to control Polar version.

It will be a variation of this URL:

```
polarUrl: 'http://froont.com/-polar/embed.js'
```

The URL may contain a version for cache busting or to load a specific embed version.
```
http://froont.com/-polar/embed.<version>.js
```

### App identifier:
```TypeScript
app: string
```
This identifies the user whose projects will be used to generate your pages.
To protect your production pages, it will most likely differ between production and development.
Something like – production: `app: 'darth-v'`, development: `app: 'darth-dev'`.


### Session token:
```TypeScript
token: string
```
You will request this token from our server each time you want to construct the Polar popup.
It will allow you to control the user access to features and projects in your possession.

To get the token, execute `POST` request to this endpoint:
```
https://froont.com/-api/get-api-token
```

with request body:
```TypeScript
interface TokenRequestBody {
  api_key: string;  // Your secret API key
  options: {  // Options that will limit the ues of token
    project: string  // Project slug, to restrict the token to the single project
    [option: string]: any  // Any other options
  }
}
```

- For testing, the domain may also be changed to `stage.froont.com`. Also, API token may be moved to request body.
- The request content may be FormData like or JSON. If sending JSON type, also pass the `Content-Type: application/json` header.
- Response will always be JSON in form of:
```TypeScript
interface APITokenResponse {
  message: string;
  data: {api_token: string};
}
```
- Responses use HTTP response codes to identify if they're successful.
- If request is not successful the `message` field will contain error message.
- Successful response Example:
```JSON
{
  "message": "ok",
  "data": {
    "api_token": "24fbe8fd-890f-41ff-9cca-201m2a7cgd3b"
  }
}
```


### Project identifier:
```TypeScript
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

### CSS string:
```TypeScript
css: string
```
The CSS generated by the Polar popup. This may be left empty when a new project is required because it will get generated and returned.
For each subsequent edits, it should be passed, so that user would continue from where he has left off.

### HTML:
```TypeScript
html: boolean
```
This defines wether the HTML of generated page will be returned in result or not.

### Preview URL (not required):
```TypeScript
previewURL: string
```
This URL is used to show preview in the popup.
The page given in the URL will be loaded in the iFrame and Polar CSS will be injected at the end of it's `<head>` section;

Element with class `b-content` must be present in the preview page. We measure if preview has loaded according to it.

If not specified, will default to the URL of Froont project being edited (e.g.: `https://froont.com/vaider-v/deth-star-plans`).


### File upload URL (not required):
```TypeScript
fileUploadURL: string
```
URL which images will be uploaded to. If the user specifies new profile image or cover image the images will be `POST`ed to the given URL one by one with field name `image`.

This is equivalent to file input with name of "image":
```HTML
<input type="file" name="image">
```

This will default to Froont image upload URL.


### Result callback (not required):
```TypeScript
resultCallback: Function|null
```
This is how you get the result from the popup. The given method will be called with the user generated results from the popup.
The result object is almost identical to the config object:

```TypeScript
interface PolarResult {
  app: string;  // Your Froont user name
  token: string;  // Authentication token you passed in
  project: string;  // Project slug you passed in or new generated slug if project was created
  variables: {[variable: string]: string};  // Variables you passed in, updated with the user changed values
  css: string;  // CSS generated by the popup
  html?: boolean;  // HTML from generated page
}
```
