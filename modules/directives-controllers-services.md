<!--
{
"name" : "directives-controllers-services",
"version" : "0.1",
"title" : "When To Use Directives, Controllers, or Services in AngularJS",
"description": "Why each concept is great at what they’ve been designed for, and why we’d use them in that fashion.",
"freshnessDate" : 2013-06-08,
"homepage" : "http://kirkbushell.me/when-to-use-directives-controllers-or-services-in-angular/",
"license" : "All Rights Reserved"
}
-->

<!-- @section -->

# Overview

Angular JS is a very powerful front-end MVC framework. It also introduces many concepts that may be unfamiliar. A few of these are:

* Directives
* Controllers
* Services

Let’s look at each one in turn and investigate why each concept is great at what they’ve been designed for, and why we’d use them in that fashion. Let’s start from the bottom.

<!-- @section -->

# Services

If you’ve worked with Angular JS already, you may have come across services - which are basically a nice name for Angular singletons. These puppies get passed around regularly, ensuring that you’re dealing with the same object each time, unlike factories. With that in mind, it allows us to do some pretty cool things, like have various controllers or directives affect its values. This is another common question in the #angularjs channel, and is how you share data between chunks of code in your application. Let’s have a look.

First we’ll setup a module. We’ll use this throughout the article. For a refresher on modules, check out the official Angular docs.

<!-- @link, "url" : "https://docs.angularjs.org/guide/module", "text": "Make sure you are good with the basics of modules." -->


```javascript
var module = angular.module( "my.new.module", [] );
```

Next, we’ll create a new service. Let’s say this module will be used for managing books. So let’s create a book service, and add to it an array of JSON objects containing various pieces of book data.

```javascript
module.service( 'Book', [ '$rootScope', function( $rootScope ) {
  var service = {
    books: [
      { title: "Magician", author: "Raymond E. Feist" },
      { title: "The Hobbit", author: "J.R.R Tolkien" }
    ],

    addBook: function ( book ) {
      service.books.push( book );
      $rootScope.$broadcast( 'books.update' );
    }
  }

  return service;
}]);
```

This is a very simple service (but sometimes that’s all you need). All we’re doing is managing an array of books, with a helper method to add more books should the need arise. This also broadcasts an event to the application to notify anything using our service that the array has been updated, and likewise - they should also do an update. Now, what we can do is pass this service around to the various controllers, directives, filters or whatever else may need it - and they’ll have access to these methods and properties. So let’s do that now.

```javascript
var ctrl = [ '$scope', 'Book', function( scope, Book ) {
  scope.$on( 'books.update', function( event ) {
    scope.books = Book.books;
  });

  scope.books = Book.books;
}];

module.controller( "books.list", ctrl );
```

Again, pretty straight forward. What we’ve done is create a new controller for our module. This is then provided with both the `$scope` provider, and the Book service we’ve just created. Notice what we’ve done? We’ve assigned the books array we created earlier on the Book service, to a property called books on the controller’s local scope object. Cool, huh?

So, what’s the point? We could have saved some time and just created the array on the controller. Correct - we could have. And it may have saved us time - but what if we needed to handle those books elsewhere? Managing data via scopes is reckless. Scopes can easily become corrupted or dirtied by other controllers, directives, models and the like. It gets messy quickly. Having a central channel (in this case a service) to manage all book data, and requests to modify it any way, not only is a lot cleaner - it’s also much easier to manage as the application grows. Lastly - it keeps your code modular (something Angular excels at). Should you ever need that service for another project, you don’t need to dig through scopes, controllers, filters, etc. to find relevant code - it’s all there in the service!

So. When do we use services? Whenever we want to share data across domains. Additionally, thanks to Angular’s dependency injection system, this is both very easy to do and very clean.

<!-- @section -->

# Controllers

That brings us to controllers! Unless you’ve worked with front-end MVC before, moving from the server to the client can be a bit of a head twist. Why? Because although controllers for front-end development serve a very similar purpose, it’s also done very differently to what you may be used to on the server. Controllers in Angular do not handle “requests” per se, unless it’s to handle routes (many are calling this type of setup a route controller), more specifically, especially if used as part of your application’s interface - they may just be managing a small subset of data.

Controllers should be used purely to wire up services, dependencies and other objects, and assign them to the view via scope. They’re also an excellent choice when needing to handle complex business logic in your views. Taking our books example earlier, there isn’t really anything I’d want to add to the controller.

But Kirk, what about if i want to add a book? Shouldn’t I add a method on the controller to handle that? No, and here’s why. It’s part of DOM interaction/modification. Put that in a directive. How? I’m glad you asked.

<!-- @section -->

# Directives

In the various applications I’ve written in Angular so far, by far the majority of the application’s complexity is in the directives. They are a powerful tool for working with and modifying the DOM, and that’s what we’re going to do here. To wrap this article, we’ll provide a button for a user to be able to add a new book to the service.

A common anti-pattern (in my humble opinion) is adding DOM interactions to the controller. Angular defines directives as chunks of code you use for DOM manipulation, but I feel it’s also great for interactions. Let’s extend out example above by providing the user with the ability to add a book to the service.

```javascript
module.directive( "addBookButton", [ 'Book', function( Book ) {
  return {
    restrict: "A",
    link: function( scope, element, attrs ) {
      element.bind( "click", function() {
        Book.addBook( { title: "Star Wars", author: "George Lucas" } );
      });
    }
  }
}]);
```

Simple stuff. We’re creating a directive, whose sole purpose is to simply add a book to the list of books already registered on our Book service. Let’s incorporate that in our view.

```html
<button add-book-button>Add book</button>
```
As you can see, we’re just adding the directive as an attribute. Every time this button is now clicked, it will add the book Star Wars to our Book service. Simple, easy, modular - and easily transferable. Now, why wouldn’t we just add a method called something like addBook on our controller, that does something like the following:

```javascript
$scope.addBook = function() {
  Book.addBook( { title: "Star Wars", author: "George Lucas" } );
};
```
This would give us the same result, right? Yes, it would - with one major caveat. Should I need to add books again from another location - I have to copy that code (very un-DRY!) or refactor (which itself isn’t a bad thing). By building a directive straight off the bat, we don’t need to worry about that later - and it takes us next to no time at all to do so. By building directives for DOM interaction and modification, we’re immediately setting ourself up to handle increasing application complexity as business requirements roll in. This is a good thing, as it ensures we fight our own implementations less, and write DRYer code all the time.

Angular’s philosophy of modular development really lends its weight to non-trivial applications. It allows us to write our front-end code in such a way that we don’t end up tripping over ourselves, or the framework - and this is probably its greatest strength.

Hopefully I have shown when and where you would like to use these various Angular concepts, in order to get the most out of your own code.

Comments and feedback always welcome!
