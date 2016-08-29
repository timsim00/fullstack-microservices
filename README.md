Based on [this very cool repo](https://github.com/apparatus/fullstack-microservices) by @davidmarkclements.

# Reason for changes

{msg} -> Stager -> Local -> Http

I'm thinking of Stager like a stage manager role in a live play.  A responsibility of that role is to get actors into their proper positions before they are to go on stage.

Stager is responsible for pulling down components that are needed now or may be needed in the near future.  Patterns are registered to it, along with the pattern that corresponds to its component.  For example, this would be called if the activity component is not brought down with the initial app load:  

`stager.add({role:'activity',cmd:'entry'},{role:'activity',cmd:'component'})`

We're just telling the app that 'activity:entry' is a valid msg, but its component hasn’t been brought down yet.  Seneca-mesh is great for identifying potential messages the client needs to know about.

Once the component has been pulled in, the msg is removed from Stager and the msg continues on through local and http.


# Changes 

Here are the changes I made, more or less:

* Started with Spike 2.1
* Patterned the repo after Node-zoo using Senenca-mesh
* Changed the Local transport to act on all pattern matches instead of just one
* Added the Stager transport to polymer-lucius, called before Local and HTTP transports
* Used seneca-mesh to find the messages whose components are not loaded in initial page load
* Imported relevant mesh service patterns into lucius on initial page load
* Commented out link import for the activity service

We end up with the pattern 'activity:entry' with no activity component on the page. When
'service1:action1' returns and runs act() on 'activity:entry', stager sends the
'activity:component' message and imports it.  Then 'activity:entry' is allowed to continue, hitting the local transport.


# Hacky stuff (that I can remember)

* Used mesh pin to identify services that have components using client:'web' and client:'web-scaffolding'. This makes the mesh pin differ from the service patterns.



# TODO

* Needs to be put in a web worker
* Implement a 'standby' functionality to bring components down before they are needed
* Figure out how to bring down multiple (related) components with one message
* Add more test cases


# Thoughts

### What if the client doesn't want the component on the screen yet? 

If the comp is proactively loaded, the layout itself decides if and where the component is loaded. If the activity component is loaded before it's needed, you'll need to add something like:

`
 				var dynamicEl = document.createElement("activity-cmp");
   			document.body.appendChild(dynamicEl);
`
...to the document before 'activity:entry' msg is sent.




----------
# Spike 3

* finding the mu api ideal

## Previous Spikes

# Spike 2.1

* addresses styling
* components use material design polymer components, with neutral styling
* component styles use the [css-apply-rule](https://tabatkins.github.io/specs/css-apply-rule/) to declare expected mixin names, e.g. `.action { @apply(--service2-btn); }`
* the new theme service supplies a component that defines relevant mixins
* theme component can then by imported as a [shared style](https://www.polymer-project.org/1.0/docs/devguide/styling#style-modules) to both the app and other components
  * currently just imported into app
* this approach gives us modular, composable CSS that can be broken down to an app, page, component or element basis
* the only tricky part is knowing the css mixin namespaces of each service, we can handle this by exposing a service action that returns relevant mixin names for a service component, and setting up namespace mixin aggregation service, which allows us to do things like, quickly create a style theme template, check available mixin namespaces against current defined mixing, etc.


# Spike 2

* polymer
* still uses bootstrap classes
* no need for main.js at all! 
* each service has a component folder, containing
  * template.html - the template for the component
  * styles.css - style for the component, obviously supports scoped styling via `:host` - no styles in this spike, will be in next
  * definition.js - defines the component, click handlers etc.
  * index.js - function that returns object with html property (as component.js before), assembles custom element from templates, styles and definition.
* import custom elements from service components
* components add their own actions on instantiation (see activity/component)
* components trigger actions based on user interaction (see service1/component)
* the component itself has `add`, `act` (uses lucius-polymer to do this, may break that into it's own module)

# Spike 1.2

* normalize error handling
  * i.e. all errors come through callback first param, instead of potentially in response objects
* simplify services.js 

# Spike 1.1

* introduces [lucius](http://npm.im/lucius) - browser-side seneca microservices
* removes the need to export script from component (pattern matching etc. replaces this approach)
* enforces idea of microservice pattern matching across browser and server



# Spike 1

* Simplest most common libs possible
* jQuery
* bootstrap
* ignores the styling issue
