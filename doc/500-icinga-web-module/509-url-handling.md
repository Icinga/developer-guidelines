
# URL handling

Those who has keep a watch on how their browser behaves, may have noticed that not every click reloads the page. Icinga Web 2 intercepts all requests and sends them separately via XHR request. On the server side, this is detected, and then only the respective HTML snippet is sent as a response. The response usually only matches the output created by the corresponding view script.

Yet, each link remains a link and can be e.g. opened in a new tab. There it is recognized that this is in fact not an XHR request and the entire layout is delivered.

Usually, links always open in the same container, but you can influence the behavior with `data-base-target`. The attribute closest to the clicked element wins. If you want to override the `_next` for a section of the page, simply set `data-base-target="_self"` on the element.