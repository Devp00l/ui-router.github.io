---
title: "UI-Router for React - Hello Solar System!"
excerpt: "Learn about parameters and resolve data"
layout: single
sitemap: true
redirect_from: /tutorial/react/hellosolarsystem/
---
{% include toc icon="columns" title="Hello Solar System!" %}

In this tutorial, we will build on [Hello World!](helloworld) and create a slightly more ambitious _Hello Solar System_ app.

We will implement a [list/detail interface](https://en.wikipedia.org/wiki/Master%E2%80%93detail_interface),
also known as master-detail.
To accomplish this, we will create two new application states:

- The `people` state will show a list of all the people.
- The `person` state will show details for a specific person.

At any time, the user can click "reload plunker", and the app will restart at the same URL.
The URL contains the information necessary to restore the application's state.
When the app is restarted, it will be in the same state as before.
{: .notice--info}

Plunker embeds can time out.
If you get a "Not Found" response, your plunker embed has timed out.
Click the "Refresh" icon to get a new plunker, then try experimenting with the "reload plunker" button again.
{: .notice--info}

## Live demo

Take a look at the completed Hello Solar System live demo below.
Click the `UISref` wrapped "People" to view the list of all people.
Click a person to view the person details.

As you navigate through the app, the [UI-Router State Visualizer](https://github.com/ui-router/visualizer) shows
the current state
{: .notice--info}

<iframe style="width: 100%; height: 450px;" src="//embed.plnkr.co/3qSxCVrm1KuDTVH9nHjO/?show=preview" frameborder="1" allowfullscren="allowfullscren"></iframe>

<br>

# New concepts

This app introduces some new concepts and UI-Router features.

- [Resolve data](#resolve-data)
- [State Parameters](#state-parameters)
- [Linking with params](#linking-with-params)

## Resolve data

When a user switches back and forth between states of a single page web
app, the app often needs to fetch application data from a server API,
such as a REST endpoint.

A state can specify the data it requires by defining a `resolve:` property.
When the user tries to activate a state which has a `resolve:` property,
UI-Router will fetch the required data *before activating the state*.
The fetched data is then passed via props to the component(s).

The `resolve:` property on a state definition is an array.
Each element of the array is an object which defines some data to be fetched.
The object has the Dependency Injection `token` (name) for the data being loaded.
It has a `resolveFn` which returns a promise for the data.
It also has a `deps` property, used to define the DI tokens for the `resolveFn`'s dependencies (function parameters).
{: .notice--info}

The `resolve` property of the `people` state is an array containing a single object.
The object defines how to fetch the `people` data, and assigns it a DI `token` (its name).

```js
resolve: [{
  token: 'people',
  resolveFn: () => PeopleService.getAllPeople()
}]
```

The object defines a `resolveFn` which returns a promise for all the people data.
The data is assigned a DI `token` of `'people'`.
{: .notice--info}

When fetching data, we recommend delegating to services which return promises.
{: .notice--info}

UI-Router waits until the promise returned from `PeopleService.getAllPeople()` resolves before activating the `people` state.
The `People` component is created, and the list of people is passed to the component via its `props`.

```js
People.propTypes = {
  resolves: PropTypes.shape({
    people: PropTypes.arrayOf(PropTypes.object)
  })
}
```


## State Parameters

We also want to allow the user to be able view the details for a specific person.
The `person` state takes a `personId` parameter, and uses it to fetch that specific person's details.

The parameter value is included as a part of the URL.
This enables the same person details to be shown when the plunker is reloaded.

The `person` state definition:

```js
const person = {
  name: 'person',
  url: '/people/:personId',
  component: Person,
  resolve: [{
    token: 'person',
    deps: ['$transition$'],
    resolveFn: (trans) => PeopleService.getPerson(trans.params().personId)
  }]
}
```

The URL will reflect the current `personId` parameter value, e.g., `/people/21`
{: .notice--info}

The `person` resolve delegates to PeopleService to fetch the correct person.
The resolveFn is injected with the `Transition` object because the first element of the `deps` property is the `$transition$` token.
This way it can access the `personId` parameter.
The `Transition` is a special injectable object with information about the current state transition.
{: .notice--info}

### Linking with params

Note that our app's main Navigation Bar links to three states: `hello`, `about`, and `people`,
but it doesn't include a link directly to the `person` state.
This is because the state cannot be activated without a parameter value for the `personId` parameter.

In the `people` state we create links to the `person` state for each person.
We still create the link using the `UISref` component, but we also include the `personId` parameter value.
As we loop over each person object using `.map()`, we provide the `UISref` with the `personId` using each person's `.id` property.

{% raw %}
```js
let list = people.map((person, index) => (
  <li key={index}>
    <UISref to="person" params={{personId:person.id}}>
      <a>{person.name}</a>
    </UISref>
  </li>
));
```
{% endraw %}

