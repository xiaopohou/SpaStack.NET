﻿SpaStack.NET
=================


> SpaStack.NET is a Single Page Application (SPA) web boilerplate framework built from `Durandal.js` `JayData.js` `ASP.NET Web API 2 oData` `Twitter Bootstrap` `phonejs` . It allows you to maintain one slim
> codebase . It be package with PhoneGap for native deployments to Android / iPhone / Blackberry / Windows Phone / Browsers / Windows 8 / etc... It follows `RESTful OData MVC` patterns on the server side
> and `MVVM` patterns in the client side.

Table of Contents
------------------
* [Demo and Screenshots](#demo-and-screenshots)
* [Examples of desireable things SpaStack can do](#examples-of-desireable-things-spastack-can-do)
* [Install](#install)
* [How to build an app in one line of code](#how-to-build-an-app-in-one-line-of-code)
* [Frameworks Used](#frameworks-used)
* [How to Create a Mobile Themed app view (Android, iPhone, Windows)](#how-to-create-a-mobile-themed-app-view-for-android-iphone-and-windows-phone))
* [How to Create a PhoneGap Build App](#how-to-create-a-phonegap-build-app)
* [How to create app icons](#how-to-create-app-icons)
* [How to create a Custom binding handler for durandal and knockout](#how-to-create-a-custom-binding-handler-for-durandal-and-knockout)
* [Automated Builds with Weyland](#automated-builds-with-weyland)
* [Autogenerate appcache manifest with Fiddler for offline web](#autogenerate-appcache-manifest-with-fiddler-for-offline-web)
* [How OAuth works in this app](#how-oauth-works-in-this-app)
* [Testing with Jasmine](#testing-with-jasmine)
* [TODO Items](#todo-items)

Demo and Screenshots
--------------------
http://spastack.azurewebsites.net
http://spastack.azurewebsites.net/mobile.html

`Desktop View`

![Screenshot](/SpaStack.NET/Content/images/SPAStack.PNG)

`Android View`

![Screenshot](/SpaStack.NET/Content/images/SpaStackAndroid.png)

`iPhone View`

![Screenshot](/SpaStack.NET/Content/images/SpaStackiPhone.png)

`Windows Phone View`

![Screenshot](/SpaStack.NET/Content/images/SpaStackWinPhone.png)

`Mobile Web View`

![Screenshot](/SpaStack.NET/Content/images/SpaStackMobile.PNG)


Examples of desireable things SpaStack can do
---------------------------------------------
 * Code organization and separation of concerns for large scale javascript development using AMD patterns and best practices such as the revealing module pattern
 * Table and Data Paging using the OData spec
 * Validation
 * Async Promises
 * Login (local, facebook, twitter, etc...) - http://www.asp.net/identity/overview/getting-started/adding-aspnet-identity-to-an-empty-or-existing-web-forms-project
 * Offline - IndexedDB, WebSql, LocalStorage providers
 * $expand OData REST entities
 * MVVM data-bind to observables in your view
 * Dashboards - charting, graphing, grids, forms (uses startbootstrap's dashboard template http://startbootstrap.com/sb-admin)
 * phonejs - for native device specific themeing, android, ios, windows phone

Install
--------
1. Download [here](https://github.com/ntheile/SpaStack.NET/raw/master/SpaStack.NET/SpaStack.NET.zip) and copy the zip file to `C:\Users\yourname\Documents\Visual Studio 2013\Templates\ProjectTemplates`
2. Open Visual Studio 2013 goto File > New Project > C# > You should see a template for SpaStack.NET
3. After the project opens right click on `index.html` and select 'Set as Start Page' in the menu
4. Now build the project to restore all the nuget packages



How to build an app in one line of code
---------------------------------------
> maybe a few more ;)


**STEP 1. Create the server side model (C#)**

First I demonstrate how to create a server side plain old C# object (POCO) representing the data model, in this case a TodoItem. 
I use a Guid as the ID for possible future implementations using a local data store and syncing to the backend database occasionally. 

`model`
```csharp

	public class TodoItem
	{
		[Key]
		public Guid Id { get; set; }
		public String Task { get; set; }
		public Boolean Completed { get; set; }
		public Boolean InSync { get; set; }    
	}

```

**2. Create the Backend**

Next I use an Object Relation Mapping tool (ORM) tool to create the backend for CRUD operations. 
I am a huge fan of the OData protocol for allowing easy access to my data, 
as well as a great mechanism for paging and filtering data. 
It only takes a few steps to create the entire backend for HTTP GET, PUT, POST, PATCH and DELETES. 

Create a `Web Api 2 oData Rest Controller` and use `Entity Framework Code first` to create the database. 
See this great article http://www.asp.net/web-api/overview/odata-support-in-aspnet-web-api/creating-an-odata-endpoint

**3. Create the Frontend Model**

Next I use a rich data library called JayData to create a Front End Model representation of my TodoItem. 
Javascript is a dynamic language and it’s not strongly typed, that’s why I turn to the JayData 
library to help me out, it also automates the ajax calls to the backend. 

Run the Jay Data Service utility to auto create the client side model (JS)
`JaySvcUtil.exe -m http://localhost:65310/odata/$metadata -o App\services\db.js`

`model`
```js

	/*//////////////////////////////////////////////////////////////////////////////////////
	////// Autogenerated by JaySvcUtil.exe http://JayData.org for more info        /////////
	//////                             oData  V3                                     /////////
	//////////////////////////////////////////////////////////////////////////////////////*/
	$data.Entity.extend('SpaStack.NET.Models.TodoItem', {
		'Id': { 'key':true,'type':'Edm.Guid','nullable':false,'required':true },
		'Task': { 'type':'Edm.String' },
		'Completed': { 'type':'Edm.Boolean','nullable':false,'required':true },
		'InSync': { 'type':'Edm.Boolean','nullable':false,'required':true }
	});
	$data.EntityContext.extend('MyDb', {
		'TodoItem': { type: $data.EntitySet, elementType: SpaStack.NET.Models.TodoItem}
	});

	$data.generatedContexts = $data.generatedContexts || [];
	$data.generatedContexts.push(MyDb);

```

**4. Create a data context service layer**

Wire up a `data context` instance on your client (JS)

`datacontext`
```js

	var db = new MyDb({
		name: 'oData',
		oDataServiceHost: '/odata'
	});

	function getTodoItems(observable){
		var promise = db.TodoItem.toArray(observable);
		return promise;
	}
	
```

**5. Wire the model to the view**

Finally we wire the data model to the view. I use the knockout.js library to do this. 
Knockout uses a two-way binding object called an observable that automatically 
binds your data to the screen, if the data in the view model changes so does the view.
This allows for good separation of concerns. It also allows for async data to return and automatically
update on the view.

Consume the data and display it using a knockout observableArray (JS)

`viewmodel`
```js

	var remoteTodos = new ko.observableArray();
	datacontext.getTodoItems(remoteTodos);

```

`view`
```html

	<table class="table table-striped">
		<thead>
			<tr>
				<th>Task</th>
				<th>Is Synchronized</th>
			</tr>
		</thead>
		<tbody data-bind="foreach: remoteTodos">
			<tr>
				<td contenteditable="true" data-bind="text: Task"></td>
				<td data-bind="text: InSync"></td>
			</tr>
		</tbody>
	</table>

```

**6. Write a Test**

We might as well write a piece of test code here. I use Jasmine in this example to write a test to see if we get results 
back from the web service for Todo Items. 

`test`
```javascript

define(['services/datacontext'], function (datacontext) {

    describe("Getting TodoItems in a web service call", function () {
        var ajaxResult = false;
        beforeEach(function (done) {
            datacontext.getTodoItems().done(function() {
                // success
                ajaxResult = true;
                done();
            });
        });
        it("should return true", function (done) {
            expect(ajaxResult).toBe(true);
            done();
        });
    });

});


```

**7. That’s it! In very few lines of code you created an app that's simple, clean, maintainable and testable using best practice like MVC, MVVVM and AMD!**






Frameworks Used
---------------

### Frontend ###


> * JayData.js – rich data management
> * Durandal.js – navigation, app life cycle and View composition
> * Knockout.js – data bindings
> * Require.js – Modularity with AMD and optimization
> * Toastr.js – pop-up messages
> * Twitter Bootstrap – robust CSS styling
> * Phonegap - Interacting with native mobile/tablet API's in javascript
> * jQuery - DOM
> * jQuery.mmenu - responsive side menu
> * phonejs - framework for native mobile themes , android, iPhone, windows phone

### Backend ###


> ASP.NET Web API 2 oData Service

How to Create a Mobile Themed app view for Android iPhone and Windows Phone
----------------------------------------------------------------------------
The phonejs http://phonejs.devexpress.com/ framework is used for this. I chose to make SpaStack have a more Native UI feel using this framework. 
Although the routing and views are handled through phonejs, the same business logic and viewmodels can be reused. 
If you don't want tp write the extra views you could also just use the responsive design that bootstrap 3 offers. The following folders go along
with the phonejs mobile code that can be wrapped up in PhoneGap and deplyed natively.

* `mobile.html` - main view container for mobile applications
* `App\mobile\*` - main views for mobile applcations
* `App\mobileviewmodels\*` - viewmodles that only belong to the mobile app
* `App\viewmodels\*` - shared viewmodels between the mobile and web app
* `App\services\*` - shared services between the mobile and web app
* `App\main-mobile.js`- main file for the mobile app lauched using requirejs

Since durandal is not being used in phonejs you need to wire a few things up to make the app work:

First in the `mobile.html` page add reference to your views using phonejs's dx-templates 

```html
	<!-- Views -->
    <!-- [Add you mobile specific views here...]-->
    <link rel="dx-template" type="text/html" href="App/mobile/home.html" />
    <link rel="dx-template" type="text/html" href="App/mobile/CustomEvents.html" />
    <link rel="dx-template" type="text/html" href="App/mobile/Form.html" />
    <link rel="dx-template" type="text/html" href="App/mobile/Gallery.html" />
    <link rel="dx-template" type="text/html" href="App/mobile/IconSet.html" />
    <link rel="dx-template" type="text/html" href="App/mobile/Lists.html" />
    <link rel="dx-template" type="text/html" href="App/mobile/Maps.html" />
    <link rel="dx-template" type="text/html" href="App/mobile/Navigation.html" />
    <link rel="dx-template" type="text/html" href="App/mobile/Overlays.html" />
    <link rel="dx-template" type="text/html" href="App/mobile/Panorama.html" />
    <link rel="dx-template" type="text/html" href="App/mobile/Pivot.html" />
```

Second modify the `App\main-mobile.js` to include a refernce to all your viewmodels. These are case sensitive
to your file name, ie KitchenSink.Home relates to the viewmodel Home.html and  KitchenSink.home relates to the viewmodel home.html.
If you want your item to appear in the nav menu then add it to the `//#region Slide Menu Config` code

```javascript
   // [add all the viewmodels here..]
    require(['viewmodels/home',
             'mobileviewmodels/Form'
             
    ], function (homeVm, formVm){
        var self = {};
        self.homeVm = homeVm;
        self.formVm = formVm;
 
        KitchenSink.home = function (params) {
            return self.homeVm;
        };
        KitchenSink.Form = function (params) {
            return self.formVm;
        };


        // now navigate to the first route
        KitchenSink.app.navigate();
    });
```

Next create your view using the phonejs data binding syntax like this example, `App\mobile\home.html`

```javascript
	<div data-options="dxView : { name: 'home' } " >
		<div data-options="dxContent : { targetPlaceholder: 'content' } " >
        
			<h1 data-bind="text: title"></h1>
        
			<div class="home-splash" data-bind="dxAction: '#Form'" >
				<div class="home-demo-logo">
                
				</div>
			</div>
        
		</div>
	</div>
```
Lastly, to switch themes for testing modify `App\main-mobile.js`

```javascript
	// Uncomment the line below to disable platform-specific look and feel 
	// and to use the specified theme for all devices
    DevExpress.devices.current({
        phone: true,
        platform: 'android' // android, ios, win8
    });
```


You can test your app in the web browser. If it works then wrap your app up into a phonegap app. 
You can use the PhoneGapBuildPhoneJs.ps1 script to help copy out the correct files to your desktop.




How to Create a PhoneGap Build App
-----------------------------------
1. Review the docs here, https://build.phonegap.com/docs 
2. Make sure you minify all your files into a file called main-built.js ...weyland can help with this http://durandaljs.com/documentation/Automating-Builds-with-Visual-Studio/
3. To get up and running quickly...simply build the app in `test` mode so weyland will build and minify the js together
4. Then reference `main-built.js` in your index.html (sometimes the file comes in as hidden in Visual Studio, you may have to include it in the project). 
5. Now make sure you api points to azure, goto App\config.js and set your apiUrl to you azure api ... for me it's "http://spastack.azurewebsites.net"
5. If you use github then simply push the code up and reference your repo on the phonegap build site.... this project is too large though and you dont want all the server code code included anyways. Therefore, you can run the 
`PhoneGapBuild.ps1` and it will output a folder on the desktop. You can manually Zip this folder and 
upload to https://build.phonegap.com . 
6. An app will be built and available for download from the Phonegap Build site.

How to create app icons
-------------------------
You can generate Android icons using this site http://android-ui-utils.googlecode.com/hg/asset-studio/dist/icons-launcher.html#foreground.type=image&foreground.space.trim=0&foreground.space.pad=0&foreColor=fff%2C0&crop=1&backgroundShape=none&backColor=fff%2C100
Then configure the `config.xml` to use them in the build


How to create a Custom binding handler for durandal and knockout
----------------------------------------------------------------
To get the jquery.mmenu plugin to work, a durandal custom binding handler was created in  
`services/binding-handlers.js`. This file is loaded at app start in main.js.

<pre>
composition.addBindingHandler('mmenu', {
    init: function (element, valueAccessor, allBindingsAccessor, viewModel) {
        $('a#open-icon-menu').click(function (e) {
            e.stopImmediatePropagation();
            e.preventDefault();
            $(element).trigger('toggle.mm');
        });
        $(element).mmenu();

    }
});
</pre>


Automated Builds with Weyland
------------------------------
Features

* JS Linting
* JS Minification
* RequireJS Optimization

Usage

* Install NodeJS and NPM
* On the command line execute npm install -g weyland
* Navigate into your web project and place a weyland-config file at the root.
* From your project directory execute weyland build


Autogenerate appcache manifest with Fiddler for offline web
------------------------------------------------------------

http://blogs.msdn.com/b/fiddler/archive/2011/09/15/generate-html5-appcache-manifests-using-fiddler-export.aspx


How OAuth works in this app
-----------------------------
I added the Individual user account authenication built into ASP.NET. I login simple hit the `/login` route. 
After you are authenicated you will be redirected to `index.html` from there you are passed a token that can be consumed like this 
(the part after Bearer is your token):

```
GET http://localhost:65310/api/Account/UserInfo HTTP/1.1
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
Host: localhost:65310
Authorization: Bearer 9P1pkFVc5rDBikSxyCuvgr_T8L7oR0lok5SdryBF4yDU5jj21sO_d-gAStm_YdZHNp8N_gIWc8kklTrydHRVI_FjeXhD66allUjw2XO1fc
```

Testing with Jasmine
--------------------
TODO


TODO Items
----------
* add testing with Jasmine
* make menu disappear when you click a menu item in mobile view
* work out login kinks on mobile, maybe try identity providers or azure mobile services
* Add /v1/odata route (http://www.asp.net/web-api/overview/web-api-routing-and-actions/attribute-routing-in-web-api-2)
* Separate admin routes from normal user routes
	* user route -  /v1/odata/TodoItems (lock down filtering where uid using this http://www.asp.net/web-api/overview/odata-support-in-aspnet-web-api/odata-security-guidance)
	<pre>
	```csharp
		// Validator to restrict which properties can be used in $filter expressions.
		public class MyFilterQueryValidator : FilterQueryValidator
		{
			static readonly string[] allowedProperties = { "ReleaseYear", "Title" };

			public override void ValidateSingleValuePropertyAccessNode(
				SingleValuePropertyAccessNode propertyAccessNode,
				ODataValidationSettings settings)
			{
				string propertyName = null;
				if (propertyAccessNode != null)
				{
					propertyName = propertyAccessNode.Property.Name;
				}

				if (propertyName != null && !allowedProperties.Contains(propertyName))
				{
					throw new ODataException(
						String.Format("Filter on {0} not allowed", propertyName));
				}
				base.ValidateSingleValuePropertyAccessNode(propertyAccessNode, settings);
			}
		}
	```
	</pre>
	* admin route - /v1/odata/admin/TodoItem
