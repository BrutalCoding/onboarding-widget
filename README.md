# Frankie Onboarding Widget

## Table of contents

- [Overview](#overview)
- [Getting started](#getting-started)
- [Configuration](#configuration)

## Overview

A embedable widget to bring a self onboarding form to any web page...

**Features**:
1. Some feature  
2. Another feature

![](screenshots/document-selection.png)
# Getting started



1. Serialise and base64 encode your Frankie Api Credentials using ":" as a separator and POST it to Frankie Financial Client Api (details in the code snippet below)  
a. "CUSTOMER_ID:API_KEY", if you don't have a CUSTOMER_CHILD_ID  
b. "CUSTOMER_ID:CUSTOMER_CHILD_ID:API_KEY, otherwise  
2. The response header will contain a temporary api token  
2. Define your optional configuration object, according to the section [Configuration](#configuration)  
3. Add both the link to the Roboto font and the widget .js file to the head of the webpage  
4. Add the web component to the page, passing the following attributes  
a. **ff**, the token  
b. **applicant-reference**, the string reference that will be injected into this applicant's data and can be used to request  their details aftwerwards, both via Frankie API and Frankie Portal  
c. *optional* **width**, the width exactly as would be defined in css (defaults to 375px)  
d. *optional* **height**, the height exactly as would be defined in css (defaults to 812px)  
e. *optional* **config**, the configuration object first stringified and then URI encoded. The algorithm needs to be compatible with Node's encodeURI  


## 1. Obtaining an API token
Example in Node + Express + Axios
```javascript
  // Have your Frankie credentials in hand
  const apiKey = process.env.FRANKIE_API_KEY,
        customerId = process.env.FRANKIE_CUSTOMER_ID,
        customerChildId = process.env.FRANKIE_CUSTOMER_CHILD_ID;
  // Serialize your credentials, by joining them with a ":" separator symbol
  //  customerId:customerChildId:apiKey OR customerId:apiKey
  //  where if you don't posses a customerChildId, you should omit it and the separator symbol ":" all together
  const decodedCredentials = [customerId, customerChildId, apiKey].filter(Boolean).join(":");
  // Base64 encode the resulting string
  const encodedCredentials = Buffer.from(decodedCredentials).toString('base64');
  // POST the endpoint "auth/v1/machine-session" of Frankie Client Api service, whose URL will be provided to you by Frankie
  // Include the encoded credentials in the "authorization" header, as follows
  // "authorization": `machine ${encodedCreentials}`
  // and extract the header "token" from the response
  const frankieUrl = process.env.FRANKIE_API_URL;
  axios.post(`${frankieUrl}/auth/v1/machine-session`, {}, {
    headers: { authorization: "machine " + encodedCredentials }
  }).then(data => {
    const headers = data.headers;
    const ffToken = headers.token;
    // pass the extracted token to the widget as an html attribute called 'ff' (see demo.ejs)
    res.render('the-web-page.ejs', {
      title: "Frankie Financial Widget Demo",
      ffToken: ffToken
    });
  })

```
## 2. Embeding widget
Head of the html page (link to font and the js file)
```html
  <head>
    <!-- viewport meta is recommended for responsive pages -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <!-- initially only the Roboto font family is supported and therefore the following line is required to be included. This will be configurable in next iterations -->
    <link href="https://fonts.googleapis.com/css2?family=Roboto:ital,wght@0,300;0,400;0,700;1,300;1,400&display=swap" rel="stylesheet">
    <!-- Include the Web component script -->
    <script src="./ff-onboarding-widget.min.js"></script>
  </head>
```
Body of the html page, wherever desired
```html
<body style="margin: 0">
  <ff-onboarding-widget
    width="500px" height="900px"
    ff="<%= ffToken %>"
    config="<%- encodeURI(JSON.stringify(widgetConfiguration)) %>"></ff-onboarding-widget>
</body>
```

# Configuration
More configurations and customisations will be available soon. Right now our goals are the following.
- [x] Customize/Disable Welcome Screen
- [x] Customize accepted document types
- [x] Customize maximum attempt count
- [x] Customize text throughout the widget
- [x] Hide the progress bar
- [x] Customize success page redirect url
- [ ] Customize font
- [ ] Customize all styles freely
- [ ] Customize success page content
- [ ] Customize progress bar range, start value and end value
- [ ] Dispatch events on every step of the progress of the user to allow greater interaction between the host platform and the widget

## All current options and their defaults
```typescript
documentTypes: DocType[] = ['PASSPORT', 'DRIVERS_LICENCE', 'MEDICARE'],
welcomeScreen: {
  // html string to be displayed in the welcome screen. It accepts style tags, but script tags will be stripped out.
  // the default welcome screen is available in the screenshot after this section
  htmlContent: string | boolean = false,
  ctaText: string = "Start Identity Verification"
}
// the number of times the applicant will be allowed to review personal details and try new documents before failing their application
maxAttemptCount: number = 5
// By default only a
successScreen: {
  // url to redirect after applicant clicks button in the successful page
  // by default the widget only displays a successful message
  // you can always include the applicant-reference as a query parameter to continue any remaining onboarding steps that might come after the identity verification
  ctaUrl: string | false = false;
}


```

![](screenshots/welcome-screen.png)


```javascript
// Optionally set widget options as defined in "Configuration", as well as their default values. All entries are optional.
  const widgetConfiguration = {
    documentTypes: ['PASSPORT', 'DRIVERS_LICENCE', 'MEDICARE'],
    welcomeScreen: {
      htmlContent: `
        <h1 class='title'>The title</h1>
        <p class='bold'>We need to collect some personal information to verify your identity before we can open your account.</p>
        <p class='bold'>You�ll need</p>
        <ul style=''>
          <li>5 mins of your time to complete this application</li>
          <li>You must be over 16 years of age</li>
        </ul>
        <style>
          ul {
            list-style-image: url(/bullet.png);
          }
        </style>
      `,
      ctaLabel: 'Start now',
    },
    maxAttemptCount: 5,
  };
  ```