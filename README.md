# Steps to reproduce

## On Google Tag Manager interface

1. If you don't have one already, create a Google Tag Manager account.

2. In the "ADMIN" section, create a new container. For this test the created container was named "localhost" and "Where to Use Container" was set to "Web".

3. In the "WORKSPACE" section, create a new trigger:

  * Choose "Form Submission" as trigger type.
  * Tick the "Wait for Tags" option. Leave the default value as "Max wait time" (2000), however with this option enabled we are forced to set at least one condition for the trigger to be enabled. Choose whatever fits your requirements. For this test the options "Page URL", "matches RegEx", ".*" were used (always active).
  * Let the trigger to fire on "All Forms".
  * Save the trigger.

  > NOTE: this configuration is not really required to reproduce the bug, but it is required in order to let the tag firing the event and be able to see it in Analytics. You can configure the trigger as you like, the only important thing is the trigger type.

3. In the "WORKSPACE" section, create a new tag:

  * Choose "Universal Analytics" as tag type.
  * Insert your Google Analytics Tracking ID in the "Tracking ID" box.
  * Choose "Event" as "Track Type".
  * Fill the rest of parameters as you like.
  * In the "Triggering" section, set the previously created trigger.
  * Save the tag.

4. Publish the changes.

## On your website

1. Create an HTML page with this example content:

  ```
  <!DOCTYPE html>
  <html>
  <!-- we are going to add GTM here... -->
  <body>
  <!-- ...and here -->
  <form><button type="submit" name="okbutton">OK</button></form>
  </body>
  </html>
  ```

  > NOTE: The page should be served by an HTTP server, otherwise you will not be able to see the event tracked in Analytics. In any case, also opening the file directly with the browser, you will be able to reproduce the bug.

2. Visit the page, for the example it was hosted by an "Apache HTTP Server" server running locally, at the URL http://localhost/testsubmit.html

3. Click the button in the page. You will notice the URL to change, by adding at the end of the URL

  `?okbutton=`

  this is because the form in the page has no action neither a method, so by default it sends the data to the same URL using GET method.

3. If we check the posted parameter we will notice the parameter `okbutton` with an empty value. This is because the button we click is named `okbutton` and its value is empty, so this is send as parameter when we submit the form by clicking the same button.

4. Now we are going to add the Google Tag Manager in the page:

  * On Google Tag Manager interface, go to "ADMIN" section
  * Select the container configured previously in the select box "CONTAINER"
  * Click on "Install Google Tag Manager"
  * Copy and paste the code, following the instruction

5. Now our page should look something like this:

  ```
  <!DOCTYPE html>
  <html>
  <head>
  <!-- Google Tag Manager -->
  <script>(function(w,d,s,l,i){w[l]=w[l]||[];w[l].push({'gtm.start':
  new Date().getTime(),event:'gtm.js'});var f=d.getElementsByTagName(s)[0],
  j=d.createElement(s),dl=l!='dataLayer'?'&l='+l:'';j.async=true;j.src=
  'https://www.googletagmanager.com/gtm.js?id='+i+dl;f.parentNode.insertBefore(j,f);
  })(window,document,'script','dataLayer','GTM-XXX');</script>
  <!-- End Google Tag Manager -->
  </head>
  <body>
  <!-- Google Tag Manager (noscript) -->
  <noscript><iframe src="https://www.googletagmanager.com/ns.html?id=GTM-XXX"
  height="0" width="0" style="display:none;visibility:hidden"></iframe></noscript>
  <!-- End Google Tag Manager (noscript) -->
  <form><button type="submit" name="okbutton">OK</button></form>
  </body>
  </html>
  ```

  > NOTE: Don't forget to change `GTM-XXX` with your Google Tag Manager ID!

6. Reload the page in the browser

7. Click again the button (did you notice something?).

8. Now check your Google Analytics account. In the section "Reporting" > "Real Time" > "Events" if everything worked properly you should be able to see the event tracked.

## The bug

If you go back to your page, and you check the URL, you will notice there are no parameters after the page name. This is because for some reason the GTM code prevents the field from being submitted. 
It seems to happen only for the submit input type, using an element of type `button` or an element of type `input`, provided that the attribute `type` is `submit` or `image` and its attribute `value` is empty or undefined.
