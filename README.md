#Mock Browser
- - -

[![NPM version](https://badge.fury.io/js/mock-browser.svg)](http://badge.fury.io/js/mock-browser) [![Build Status](https://travis-ci.org/darrylwest/mock-browser.svg?branch=master)](https://travis-ci.org/darrylwest/mock-browser) [![Dependency Status](https://david-dm.org/darrylwest/mock-browser.svg)](https://david-dm.org/darrylwest/mock-browser)

## Overview

A mock browser with window, document, location, navigation, local and session storage to use for client side code tests in a node environment.  A majority of the implementation is from the [jsdom project](https://github.com/tmpvar/jsdom) with enhancements for local and session storage plus some fixes for un-implemented document features (classList).

The mock browser eliminates the need for a headless browser like phantomjs to provide a much faster unit test framework.  It's perfect for browserify projects that run tests prior to compiling the bundle.

## Installation

~~~
    npm install mock-browser --save
~~~

If you just use the mock for testing then '--save-dev' is more appropriate.  But, the AbstractBrowser could (should) be used for run time to replace client logic that normally accesses globals provided by the local browser.  This not only enhances testing and mocking but also enables cross-browser workarounds through interception rather than override.

## Use

### Mock Browser

The use case is you have a browserify client project and unit tests need to provide a mechanism for client modules to access the window, document, location, etc.  The best way to simulate this is to use a proxy wall between the normally global browser objects a require the application to get instances of window, document, etc.  That way you can supply a mock for all of these objects with a common interface (AbstractBrowser).  It looks like this...

~~~
	var MockBrowser = require('mock-browser').mocks.MockBrowser;
	var mock = new MockBrowser();
    
    // and in the run-code
    
    var doc = mock.getDocument();
    var div = doc.createElement('div');
    
    var localStorage = doc.getLocalStorage();
    localStorage.setItem('mykey', 'my value');
    assert localStorage.getItem('mykey') === 'my value';
~~~

### Mock Browser Object Methods

Two convenience methods are added to make it easy to get either a window or dom document mock.

* MockBrowser.createDocument()
* MockBrowser.createWindow()

### Abstract Browser

The AbstractBrowser object can be used as an interface for run-time client apps to access browser window related objects.  This approach enables code to be used both in and outside the browser.

~~~
	var AbstractBrowser = require('mock-browser').delegates.AbstractBrowser;
    
	// configure in some factory
	var opts = {};
    opts.window = (typeof window === 'object') ? window : MockBrowser.createWindow();
    
    // create the browser object with a real window in brwosers and mock when not in browser
	var browser = new AbstractBrowser( opts );
    
    var doc = browser.getDocument();
    var element = doc.getElementById('my-dom-element');
~~~

### Abstract Browser API

Instance methods...

* getDocument()
* getWindow()
* getLocation()
* getHistory()
* getLocalStorage()
* getSessionStorage()

Object Methods...

* AbstractBrowser.extend(child, options)  <- extend the abstract object

These methods are inherited by MockBrowser.

### Extending AbstractBrowser

For run-time use you can extend AbstractBrowser to inherit the API.  This enables attaching other object to the browser object rather than using globals.

~~~
	var AbstractBrowser = require('mock-browser').delegates.AbstractBrowser;
    
    var MyBrowser = function(options) {
    	var browser = this,
        	builder = options.componentBuilder;
        
        // inherit getWindow(), getDocument(), getLocation(), getHistory(),
        // getLocalStorage(), getSessionStorage()
        AbstractBrowser.extend( this, options );
        
        // my browser API extension...
        this.getComponentBuilder = function() {
        	return builder;
        };
    };
~~~

### Mock Storage

The MockStorage object is used to mock out local and session storage.  When  used with mock browser, they are attached to the window object and can be accessed from AbstractBrowser instances by invoking getLocalStorage() or getSessionStorage().

This object may also be used standalone when you just need to mock out either local or session storage.  It looks like this:

~~~
	var MockStorage = require('mock-browser').mocks.MockStorage;
    
    var storage = new MockStorage();
    
    storage.setItem( 'mykey', 'my string value' );
    assert storage.getItem( 'mykey' ) === 'my string value';
    
    assert storage.length === 1;
~~~

## Tests

All objects are tested using mocha.  You can run tests by doing this:

~~~
	make test
    
    // or
    
    npm test
~~~

- - -
<p><small><em>copyright © 2014 rain city software | version 0.90.11</em></small></p>
