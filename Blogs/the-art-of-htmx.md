# The Art of HTMX
---
<quick-description>
</quick-description>
---
Why use JSON to send UI information to a client that understands and renders HTML? One can argue that the data would contain context regarding which is what, but such an argument falls when the pure object of the JSON data is to be displayed on the screen to the user as HTML.

There is an extra step involved when we use JSON for UI, as we can see in the diagram below:
![[the-art-of-htmx-img-1.png]]

Here, we can visually see the extra step (step 2) where the JSON is being decoded and re-encoded in HTML for the user to see that content.

If instead of sending JSON/XML from the server (step 1), we send HTML directly from the server, we can render the UI contents directly for the user and remove the translation step. To understand this requirement of removal, we can say that the browser now no longer can have to hold a dictionary to translate these UI information.

This raises 2 more question:
1. Can the server even talk in HTML without an overhead? Most languages gives support for JSON & XML and they are much more conventionally faster than their HTML counterpart.
2. What if some content of the data is not for the UI, but for some other activity? How would we integrate them into the server API response?

## 1. Can the server even talk in HTML without an overhead?
The answer to this is *possibly no*. Usually, servers creating JSON over HTML is faster due to the lack of overhead of *context-aware escapes* that is required for HTML for XSS safety.

This however raises another question in me - would we require this overhead if the data is completely under our control. Most usually, the data that we use to create our UI comes from the our databases - where the content is already been sanitized for any form of contamination.

In such a case, we can use a library/package that allows us to avoid the *heavy escaping* that is done. For example, in GoLang - the `net/html` is the *AST* package for creating HTML nodes without the extra escaping. GoLang also provides a templating option using `html/template` package with additional feature to bypass the context checks using `template.HTML`.

As such, we can say that the usages of HTML as the response is valid if one of the following is satisfied:
1. The speed of creating the HTML template isn't as significant as creating the JSON - this can also be optimized via caching the HTML nodes that changes the least.
2. The UI data is completely under the control of the server and no XSS issue may arise from server-side rendering.

## 2. What if some context of the data is not for the UI, but for some other activity?
It can so happen that some of the data that we send to the client end, isn't required for UI display, but for how some UI items behave. This can be themes, animation or even the strategy a button click can use to perform an action. In such a case I believe we can say the following:

1. If the behaviour/display of the UI is dependent on some context present in the JSON - we can see that the client side code would watch this contextual data and apply the required animation/ui class to the HTML nodes. If such is the case, why not simply send these information in the HTML node from the server with the required class-names/animations? This would get rid of the extra step on the client.
2. If the strategy of an user action is decides based on some context from server - for example: to use faster word processing API for a premium user, or to use client side word processing for a standard user? In such a case, how a client sided would works is:
	1. The context would be read and the required pre-loaded method will be called.
	2. Or, the context would be read and the required script would be fetched and executed.
	We can avoid this, by simply supplying the strategy as JS script that needs to be executed. A premium user would not get the details of the standard user and vice-versa.