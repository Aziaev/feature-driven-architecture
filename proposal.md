# React-Redux-Saga architecture for large applications. [DRAFT]

A set of conventions and principles to make React-Redux application more maintainable. This example is a fork of [feature-driven-architecture](https://github.com/technology-ebay-de/feature-driven-architecture) convention. 
It **will** appear as over-engineered, because this structure is designed for large applications.

If you think some of those are not a good fit for your application, feel free to fork and adapt for your use cases.

## Motivation

While React provides us with components and redux with state management layer, they don't come with a structure and architecture out of the box, that fits well for working on a large application with multiple teams.

Structuring application based on basic Redux examples imposes a high risk of namespace collisions of action types and action creators. It also comes with no guidance about when to connect a component to the store and leads to a props passing overhead, when too many props need to be passed from the connected top level component down the deeply nested tree.

Global state brings 2 further challenges:

1.  State dependencies are implicit. You never know what will break once you change it.
1.  It is easy to forget to remove properties when component stops using them over time, which leads to state pollution.

React has no opinions on how to structure an application since components are universal. We need a structure that enforces high cohesion principle by keeping code implementing same feature in the same directory.

Another unsolved problem when every component can use every component is - all components become highly interconnected at some point. To avoid this, we introduce 2 complex types: `module`, `container` and `view`. Also, we keep shared "components" clearly separated.

## Solution

We use Redux to its fullest while avoiding namespace collisions and implicitness as much as possible by defining a set of conventions and principles, without introducing additional abstractions.

1.  We introduce highly cohesive features structure.
1.  We introduce new complex types: `module`, `container` and `view`
1.  We introduce `module` ??????????????and `pages` namespaces to the state.
1.  We introduce scoped action types.

## Terminology

- "Container" - "Smart" React component with connecting to Redux store. Without rendering DOM elements or CSS, may use renderers
- "View" - A rendered entire page view which created with composing of "Features" and "Components". Provides full functionality of app. Can have own state. 
- "Module" - Компонент который реализует в себе законченную функциональность которая включает в себя контейнер, набор страниц, композицию feature, и т.д.
- "Feature" - Renders complex user-facing functionality, reusable between pages. Can have own state.
- "Components" - "Dumb" React components. 
- "renderers" - Stateless function which renders a React Components or DOM elements.
- "store" - Redux store
- "action creators" - Redux action creators
- "action types" - Redux action types
- "selectors" - Reselect selector functions

view > component > block > element

## App structure

```
APP DIRECTORY
├──node_modules
├──public
│  ├──favicon.ico
│  ├──index.html
│  └──manifest.json
│  
├──build
├──src
│  ├──assets
│  │  ├──css
│  │  ├──fonts
│  │  ├──icons
│  │  └──img
│  │
│  ├──components - Directory for common reusable components
│  │  └──<Component directory>
│  │     ├──__tests__
│  │     ├──<Component name>.css - CSS file for component (optional)
│  │     ├──styled.js - Styled components file
│  │     ├──[other jsx files]
│  │     └──index.jsx - Default export by component name
│  │     
│  ├──helpers - Directory or shared utility functions
│  │  ├──__tests__
│  │  ├──<file name>.types.js - Optional file for flow types
│  │  └──<file name>.js - JS files with helper functions
│  │
│  ├──modules (page views)
│  │  └──<module name>
│  │     ├──__tests__
│  │     ├──core - Directory for files needed to Redux Store and sideffects
│  │     │  ├──actions.js - Actions
│  │     │  ├──initialState.js - Initial state for particular module
│  │     │  ├──reducer - Reducer for particular module
│  │     │  └──sagas - Sagas for particular module
│  │     │  
│  │     ├──types.js
│  │     ├──constants.js
│  │     ├──view.jsx - View component
│  │     ├──styled.js - Styled components file
│  │     ├──[...<componentname>.jsx] - Other react stupid components
│  │     └──index.jsx - Entry point and container with connection to store
│  │  
│  ├──core - Directory for redux utility code such as actions, reducers, sagas and async api functions
│  │  ├──__tests__
│  │  ├──constants.js - For common constants
│  │  ├──async.js - Async utility functions are here
│  │  ├──initialState.js - Initial states of all modules are imported here
│  │  ├──rootReducer.js - Reducers of all modules are imported here
│  │  ├──rootSaga.js - Sagas of all modules are imported here
│  │  ├──utils.js - Utility functions
│  │  └──store.js - Redux store file
│  │
│  └──index.js - Entry point
│
├──README.md
├──.flowconfig
├──.gitignore
├──.eslintrc
└──package.json
```


## Page (`src/pages/{page}/`)

Every page renders contents of the entire document. It is designed to use features and connect them. It acts as an interoperability layer between the features. A change on one page should never break a different page.

#### Must not

- A page must not import from other pages.
- A page must not access features state `state.features.{feature}`.

#### Must

- A page must export a component.
- A page must export a route for the router.
- A page must use the following naming schema for the action types `page/{page}/{action}`.

#### May

- A page may export a reducer.
- A page may connect to the store.
- A page may access the global `state`.
- A page may access the page state `state.pages.{page}`.
- A page may render any feature.
- A page may render feature A inside of feature B by passing a render prop or component.
- A page may exchange data between features.
- A page may provide data to a feature if a feature can't fetch it by itself.

## Feature (`src/features/{feature}/`)

A feature is a self-contained, renderable, user-facing functionality, that is encapsulated and reusable on different pages. Feature is to be defined by the engineers in a technical scope. Product managers may have different understanding of a feature.

#### Goals

- Be able to remove a feature completely by removing its directory, without leaving unused code behind.
- Allow more autonomy in feature development for different teams.
- A change in a feature should not implicitly break a different feature.
- You should be able to swap out a feature on a page without breaking other pages still using the old one.
- Enforce separation of container and presentational components, because it leads to a cleaner code.

#### How

- We need to keep it as cohesive as possible - action creators, action types, selectors, components and everything else a feature needs should be in a corresponding feature directory.
- Extract code into shared directories only when required by more than one feature and only if functionality is complex enough.
- Always use one subtree in the global state: `state.features.{feature}`. Never access anything else from state directly.
- Accept props from the page if there is external data that a feature doesn't have.

#### Must not

- A feature must not import from other features.
- A feature must not import from pages.
- A feature must not access any other state than `state.features.{feature}`.

#### Must

- A feature must export a component.
- A feature must use the following naming schema for the action types `feature/{feature}/{action}`.

#### May

- A feature may export a reducer.
- A feature may connect to the store.
- A feature may export a subroute for the router.
- A feature may access the feature state `state.features.{feature}`.
- A feature component may accept props.
- A feature may access shared resources.
- A feature may fetch data from an API.

## Shared components (`src/shared/components/`)

Every directory corresponds to one or to a set of components shared between the features or pages.

#### Must not

- Must not connect directly to the store, router or any other global system.

#### Must

- If directory contains one component, its name must start upper case.
- If directory contains multiple components, its name must start lower case.

#### May

- May be containers or presentational components.

## Store (`src/store/`)

We use a single store for the entire application. Here you may:

- Setup the store.
- Combine all reducers.
- Apply middleware.

#### State shape

```json
{
  "pages": {
    "pageA": {},
    "pageB": {}
  },
  "features": {
    "featureA": {},
    "featureB": {}
  }
}
```

## Router (`src/router/`)

Provides a router component that uses routes from pages.

## Containers and renderers

## Todo

- Where does feature starts and where does it ends? What if feature gets too big?
- What is MUST and MAY regarding directory structure in a feature?
- How to implement and test side effects in features?
- How to get the data of one feature from another?
  - Introduce clusters.
  - Page local clusters.
  - Shared clusters.
  - Implement a new feature that demonstrates data exchange between features.
- Remove dependency to features in store by using registration pattern. Currently when removing a feature, it needs to be removed from store.
- How to use subroute in a feature (nested routing)?
- Once all routes are in features and pages, can we render an overview of all routes somewhere?
- Describe the role of selectors in features and pages
- Describe the role of reducers in features and pages
- Root currently isn't a page (mb we need to reuse it over relations or containers)
- Use reselect.
- Use state based router?
- Make it work on codesandbox
- Flow/TypeScript?
- Graphql?
- forms
- Workspace for features with yarn? (npm has plans to implement this)
