#Common Pitfalls in CanJS
It is often the most powerful parts of a framework that create confusion in the beginning. These elements, when comprehended, often become the favorite tools of the developers using the framework. Angular's directives are a well-known example of this. Directives are very powerful, and often misunderstood. Once grasped, however, they are well loved, and well used.

In CanJS two aspects that can seem a bit mistifying to the new CanJS developer are Observables and Shared Scopes.

##Observables
If you're already familiar with the Observer pattern, you can skip this section.

CanJS applications are component based. A correctly built CanJS app will be composed of several can.Components, carefully integrated together to form a whole. When we use a component-based method of appliction composition, each component  should address a specific, encapsulated abstraction within the problem domain.  This ensures that there is a clear boundary between components (a separation of responsiblities), which makes them easier to test, reuse, and maintain.

While it is important for components to remain separate, and encapsulated, for them to form a unified whole, they must be able to communicate with each other. This is where the Observer pattern comes in. In the Observer pattern, two actors, the Observer and the Subject forge a relationship. The Observer watches the Subject. This means that when a change occurs in the subject, the observer is notified of  this change and can respond accordingly.

To help clarify this concept, let's look at an example. Imagine that we have a site that sells widgets. Our application uses a can.Model.List named "Widgets" to provide a list of all the widgets currently available for sale. Each widget returned by the can.Model.List contains an id, a description, a price, and an isSold property. In order to display this information to the user, the application uses a WidgetDashboard can.Component. The WidgetDashboard shows all of the widgets, and provides a means of selecting Widgets to purchase.

Within this application, the "Widgets" can.Model.List (and the individual Widget objects it contains) is the Subject. WidgetDashboard is the Observer. If a widget is updated (e.g., purchased, edited, or removed), because the WidgetDashbaord is observing the widgets, its state changes are displayed to the user as they occur.

###Implementing an Observer
While we talk about the observer watching the subject, this isn't actually how the pattern works when you implement it. The observer actully registers with the subject. Then, when a state change occurs, the subject notifies the observer of the change.

In a classic Observer pattern, the observer invokes the Register method on the subject, passing itself as an argument. Once the subject receives this reference, it must store it in order to notify the observer when a state change occurs sometime in the future.

CanJS works a little differently. It's a bit easier. To create an observer-subject relationship in CanJS, all you have to do is include a reference to an observable element in the Subject. Let's look at an example, using our ficitious Widget application.

My Widgets model will be defined as follows:

    import can from "can"

    export default can.Model.extend({
            resource: "/services/widgets"
        },
     {});

>As a side note, in the code above, we're using the `resource` property of can.Model. the `resource` property is a very useful shorthand for defining default methods on a can.Model. The code above is equivalent to the longer can.Model definition:

    export default can.Model.extend({
        findAll: "GET /widgets",
        findOne: "GET /widgets/{id}",
        create: "POST /widgets",
        update: "PUT /widgets/{id}",
        destroy: "DELETE /widgets/{id}"
    },{});

>You can use the `resource` property short hand if the objects returned by your service have an id of "id".

The ViewModel for my WidgetDashboard will be defined as follows:

    import can from "can";
    import WidgetModel from "models/widgets";
    import "can/map/define/";
    ... //Other imports

    export var WidgetDashboardViewModel = can.Map.extend({
        define: {
      		widgets: {
            	Value: WidgetModel.List
            }
            widgetsAvailable: {
                get: function () {
                    //Create the Observer-Subject relationship between
                    //WidgetDashboardViewModel, and WidgetModel
                    var widgets = this.attr('widgets');
                    if (widgets) {
                        return widgets.length;
                    }

                }
            }
        },

We create the Observer-Subject relationship between WidgetDashboardViewModel (Observer) and WidgetModel (Subject) when we make reference to the length property of widgets in the widgetsAvailable property's get function. In fact, it is more accurate to say that we've created an Observer-Subject relationship between the widgetsAvailable property's get function (Observer), and the WidgetModel(Subject).  The get function is watching for changes in (i.e., observing) the widgets can.Model.List collection's length.

What this means is that the WidgetDashboardViewModel's widgetsAvailable get function will run if the length property of the WidgetModel.List changes (i.e., an item is added to or removed from WidgetModel.List).

#Understanding The Scope Chain in CanJS
One of the keys to properly constructing a CanJS application is understanding how the CanJS scope chain functions. Understanding the scope chain is essential to effectively:

- Manage application state changes,
- Communicate between can.Components,
- Build powerful, and efficient view templates, and
- Debug

##Scope
Like many other JS frameworks, CanJS makes use of a scope object to share values between a can.Component, and it's view template. A scope is a can.Map object. There are several ways to define a scope for a can.Component:

- Direct,
- ViewModel, and
- Inherited

Both the Direct, and ViewModel methods are component-specific scopes. They belong to the component, as a distinct, encapsulated entity. Both require that you include a `scope` property on the can.Component.

###Direct
To directly define the scope of your can.Component, you provide the implementation of the scope as an anonymous object literal, as follows:

	can.Component.extend({
    	tag: 'my-tag',
   		template: myStacheTemplate,
  		scope: {
        	sampleProperty: 'Sample Property'
        }
    });

While this is possible, and may make sense in certain instances, it is not the generally preferred method.

###ViewModel
Using a ViewModel is the preferred method of implementing a component-specific scope. Because a can.Component's scope is a can.Map, a view model is a can.Map. The advantage of separating the ViewModel from the can.Component is that you can independently test your ViewModel outside of your can.Component.

You can create a ViewModel, and assign it to a can.Component's scope, as follows:

    var SampleViewModel = can.Map.extend({
        define: {
            mySampleProperty: {
                value: 'Sample Property'
            }
        }
    });

	can.Component.extend({
    	tag: 'my-tag',
   		template: myStacheTemplate,
  		scope: SampleViewModel
    });

Calling can.Map.extend creates a constructor function for a can.Map. However, in most cases, you will not create an instance of the ViewModel and assign it to the scope. When you assign the constructor function to the scope, CanJS will instantiate the ViewModel automatically. If you were to provide an instance of the ViewModel to the scope, that instance would be shared amongst all instances of the can.Component. By providing a constructor function, rather than an instance, each time the can.Component is created, it will receive its own instance of the ViewModel you defined.

###Inherited
can.Components can automatically inherit values on their scope from either the Application State, or the scope of a parent can.Component. This is one of the areas of CanJS that can get tricky, because a can.Component can have several objects or properties available to its scope that are not defined anywhere on the can.Component itself. Furthermore, when values are passed to a can.Component it is done using the view template. There is no indication anywhere in code that there is a connection between a can.Component and its parent scope.




