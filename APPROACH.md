# Listing approach and inner monolog

I'm creating simple app in vanilla js - no frameworks to help..maybe just bootstrap, because I still don't want to spend time learning css at the moment.

The app will have a list of headlines with a corresponding picture on the first page, and a summarised article on the detail page.

This involves 2 api calls, one to the news outlet API and one to the summariser API

I will be using ES6 class keyword in this.

# Before I start

## What am I developing

The app will have the following pages

- Home: which will contain a list of headlines - using bootstrap card system to display them (centered in the page)
- Headline for specific id: contains the summarised text of the article

Each page in my app will have the following structure:

- Navbar
- Content section

Each route in my app will have the following structure

- Resource
- Identifier

`localhost:8080/#/articles/id`

- resource = article
- identifier = id

I am making hash based router.
In the router, the resource is pre-defined, but the identifier is dynamic

I am using mockapi.io for initial testing of the API, before hitting the Guardian API for initial tests

The app will focus on:

- routing
- architecture - using modules

## Routing

- reliant on th # - fragment identifier [WIKI link](https://en.wikipedia.org/wiki/Fragment_identifier)
- when browser hits this char, they skip everything after it, which means that sending the user to a different page is our responsability, not the browser's

`localhost:8080/#/something` or `localhost:8080/#/somewhere` are the same for the browser and it dosen't send a server request to fetch a route

### Steps:

In the routing-module.mjs

1. Hard code the routes

```js
const routes = {
  '/': 'Home',
  '/articles/:id': 'ArticleShow'
};
```

2. Router code takes a URL and checks against the list of supported routes

```js
const router = function() {
  //Lazy loading of view element
  const content = null || document.getElementById('page_container');

  //Get the request object : {resource, id} elements
  let request = Utils.parseRequestURL();

  //Parse url - if it has id route to resource/id
  let parsedURL =
    (request.resource ? '/' + request.resource : '/') +
    (request.id ? '/:id' : '');

  //Get the page from the routes
  //If parsed page is not in the routes - select 404 page

  let page = routes[parsedURL] ? routes[parsedURL] : Error404;

  content.innerHTML = page.render(); //where is render()

  return parsedURL;
};
```

In the app.js

3. App listens on change of urls - and routes accordingly

```js
window.addEventListener('hashchange', router);
```

4. App listens for page load and loads the router once page load completed

```js
window.addEventListener('load', router);
```

In the helper-method.js

5. Created a Utils module

- parseRequestURL

```js
const Utils = {
  parseRequestURL: function() {
    let url = location.hash.slice(1).toLowerCase() || '/';
    let r = url.split('/');

    //create request object to return from this function
    let request = {
      resource: null,
      id: null
    };

    //assign url elements after #
    request.resource = r[0];
    request.id = r[1];

    return request;
  }
};
```

**What's happening above:**

Page load:

1. When app loads, and event is fired and router is called
2. Router takes url and breaks it down into the route schema of resource and identifier
3. Url is concatenated with each schema element resource/id
4. Url string is compared against existing map of routes that we support
5. If there is a match we render the page which comes `routes[parsedURL]`, by calling render(). We insert the html int the content element, which we obtained by `document.getElementById('page_container')`

Dynamic route:
`localhost:8080/#/articles/1`

1. When page loads, the `hashchange` even is triggered and router is called again
2. Router takes the url and breaks it down into resource and id
3. Url is reconstructed with schema elements, however, if there is an id, that part of the schema is replaced with `:id` so that we can map it to our route. This helps me define a simple route to show the data dynamically
4. Url string is compared to existing routes
5. If there is a match, page is rendered

> We re-render the page content of the app every time the user changes the url and loads the page (enter,refresh) of if he uses in app navigation links

> Navigation state is saved in the browser history - back/forward works

> For # routing to work, all in app links must have the # in the href

## Creating the pages:

To render the pages i created a separate module with separate objects for the pages I am rendering.  
I'm using ES6 template literals to define my html template.
I'm using a function so that the rendering only happens when the page is called.  
Functional approach useful for working with different components on a page - data can be passed to it for rendering using the function params

```js
//pages.mjs module
let Home = {
  render: function() {
    let view = `<section class = 'section'>
    <h1> About </h1>
    </section>
    `;
    return view;
  }
};
```

I've also added a button for testing purposes

- the event listener can be added in the router, or through a method of the page
- i'm adding an `after_render` method which will be called in router after rendering the page. It will add an event listener. Code looks like this now

```js
//pages.mjs module
let Home = {
  render: function() {
    let view = `<section class = 'section'>
    <h1> Home page </h1>
    <button id="myBtn">Button</button>
    </section>
    `;
    return view;
  },
  after_render: function() {
    document.getElementById('myBtn').addEventListener('click', () => {
      alert('Button pressed');
    });
  }
};
```

## Create Articles and Article models

- This are needed in order to map data received from the api and display it on page

- Article model will have text and image properties as well as a getData function to retrieve the text and image
- Articles will have a list to hold articles and a method to retrieve 1 or all articles

Rendering:

- Home page will render all the articles
  - require the list of article objects
  - use the list getData to get a list of article objects
  - use the article getData function to get access to the article attributes
  - use the attributes to render the article on the page
- ShowPage will render 1 article
  - event listener on the article card
  - when triggered ShowPage is displayed
  - it will need the id of the article to retrieve it and render it
  - the article getData will be used here