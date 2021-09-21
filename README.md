

# Javascript Event Handling Patterns

Event handling in javascript is a crucial technique as all user-interactions are based on events and handling user interactions is one of the key purposes of javascript.

Events are literally actions that happen during the runtime of a program. In a graphical user interface driven program these actions often are user actions that where exposed to the program as mouse events such as `click`, `mousemove`, `mouseenter` and `mouseleave`. There are dozens of different events that are detected by the javascript engine but for this post I want to stay with the click event that occurs whenever a mouse click happens. The things behave very similar on any other event type.

For each user (or other) action that is recognized as event an **event object** is created. This event object holds lots of event-related information. It's lifetime does not exceed the duration of the action itself, so the event is very fleeting.

The main purpose of events is that we can catch them and perform well-defined instructions on their occurance. In other words we can handle these events. This is what makes a program interactive.

In javascript we can distinguish between an older approach referred as **event handlers** and a more modern implementation referred as **event listeners**.

**Event Handlers**
An event handler is a property on an object (most likely an HTMLElement-Object) that can get a function assigned to it. Whenever the corresponding action occurs, this function will be executed. One key characteristic of an **event handler** is that only one function can be assigned at a given time because we only have this one (atomic) property per event.

	document.querySelector('body').onclick = function() {
		console.log('I was clicked');
	}

If we would assign another function to the `onclick` event handler we would overwrite the previous one.

**Event Listeners**
Event listeners work a bit different. We assign a listener to an event with a function called `addEventListener`.  This function can be called multiple times and instead of overwriting the previously assigned listener, the new listeners will be added. So listeners are stored in a collection whereas handlers are stored in an atomic property.

	document.querySelector('body').addEventListener('click', function() {
		console.log('I was clicked');
	})

	document.querySelector('body').addEventListener('click', function() {
		console.log('I was clicked');
	})

In this example we would get the message `I was clicked` twice because both event listeners are processed when the event occurs.

In the following text I will focus on the **event listeners** as they are the much modern approach and way more flexible than the older **event handlers**.

I will cover

- assignment and removal of event listeners
- separation of the listener function and the business logic (the application-specific code)
- execution context
- scope

As we saw, adding one or more event listeners to an element is pretty easy, but sometimes we want to remove those listeners.
For example if we want to drag an element, we modify the element's position when moving the mouse (`mousemove`-event) according to the change of the mouse position. But normally we only want to do this when the mouse is pressed and not when the mouse is not pressed. So we could add a `mousemove`event listener when the mouse is pressed (`mousedown`-event) and remove it when the mouse is released (`mouseup`-event). By doing this, the dragging action is only performed as long as the mouse button is pressed. Surely we could get this result with other techniques like setting a variable to `true` when the mouse is pressed and setting it to `false` when the mouse is released and validate this variable within the alway assigned `mousemove` listener function.

But this means that we alway had a custom function running that is triggered by the moving mouse even if it hardy would perform a real action, because this would only happen on a dedicated dragging operation. This does not seem to be very efficient, whereas having the event listener function only assigned when needed seems to be the more elegant way. This means that we not only have to assign event listeners to elements but also have to remove them.

**How to remove event listeners**

Whereas we could simply set the event-related property (in the event handler example above it is  the `onclick` property of the html-element) to `null` when dealing with **event handlers**, the removal of an event listener function is done by calling a dedicated method named `removeEventListener` and addressing the previously assigned listener function that we now want to remove.

In consequence we need **access** to the listener function when we want to remove it. This can be more tricky than  expected at first sight.

When we look at the first event listener example above, we see that we deal with **anonymous** functions as listeners. So these functions are not accessible at all, because they have not been assigned to a variable or an object's property. So we don't have an identifier for them. Thus we cannot reference them and cannot remove them!

**Use a pre-defined function**
The classical and most easy way to deal with this issue is to reference a pre-defined function as the listener function.

	function businessLogic(e) {
		console.log('I was clicked');
		console.log('Event catching object: ', this);
		console.log('Event object: ', e);
	}

	document.querySelector('body').addEventListener('click', businessLogic);

	// or

	businessLogic = function(e) {
		console.log('I was clicked');
		console.log('Event catching object: ', this);
		console.log('Event object: ', e);
	}

	document.querySelector('body').addEventListener('click', businessLogic);

It's important to notice that we do not **call** the function `businessLogic` in the `addEventListener`-method, but just **reference** it. If we would **call** it there, it would immediately being executed, but not being executed on event occurance.

As we now have a named function assigned as the listener function we can remove it by referencing it by it's name.

	document.querySelector('body').removeEventListener('click', myListenerFunction);

Surely we have to consider scope. So the function must be accessible from where we want to add and remove it.

