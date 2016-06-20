# Components and Addons

## Web Components

Web Components are a new mechanism that allows us to extend the DOM
with our own elements rather than limit ourselves to traditional
tags. We can define our own tags, wrapping up all the display logic in a
single bundle, and reuse it between different applications. We'll
use the component as any other tag and the browser will
understand how to render it based on its definition.

Let's examine how a Share on Twitter button works. Currently, we need
to include some JavaScript and then create an anchor tag that
will be transformed by the JavaScript snippet:

{title="Twitter Share Button", lang="html"}
~~~~~~~~
<a class="twitter-share-button"
  href="https://twitter.com/share">
Tweet
</a>
<script type="text/javascript">
window.twttr=(function(d,s,id){var t,js,fjs=d.getElementsByTagName(s)[0];if(d.getElementById(id)){return}js=d.createElement(s);js.id=id;js.src="https://platform.twitter.com/widgets.js";fjs.parentNode.insertBefore(js,fjs);return window.twttr||(t={_e:[],ready:function(f){t._e.push(f)}})}(document,"script","twitter-wjs"));
</script>
~~~~~~~~


The previous code will be the same regardless of the
application we are developing. Sounds like a good candidate for a
component since it's a chunk of code that can be reused across
applications. A possible twitter-button component could look something
like the following:

{title="Twitter Share Button", lang="html"}
~~~~~~~~
<twitter-button>
  Ember.js Rocks!
</twitter-button>
~~~~~~~~

All the implementation details are hidden in its definition.
As consumers, we are only interested in the final product and we needn't worry about how it is accomplished.

Web Components are a great tool that we can use to write more
expressive applications and to avoid code repetition within projects.
Unfortunately, it is not yet supported in all browsers.

To tackle this problem, ember introduced the concept of Components.
This is an API that allows us to write components today by
following the **W3C** specification as closely as possible. The day components become widely available, we'll be
able to switch without hiccups.

