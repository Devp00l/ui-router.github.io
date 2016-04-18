---
title: "UI-Router Features"
layout: splash
excerpt: "UI-Router features"
sitemap: true
permalink: /about/features
---

{% include toc icon="columns" title="Features" %}

## States

UI-Router is state based.  A state defines each logical "place" within the application. 
It describes the URL, UI (view or views), behavior, logical prerequisites, and data dependencies for that "place". 
  
UI-Router states are hierarchical; states can be nested inside other states, forming a tree of states.  
Child states may inherit data and behaviors from their parent states.

[Read more about ui-router states](/about/states)

## Views

A state defines its UI and behavior using views.  The view declares the component which should fill the 
parent viewport (`<ui-view>`).

- **Nested**: a state's view(s) can nest inside other views
- **Multiple Named**: a state can define multiple named views
    - Named views can target any viewport, even outside the current DOM hierarchy.
    - This can be used to fill (or override) headers or nagivation, based on the current state.

## Urls

- **Optional**: states may have URLs, but it isn't required
    - after a transitioning to a state with no URL, the browser's URL does not change
- **Hierarchical**: a child state's url fragment is appended to the parent state's url fragment
- **Parameterized**: a url may be parameterized  
    - when a state needs specific data (e.g., `contacts.edit` needs to know which contact to edit), the url
    may contain a parameter, e.g., `url: '/edit/:contactId'`
    
## Parameters

- **Types**
    - **Path**: `/foo/:fooId` matches '123' in `http://mysite.com/foo/123`
    - **Query**: `/foo?fooId` matches '123' in `http://mysite.com/foo?fooId=123`
    - **Non-url**: arbitrary parameter data may be passed programmatically
- **Default Values**: a parameter may define a default value
    - the default parameter value is used when the parameter value is missing (from the URL, for example)
- **Squash**: when a default value is found, it may be removed from the URL
    - if `123` is the default value for the parameter `fooId` in the state url `/foo/:fooId`, then `/foo/123` can 
    be squashed to `/foo` in the browser's URL.
- **Typed**: parameters can be typed.
    - Typed parameters are encoded in the URL, but converted to their native type when used in the code.
    - Built in types: number, boolean, date, json
- **Custom Types**: custom parameter types may be defined by the user
    
        
## Resolve Data

A state often requires that some data be fetched from a server-side API (often, the data to fetch is specified
in a url parameter).  

The `resolve` mechanism allows data retrieval to be a first class participant in the transition.  When a state is
being entered, its resolve data is fetched.  If any of the resolve promises are rejected (perhaps due to a 401, 
404, or 500 server response from a REST API), then the transition's promise is rejected and the error hooks are
invoked.  This enables robust error handling for applications, and helps to avoid leaving the application in an 
inconsistent state.

- **Resolve data**: 
    - **declarative**: A state defines what data should be fetched (generally delegating to a service)
    - **views**: The data is provided to the state's views
    - **hierarchical**: A child state (and its views) can use the parent state's resolve data
    - **dependency resolution**: A resolve may depend on another resolve's result within the same state
    
## Transitions

Navigating between parts of the application occurs by transitioning from one state to another.
Transitions between states are transaction-like, i.e., they either completely succeed or completely fail.

- **Lifecycle**: transitions have a well defined lifecycle
    - **before**: before the asyc portion of a transition has begun
    - **start**: the transition has begun
    - **exit**: the transition is exiting states
    - **retain**: states are retained (a state was active, and is neither being exited nor entered)
    - **enter**: the transition is entering states
    - **finish**: the transition is finishing
    - **success/error**: after the transition is complete
- **Transition Lifecycle Hooks**: 
    - Hooks may be registered for any stage of the transition lifecycle.
    - Hooks can alter the transition:
        - **pause** the transition, waiting on some promise
        - **cancel** the transition
        - **redirect** the transition to a new target state
    - **match criteria**: 
        - A hook can choose which transitions it should be applied to.  
        - **to/from**: only run the hook if the transition is going to or coming from a specific state
        - **entering/exiting**: only run the hook if the transition is going to enter or exit a specific state
    - **criteria types**:
        - **state name**: the hook's match criteria can be state names, such as `banking.account`
        - **glob**: the criteria can be a state glob pattern, such ash `banking.**`
        - **callback**: the criteria can be a callback function, such as `tostate => tostate.data.requiresAuth == true`