A drawback of this technique is, that we cannot pass arguments to the listener function, as we do not call it explicitly. Instead this function is called internally whenever  the corresponding event occurs.
Luckily the most important data will be passed in: This is on the one hand the **event catching element** (the element where the listener is registered) and on the other hand the **event object**. The event catching element can be accessed via the `this` keyword in the listener function and the event object is passed as the first and only argument to the listener function.

But sometimes we want to pass more information from the event catching scope (the scope in which the assignment of the listener function is defined) to the listener function, that itself might reside within another scope (in the following example it's the global scope) where we don't have access to the event catching scope.

That's why the approach with the anonymous function is so popular: As we saw in the first event listener example above this anonymous function is defined directly in place (thus having full access to the event catching scope).
To keep our code clean and modular, we can keep our pre-defined business logic function and actually **call** it from within the anonymous listener function. We can even pass values from the event catching scope to the business logic function.

	// global scope
	function businessLogic(val) {
		console.log('I was clicked');
		console.log('Value from event catching scope: ', val);
	}

	(function(){
		// event catching scope
		let localscopedValue = 'some value';
			document.querySelector('body').addEventListener('click',function() {
			businessLogic(localscopedValue);
		});
	})();

But in this scenario we have to deal with the fact that the pre-defined function `businessLogic` is not the listener function anymore. Instead we again have an anonymous function set as listener function. Like in the first event listener example we cannot later reference this function due to the lack of an identifier. So this allows us to deal with local scoped variables while again introducing the reference problem and the inability to remove the listener function.

But luckily there are some ways to overcome this limitations.

**Give the anonymous function a name**

Instead of using an anonymous function we can use a **named function** that is defined in place as the event listener. Again we could either have the business logic directly inside the listener function or have it in another dedicated business logic function as in the following example.

	function businessLogic() {
		console.log('I was clicked');
	}

	document.querySelector('body').addEventListener('click', function listenerFunction() {
		businessLogic();
	});

In this case we have a named function that we generally can access later. But as it is defined directly in place, the scope where we can access this function is limited to the **event-assigning construction**, so in the end it is limited to the listener function's body itself. It's even not possible to access it within our business logic function unless we pass the identifier explicitly to this function as an argument.

	function businessLogic() {
		console.log('I was clicked');
	}

	document.querySelector('body').addEventListener('click', function listenerFunction() {
		businessLogic();
		this.removeEventListner('click', listenerFunction);
	});

	// or

	function businessLogic(lf) {
		console.log('I was clicked');
		document.querySelector('body').removeEventListener('click', lf);
	}

	document.querySelector('body').addEventListener('click', function listenerFunction() {
		businessLogic(listenerFunction);
	});

The only situation where this is useful is when we have some kind of condition that we can check in place to decide if the listener should be removed or not. But in many cases we more likely will have another action that results in the removal of the listener (just think of the dragging example where the removal of the `mousemove`-listener happens on the `mousup`-event).

	document.querySelector('#button-1').addEventListener('click', function listenerFunction() {
		console.log('I was clicked');
	});

	document.querySelector('#button-2').addEventListener('click', function() {
		document.querySelector('#button-1').removeEventListener('click', listenerFunction);
	});

But in this example the listener function `listenerFunction` is not accessible from the second button's event listener.

We can solve this with an **in-place-assignment**. Instead of using a named function, we use a function that we assign to a pre-defined variable:

**Assign the anonymous function to a variable**

	let listenerFunction;

	document.querySelector('#button-1').addEventListener('click', listenerFunction = function() {
		console.log('I was clicked');
	});

	document.querySelector('#button-2').addEventListener('click', function() {
		document.querySelector('#button-1').removeEventListener('click', listenerFunction);
	});

Here we have defined the variable for the listener function globally and later on assigned our listener function in place to that variable. Because the variable was defined globally, it is accessible from everywhere and thus also from the second event listener function.

When designing our program we can carefully select the scope for this variable and keep it as narrow as possible.

So the solution-bringing good part of javascript in this case is the **ability to pass an anonymous function as an argument to another function while at the same time assigning this anonymous function to a pre-defined variable**.

But for completion sake I will modify the above example a little bit: Instead of assigning the listener function when defining it in place, it is possible to reference it from inside the listener function's body with the function's object property `arguments.callee` and assign it to our predefined variable.

	let listenerFunction;

	document.querySelector('#button-1').addEventListener('click', function() {
		listenerFunction = arguments.callee
		console.log('I was clicked');
	});

	document.querySelector('#button-2').addEventListener('click', function() {
		document.querySelector('#button-1').removeEventListener('click', listenerFunction);
	});

However I think the first approach is the cleaner way, so I would recommend that.

**Another approach**

Another approach is to pass a **function call** rather than a **function reference** as the second argument in the `addEventListener`-method. As said before this would result in the immediate execution of this function, so this function can not act directly as the listener function but it can return the function that is meant to be the listener function.

	// global scope
	function getListenerFunction(val) {
		return function(e) {
			console.log('I was clicked');
			console.log('Event catching object: ', this);
			console.log('Event object: ', e);
			console.log('Value from event catching scope: ', val);
		}
	}

	(function(){
		// even catching scope
		let localscopedValue = 'some value';
		document.querySelector('body').addEventListener('click', getListenerFunction(localscopedValue));
	})();

So in this example only the assignment of the function to execute on event occurance is performed immediately whereas the execution of the returned listener function itself happens only on event occurance. This allows us to pass values from the initial scope to the scope of the event listener function, while remaining it's default execution context. So the event listener (the retuned function) is called on the element on which the listener is registered and it gets passed the event object as first and only argument as we have seen in the **Use a pre-defined function** example. The benefit is that we can pass additional values (i. e. scoped variables to the predefined listener function.

To access and remove this – in the above example anonymous – listener function we again need to assign the returned function to a well-scoped variable either by assigning the function directly or by assigning it from inside it's body with `arguments.callee` . Again I would recommend the first approach.

	// global scope
	let listenerFunction;

	function getListenerFunction(val) {
		listenerFunction =  function() {
			console.log('I was clicked');
			console.log('Event catching object: ', this);
			console.log('Event object: ', e);
			console.log('Value from event catching scope: ', val);
		}
		return listenerFunction;
	}

	(function(){
		// event catching scope
		let localscopedValue = 'some value';
		document.querySelector('#button-1').addEventListener('click', getListenerFunction(localscopedValue));
	})();

	(function(){
		// event removal scope
		document.querySelector('#button-2').addEventListener('click', function() {
			document.querySelector('#button-1').removeEventListener('click', listenerFunction);
		});
	})();

	// or

	// global scope
	let listenerFunction;

	function getListenerFunction(val) {
		return function() {
			console.log('I was clicked');
			console.log('Event catching object: ', this);
			console.log('Event object: ', e);
			console.log('Value from event catching scope: ', val);
			listenerFunction =  arguments.callee;
		}
	}

	(function(){
		// event catching scope
		let localscopedValue = 'some value';
		document.querySelector('#button-1').addEventListener('click', getListenerFunction(localscopedValue));
	})();

	(function(){
		// event removal scope
		document.querySelector('#button-2').addEventListener('click', function() {
			document.querySelector('#button-1').removeEventListener('click', listenerFunction);
		});
	})();

**Calling context**

Finally we have to look at the calling context. As stated in **Use a pre-defined function** the default behavior is that the listener function is called on the element on which the listener is registered. This means that the `this` keyword refers to this element. In addition the event object is passed in as the first and only argument to the listener function. This object provides many useful event- and element-related values. So in many cases this will satisfy our needs.

If we assign an anonymous function to the listener (as done in **Assign the anonymous function to a variable**) and call another function that holds our business logic from within this listener function it is our responsibility to pass the execution context to the business logic function (what is not mandatory but recommended).

	function businessLogic() {
		console.log('I was clicked');
		console.log('Event catching object: ', this); // window
		console.log('Event object: ', e); // undefined
	}

	let listenerFunction;

	document.querySelector('#button-1').addEventListener('click', listenerFunction = function(e) {
		businessLogic();
	});

So in this case the listener function – what is the in place defined anonymous function – is called on the event catching element, so the `this` keyword refers to the element where the listener is registered. In addition we have access to the event object via the first and only argument that is passed in. That's why we normally declare an `e` or `event` argument to access this event object in a convenient way.
The business logic function in contrast is executed on the window object and it does not know anything about the event object as it is not passed in.
To fix this we can pass in the event object as an argument and set the event-catching element as the base object for the execution of the business logic function. This is done with the following line

	businessLogic.call(this, e);

So the complete example looks like this:

	function businessLogic(e) {
		console.log('I was clicked');
		console.log('Event catching object: ', this); // button#button-1
		console.log('Event object: ', e); // event object
	}

	let listenerFunction;

	document.querySelector('#button-1').addEventListener('click', listenerFunction = function(e) {
		businessLogic.call(this, e);
	});

	document.querySelector('#button-2').addEventListener('click', function() {
		document.querySelector('#button-1').removeEventListener('click', listenerFunction);
	});

As functions are objects in javascript, they can have properties and methods as every other object can have. One pre-defined method is the method `call`. This can be used to set the execution context by passing it in as the first argument. In this example we set the event's catching element (that we can refer with the `this` keyword within the listener function) as the execution context by passing it in as the first argument. The second argument is the event object. And that's it. Within the business logic function we now also can access the event-catching element with the `this` keyword. The event object is passed in as first argument (though being the second argument in the call-method). Again for convenience reasons we give this argument a name `e` or `event`.

**Conclusion**

This last example is my recommended way to
- register event listeners on html-elements,
- separate  listener functions from business logic functions (for better reusability)
- execute business logic functions within the same context as the listener functions
- pass event catching scoped variables to the more globally defined business logic function
- assign listener function to well-scoped variable for later access i. e. for removal of the listener.