I> The official name for Web Components in the **W3C** is
I> (Custom Elements)[http://w3c.github.io/webcomponents/spec/custom/#about].

## ember-cli addons

**ember-cli** has a built-in mechanism that allows us to augment **ember-cli**'s
functionality and share code easily between different applications.
This mechanism is known as addons.

Using addons, we can easily write ember components and share them with
others using npm. Let's create our first component, which will help us
grab an image to use as placeholder in our friends' profiles.

## ember-cli-fill-murray

http://www.fillmurray.com is a service we can use to get random images
of Bill Murray to use as placeholders. Let's write an addon so that we
can do something like the following in any of our templates:

{title="Fill Murray Component", lang="handlebars"}
~~~~~~~~
{{fill-murray width=300 height=300}}
~~~~~~~~

First we need to create the addon. **ember-cli** has a command for this. Outside of our borrowers directory, let's run the following:

{title="Creating an ember-cli addon", lang="bash"}
~~~~~~~~
$ ember addon ember-cli-fill-murray
installing addon
  create .bowerrc
  create .editorconfig
  create .ember-cli
  create .jshintrc
  create .travis.yml
~~~~~~~~

The **addon** command creates a directory very similar to the one
created by **new**, but the former is done in a way that allows it to be
distributed as an addon.

If we go to the directory **ember-cli-fill-murray**, it will look like
 the following:

{title="Addon directory", lang="bash"}
~~~~~~~~
.
|-- LICENSE.md
|-- README.md
|-- addon
|-- app
|-- bower.json
|-- bower_components
|-- config
|-- ember-cli-build.js
|-- index.js
|-- node_modules
|-- package.json
|-- testem.json
|-- tests
+-- vendor
~~~~~~~~

If we open **package.json**, we'll see the following section:

{title="", lang="json"}
~~~~~~~~
  "keywords": [
    "ember-addon"
  ],
~~~~~~~~

That's how **ember-cli** detects the presence of an **addon**. When we
include the library in an **ember-cli** project, it will transverse
the dependencies and identify as **addons** the items with the
keyword **ember-addon**. We'll also use **package.json** to specify
any dependency our library might have.

Next we have **index.js**, which is the entry point for loading our
**addon**. If we need to add any extra configuration for our addon,
we'll specify it in this file. For now let's work with the basic one, which looks like this:

{title="", lang="JavaScript"}
~~~~~~~~
module.exports = {
  name: 'ember-cli-fill-murray'
};
~~~~~~~~

Next we have the directories **app** and **addon**. This is where
the code for our **addon** will live.

Whatever we put into **app** will be merged into our application's
namespace, meaning we'll consume it just as if it were inside our
**ember-cli** project.

For example, if we had an **app/model/friend-base.js** file in our
**addon**, we could consume it in any of the models in our
**borrowers** app thusly:

{title="Consuming an addon's app/model/friend-base.js", lang="JavaScript"}
~~~~~~~~
import FriendBase from './friend-base';

...
~~~~~~~~

If we had put **friend-base.js** into the **addon** directory, instead
of getting merged into the consuming application namespace, it would be
kept under the **addon namespace** with the previous example. If our
**addon** was called **borrowers-base** and we had
**addon/models/friend-base.js**, then we would have consumed it like
this:

{title="Consuming modules from addon's namespace", lang="JavaScript"}
~~~~~~~~
import FriendBase from 'borrowers-base/models/friend-base';

...
~~~~~~~~

Going back to our **ember-cli-fill-murray addon**, we have the directory in
place and we want to distribute a component called **fill-murray**.
Inside the directory, we can also use **ember-cli** generators.
We'll do that in order to create the component:


{title="Bill Murray Component", lang="bash"}
~~~~~~~~
$ ember generate component fill-murray
installing component
  create addon/components/fill-murray.js
  create addon/templates/components/fill-murray.hbs
installing component-test
  create tests/integration/components/fill-murray-test.js
installing component-addon
  create app/components/fill-murray.js
~~~~~~~~

In **addon/components/fill-murray.js**, we can specify the properties for
our component:

{title="addon/components/fill-murray.js", lang="JavaScript"}
~~~~~~~~
import Ember from 'ember';
import layout from '../templates/components/fill-murray';

export default Ember.Component.extend({
  layout: layout,
  height: 100, // Default height and width 100
  width: 100,

  //
  // The following computed property will give us the url for
  // fill-murray. In this case it depends on the properties height and
  // width.
  //
  src: Ember.computed('height', 'width', {
    get() {
      let base = 'http://www.fillmurray.com/';
      let url  = `${base}${this.get('width')}/${this.get('height')}`;

      return url;
    }
  })
});
~~~~~~~~

Next we need to specify the body of our component in
**addon/templates/components/fill-murray.hbs**:

~~~~~~~~
<img src={{src}}>
~~~~~~~~

When rendering the component, it inserts an **img** tag that reads the
source from the computed property we specified.

Our **addon** is now ready. The next step is to distribute it
via npm. First let's change the name in **package.json**
because **ember-cli-fill-murray** is already taken.
We'll use **ember-cli-fill-murray-your-github-nickname** and set
version to **0.1.0**. It will look something like this:

~~~~~~~~
  "name": "ember-cli-fill-murray-your-github-nickname",
  "version": "0.1.0",
~~~~~~~~

Since our addon ships with templates, we need to include
`ember-cli-htmlbars` in the dependencies, which we can do running `npm
i ember-cli-htmlbars --save`.

With the previous in place, let's do **npm publish**.  Our **addon**
is now ready to be consumed.

## Consuming fill-murray in borrowers

Once our package is in **npm**, we can add it to our application by
running the following command:

{title="", lang="bash"}
~~~~~~~~
$ ember install ember-cli-fill-murray-your-github-nickname
~~~~~~~~

Once it is installed, we can consume the component in any of our
templates as follows:

{title="", lang="handlebars""}
~~~~~~~~
{{fill-murray width=200 height=200}}
~~~~~~~~

Let's modify our **app/templates/friends/show.hbs** to look like the
following:

{title="Consuming fill-murray in app/templates/friends/show.hbs", lang="handlebars"}
~~~~~~~~
<div class="clearfix flex p3 h2">
  <div class="">
    {{fill-murray width=200 height=200}}
    <p class="">{{model.fullName}}</p>
    <p class="">{{model.email}}</p>
    <p class="">{{model.twitter}}</p>
    <p>{{link-to "Lend article" "loans.new"}}</p>
    <p class="">{{link-to "Edit info" "friends.edit" model}}</p>
    <p class=""><a href="#" {{action "delete" model}}>Delete</a></p>
  </div>
  <div class="flex-auto ml1 loans-container">
    {{outlet}}
  </div>
</div>
~~~~~~~~

After editing the template, the Bill Murray placeholder will appear
when we visit a friend's profile.

With that we have created and published our first addon.
