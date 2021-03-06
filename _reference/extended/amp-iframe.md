---
layout: page
title: amp-iframe
order: 10
---

<!---
Copyright 2015 The AMP HTML Authors. All Rights Reserved.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS-IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

-->



Displays an iframe.

`amp-iframe` has several important differences from vanilla iframes that are designed to make it more secure and avoid AMP files that are dominated by a single iframe:

- `amp-iframe` may not appear close to the top of the document (except for iframes that use `placeholder` as described below). They must be either 600px away from the top or not within the first 75% of the viewport when scrolled to the top – whichever is smaller. NOTE: We are currently looking for feedback as to how well this restriction works in practice.
- They are sandboxed by default. [Details](https://github.com/ampproject/amphtml/blob/master/extensions/amp-iframe/#sandbox)
- They must only request resources via HTTPS or from a data-URI or via the srcdoc attribute.
- They must not be in the same origin as the container unless they do not allow `allow-same-origin` in the sandbox attribute. See the doc ["Iframe origin policy"](https://github.com/ampproject/amphtml/blob/master/extensions/amp-iframe/../../../spec/amp-iframe-origin-policy.md) for further details on allowed origins for iframes.

Example:
{% highlight html %}
<amp-iframe width=300 height=300
    sandbox="allow-scripts allow-same-origin"
    layout="responsive"
    frameborder="0"
    src="https://foo.com/iframe">
</amp-iframe>
{% endhighlight %}

#### Attributes

##### src, srcdoc, frameborder, allowfullscreen, allowtransparency

The attributes above should all behave like they do on standard iframes.

##### sandbox

Iframes created by `amp-iframe` always have the `sandbox` attribute defined on them. By default the value is empty. That means that they are "maximum sandboxed" by default. By setting sandbox values, one can opt the iframe into being less sandboxed. All values supported by browsers are allowed. E.g. setting `sandbox="allow-scripts"` allows the iframe to run JavaScript, or `sandbox="allow-scripts allow-same-origin"` allows the iframe to run JavaScript, make non-CORS XHRs, and read/write cookies.

If you are iframing a document that was not specifically created with sandboxing in mind, you will most likely need to add `allow-scripts allow-same-origin` to the `sandbox` attribute and you mights need to allow additional capabilities.

Note also, that the sandbox applies to all windows opened from a sandboxed iframe. This includes new windows created by a link with `target=_blank` (Add `allow-popups` to allow this to happen). Adding `allow-popups-to-escape-sandbox` to the `sandbox` attribute, makes those new windows behave like non-sandboxed new windows. This is likely most of the time what you want and expect. Unfortunately, as of this writing, `allow-popups-to-escape-sandbox` is only supported by Chrome.

See the [the docs on MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/iframe#attr-sandbox) for further details on the sandbox attribute.

#### Iframe Resizing

An `amp-iframe` must have static layout defined as is the case with any other AMP element. However,
it's possible to resize an `amp-iframe` in runtime. To do so:

1. The `amp-iframe` must be defined with `resizable` attribute;
2. The `amp-iframe` must have `overflow` child element;
3. The IFrame document has to send a `embed-size` request as a window message.

Notice that `resizable` overrides `scrolling` value to `no`.

Example of `amp-iframe` with `overflow` element:
{% highlight html %}
<amp-iframe width=300 height=300
    layout="responsive"
    sandbox="allow-scripts allow-same-origin"
    resizable
    src="https://foo.com/iframe">
  <div overflow tabindex=0 role=button aria-label="Read more">Read more!</div>
</amp-iframe>
{% endhighlight %}

Example of IFrame resize request:
{% highlight javascript %}
window.parent.postMessage({
  sentinel: 'amp',
  type: 'embed-size',
  height: document.body.scrollHeight
}, '*');
{% endhighlight %}

Once this message is received the AMP runtime will try to accommodate this request as soon as
possible, but it will take into account where the reader is currently reading, whether the scrolling
is ongoing and any other UX or performance factors. If the runtime cannot satisfy the resize events
the `amp-iframe` will show an `overflow` element. Clicking on the `overflow` element will immediately
resize the `amp-iframe` since it's triggered by a user action.

Here are some factors that affect how fast the resize will be executed:

- Whether the resize is triggered by the user action;
- Whether the resize is requested for a currently active IFrame;
- Whether the resize is requested for an IFrame below the viewport or above the viewport.

#### Iframe with Placeholder
It is possible to have an `amp-iframe` appear on the top of a document when the `amp-iframe` has a `placeholder` element as shown in the example below.

{% highlight html %}
<amp-iframe width=300 height=300
   layout="responsive"
   sandbox="allow-scripts allow-same-origin"
   src="https://foo.com/iframe">
 <amp-img layout="fill" src="https://foo.com/foo.png" placeholder></amp-img>
</amp-iframe>
{% endhighlight %}

- The `amp-iframe` must contain an element with the `placeholder` attribute, (for instance an `amp-img` element) which would be rendered as a placeholder till the iframe is ready to be displayed.
- Iframe readiness can be known by listening to `onload` of the iframe or an `embed-ready` postMessage which would be sent by the Iframe document, whichever comes first.

Example of IFrame embed-ready request:
{% highlight javascript %}
window.parent.postMessage({
  sentinel: 'amp',
  type: 'embed-ready'
}, '*');
{% endhighlight %}
