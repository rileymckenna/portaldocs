<a name="frameblade"></a>
## FrameBlade

FrameBlades provide an alternative programming model for developing UI in Ibiza. This alternative gives the extension author full control over the DOM via an IFrame. Fx controls cannot be used within FrameBlades. 

Because Frame blades do not use Ibiza Fx controls, extension developers are fully responsible for accessibility, theming, and consistency.

While this programming model results in maximum flexibility, it  also adds a significant burden of accessibility, theming, and consistency on the developer.   We recommend using Frame blades under the following conditions.

* An existing web experience needs to be migrated to Ibiza without being re-implemented
* An existing web experience needs to be hosted in many environments where Ibiza is just one of the hosts
* Developers want to implement user interactions and experiences that are not supported by Ibiza Framework components. For example, you need to build a very rich, custom UX that is not likely to be reused across services.

When using AppBlade, developers are responsible for the following.

* Accessibility

    Making the blade accessible, as specified in [portalfx-accessibility.md](portalfx-accessibility.md)

* Theming

    The extension's UI should always reflect the user's currently selected theme, and should react dynamically when the user changes themes

* Consistent Look & feel

    Designing a visual experience that is consistent with the rest of Ibiza

* Controls

    Building your own controls, or using available alternatives to Ibiza Fx controls

To create a FrameBlade, you need to create 3 artifacts.

**NOTE**: In this discussion, `<dir>` is the `SamplesExtension\Extension\` directory, and  `<dirParent>`  is the `SamplesExtension\` directory, based on where the samples were installed when the developer set up the SDK. If there is a working copy of the sample in the Dogfood environment, it is also included.

1. Register the FrameBlade with your extension by creating a TypeScript class with the @FrameBlade decorator. The samples extension file for this is located at   `<dir>/Client/V2/Blades/FrameBlade/SampleFrameBlade.ts` and in the following example.

code sample coming soon to SamplesExtension in D:\ws\Ship-Sync-AuxDocs-Github\doc\portal-sdk\Samples\SamplesExtension\Extension\Client\V2\Blades\FrameBlade\SampleFrameBlade.ts

2. Create an html page that will serve as the main contents of your iframe.  The samples extension file for this is located at `<dir>/Content/SamplesExtension/framebladepage.html` and in the following example.

```html
﻿<!DOCTYPE html>
<html>

<head>
    <title></title>
    <meta charset="utf-8" />
</head>

<body>
    <h1 class="fxs-frame-header" style="margin: 0;"></h1>
    <div class="fxs-frame-token"></div>
    <input type="text"/>
    <script src="../Scripts/_oss/q-1.4.1.js"></script>
    <script>var frameSignature = "FxFrameBlade";</script>
    <script src="../Scripts/framepage.js"></script>
</body>

</html>

```

3. Create a script that will communicate with your extension by using post messages. This is how your extension can get the auth token, respond to theme changes, and other tasks. The samples extension file for this is located        is located at  `<dir>/Content/Scripts/framepage.js`, and is also in the following example.

```js
(function() {
    "use strict";

    // ---------------------------------------------------------------------------------------------
    // ------------------------------------- Helper Functions --------------------------------------
    // ---------------------------------------------------------------------------------------------

    // var frameSignature = ...;  Defined by .html page that loaded this script.

    // Capture the client session ID to use to correlate user actions and events within this
    // client session.
    var sessionId = location.hash.substr(1);

    var queryMap = (function() {
        var query = window.location.search.substring(1);
        var parameterList = query.split("&");
        var map = {};
        for (var i = 0; i < parameterList.length; i++) {
            var pair = parameterList[i].split("=");
            map[decodeURIComponent(pair[0])] = decodeURIComponent(pair[1]);
        }
        return map;
    })();

    function getQueryParameter(name) {
        return queryMap[name] || "";
    }

    function postMessageToParent(kind) {
        window.parent.postMessage({
            signature: frameSignature,
            kind: kind
        }, trustedParentOrigin);
    }

    // ---------------------------------------------------------------------------------------------
    // --------------------------------------- Security Code ---------------------------------------
    // ---------------------------------------------------------------------------------------------

    // Get the below trusted origins from configuration to include the origin of the portal in
    // which the page needs to be iframe'd.
    var allowedParentFrameAuthorities = ["localhost:55555", "portal.azure.com"];

    // Capture the origin of the parent and validate that it is trusted. If it is not a trusted
    // origin, then DO NOT setup any listeners and IGNORE messages from the parent/owner window
    var trustedParentOrigin = getQueryParameter("trustedAuthority");
    var isTrustedOrigin = (function() {
        var trustedAuthority = (trustedParentOrigin.split("//")[1] || "").toLowerCase();

        return allowedParentFrameAuthorities.some(function(origin) {
            // Verify that the requested trusted authority is either an allowed origin or is a
            // subdomain of an allowed origin.
            return origin === trustedAuthority
                || (trustedAuthority.indexOf("." + origin) === trustedAuthority - origin - 1);
        });
    })();

    // TODO: Uncomment below code to prevent untrusted origins from accessing the site.
    // if (!isTrustedOrigin) {
    //     var errorMessage = "The origin '" + trustedParentOrigin + "' is not trusted.";
    //     console.error(sessionId, errorMessage);
    //     throw new Error(errorMessage);
    // }

    // ---------------------------------------------------------------------------------------------
    // -------------------------------- Handshake Code with Portal ---------------------------------
    // ---------------------------------------------------------------------------------------------

    window.addEventListener("message", function(evt) {
        // It is critical that we only allow trusted messages through. Any domain can send a
        // message event and manipulate the html.
        if (evt.origin.toLowerCase() !== trustedParentOrigin) {
            return;
        }

        var msg = evt.data;

        // Check that the signature of the message matches that of frame parts.
        if (!msg || msg.signature !== frameSignature) {
            return;
        }

        // Handle different message kinds.
        if (msg.kind === "frametitle") {
            makeViewPresentableToUser(msg);
        } else if (msg.kind === "framecontent") {
            document.getElementsByClassName("fxs-frame-content")[0].innerText = msg.data;
        } else if (msg.kind === "getAuthTokenResponse") {
            document.getElementsByClassName("fxs-frame-token")[0].innerText = "Token: " + msg.data;
        } else {
            console.warn(sessionId, "Message not recognized.", msg);
        }
    }, false);

    // ---------------------------------------------------------------------------------------------
    // -------------------------------- Code to reveal view to user --------------------------------
    // ---------------------------------------------------------------------------------------------

    function makeViewPresentableToUser(msg) {
        document.getElementsByClassName("fxs-frame-header")[0].innerText = msg.data;
        document.head.getElementsByTagName("title")[0].innerText = msg.data;

        // Post message 'revealcontent' to the parent to indicate that the part is now in a state to
        // dismiss the opaque spinner and reveal content.
        postMessageToParent("revealcontent");

        completeInitialization();
    }

    // ---------------------------------------------------------------------------------------------
    // ------------------------------ Code to complete initialization ------------------------------
    // ---------------------------------------------------------------------------------------------

    function completeInitialization() {
        // Mimic an async operation that takes 2 seconds.
        Q.delay(2000).then(() => {
            // Post message the 'initializationcomplete' to the parent to indicate that the part is
            // now ready for user interaction.
            postMessageToParent("initializationcomplete");
        });
    }

    // Send a post message indicate that the frame is ready to start initialization.
    postMessageToParent("ready");

    // This is an example of posting the 'getAuthToken' event to Portal.
    postMessageToParent("getAuthToken");
})();

```

<a name="changing-ui-themes"></a>
## Changing UI themes

When using a FrameBlade, extension developers can implement themes. Typically, the user selects a theme, which in turn is sent to the UI IFrame. The following code snippet demonstrates how to pass the selected theme to the UI IFrame using the **postMessage** method.

```typescript

// Get theme class and pass it to App Blade
MsPortalFx.Services.getSettings().then(settings => {
    const theme = settings["fxs-theme"];
    theme.subscribe(container, theme =>
        this.postMessage(new FxAppBlade.Message("theme", theme.name))
    ).callback(theme());
});

```

On the iframe side you can respond to the message just like you would respond to the auth token message. You can then adjust your css accordingly.