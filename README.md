
<h1 align="center">
<img width="40" valign="bottom" src="https://angular.io/assets/images/logos/angular/angular.svg">
Angular Architecture Guidelines
</h1>

# Overview
Architecture should be the most important aspect of an application. 
As we are increasing complexity by adding more and more business rules and features to the app, we can think of building an app that could easily scale and be maintainable.

In this documentation, we will present high-level guidelines for a well-designed angular application architecture based on the best practices and recommendations of the framework.

The goal would be to have a solid foundation of an app so that we can have a maintainable development speed and ease of adding new features in the long term.

The topics that will be covered are:
-	Overall application architecture
-	Application Layers
	-	Presentation Layer
	-	Core Layer
	-	Abstraction Layer
-	App Organisation 
	-	Features and modules organisation
	-	Core and Shared Modules
	-	Lazy-loading technique ( Performance )
	-	Module directory structure
- Structuring components
	-	Container and Presentation Components
	-	Passing State with Input and Output Properties
	-  Change Detection Strategies
- State Management
	- Purpose of state management
	- Unidirectional data-flow
	- NgRx
	- Store directory structure
- Additional references / tips
	- Linter (Ts Lint , HTML Hint, StyleLint)
	- Typescript Configuration
	- Formatters

## Overall application architecture

The application skeleton should be provided and created by the angular CLI.
 And in addition we will add new folders that will keep mostly configuration files
```
|-- config (Will contain all the variables and values needed for configuration)
|   |-- conf.dev.json
|   |-- conf.prod.json
|-- i18n (Will hold the translation keys in case we add a translation module)
|   |-- en.json
|	|-- fr.json
|-- index.html
|-- proxy.conf.json
|-- proxy.conf.dev.json (Proxy configuration used for dev mode)
|-- mock ( Will hold all the mock data that will be used)
|	|-- service-1.json
|	|-- service-2.json
|-- src
|   |-- app (Will be detailed on module and sections organisation)
|   |-- assets ( Folder that should only keep assets such as images / icons / svg ...)
|   |-- index.html
|   |-- sass( Folder for the app global styling)
|	|	|-- components (will hold the generic styles for components like buttons, modals, table)
|	|	|-- mixins (Will contain the utilities and mixins used over the application)
|	|	|-- variables
|	|	|-- layout
|	|	|-- vendors (Will add the styling / overrides styling for external libraries)
|   |   global.scss
|   |-- styles.scss
```

## Application Layers

The first step for decomposing our architecture will be splitting it into different layers.
The goal is to assign a **proper responsibility** to a **particular layer**.
Our main three layers are :
- Presentation Layer
- Abstraction Layer
- Core Layer

### Presentation Layer

The presentation layer is the place where all of our components will be stored. His responsibilities are only to **present** and **delegate**.
He will take care of presenting the UI and delegate all the end-user's action to the core layer via the abstraction layer. We should never implement any logic inside of this layer as he should only focus on his data display responsibility, and he does not need to know what's going on behind. 
Components do not need to know who provides the data and where do we save the final information.

### Abstraction Layer

The abstract layer represents a facade which acts as a middle-ware between the core and the presentation.
His main role is to decouple the core layer from the presentation layer, as he will be responsible for exposing the stream of state and logic that is exposed from the core layer to the presentation one. He will only let components know what they can see and do.

If we consider the fact that we will use a specific state management for the app (e.g: NgRx), we do not let the components interact directly with the state.

### Core Layer

The core layer is where the entire logic of a feature / application is stored.
All the data manipulation and the communication to external Back-end (API) should be handled in this layer.
If we are using a specific state management, like NgRx,  here would be the place to put our state definition, actions, reducers and effects. 

## App Organisation

Regarding the app organisation, we should consider following the Angular approach which is a feature-based module organisation.
The main idea is to split the application into **feature modules** representing different business functionalities. This is an approach to deconstruct the app into smaller pieces for better maintainability.
Each of the features modules share the same separation of the core, abstraction, and presentation layer. 
The advantage of those self-contained features is that we can have different members of a team working on a specific feature without breaking anything in the app during integration.

Application should also have two additional module for some technical reasons.
- **CoreModule**:
	Will define the singleton services, configuration, and exports of third-party modules that would be need on the app module. This module will be imported only once in the RootModule / AppModule. It is a module that is recommended by the Angular Style Guide. He is designed for your singleton type of services, anything that would be shared throughout the app and obviously any singleton would be good for that.  
Note: Services that are specific to a feature should only go in the feature folder / module. 
- **SharedModule**
 Here we have our reusable/common components, pipes, directives and so on. From there, we should also export commonly used Angular modules (like CommonModule). It can be imported by any feature module. 

To sum up, **shared modules** would normally be imported multiple times. on the other hand, **Core Module** should only be imported once and that should only happen into the root.

### Lazy Loading

The advantages of splitting the app into features is that we can take advantages of Lazy Loading which will decrease the initial load time of the application. We would only load the bundle related to a feature only when we will need it.

### Module directory structure

Below folder tree presents how we can place all the pieces of our Module inside a directory.
```
feature-module
|-- services
|   |-- feature-service.ts
|-- components (Will contain all the presentational components)
|   |-- feature-component (html, scss, spec, ts)
|   |-- another-feature-component (html, scss, spec, ts)
|-- containers(Will contain all the smart components)
|   |-- feature-container-component(html, scss, spec, ts)
|   |-- another-feature-container-component (html, scss, spec, ts)
|-- models (Will hold all the View model for the app, Classes, Interfaces, Type...)
|   |-- feature-model.ts
|   |-- another-feature-model.ts
|-- guards (if needed)
|-- resolvers (if needed)
|-- store (will hold the store definition of the feature, state, effects, reducers ...)
|-- utils.ts (if needed- could be a folder)
|-- feature.config.ts (if needed - contains configuration file needed for this specific feature
|-- feature.facade.ts (Will act as the abstraction layer between containers and services)
|-- feature.routing.module.ts (contains all the routing for this specific module )
|-- feature.module.ts
```

## Structuring components
In Angular applications, we should divide our components into two categories depending on their responsibilities.
First, are the **SmartComponents** aka (Containers) which usually:
- Have services / facades injected
- Interacts with the core layer
- Hold the data that will be passed to dumb components
- React to dumb components event
- Are Routable Component

In the other hand, there are  the **Dumb components**  (aka Presentational Component) which responsibilities are:
- to display UI element and data
- to delegate user interaction “up” to the smart components via events

## State management

### Purpose of state management

A classic challenge found while building large front-end application is keeping different parts of the UI synchronised.
Usually, changes performed in the state need to be reflected in different components, and as the application grows this complexity can only increase.
With large application, we should have a proper way of handling the state and regarding the way we will manage it, the abstraction layer will make our components independent of the state management solution.

The main goal would be to have **data consistency** across the application and have an architecture that would be less prone to bugs and easy to refactor and maintain. 

### Unidirectional data-flow

Angular itself uses unidirectional data flow on presentation level (via input bindings), so we should impose a similar restriction on the architecture level.
The idea would be to provide high-level components ( containers ) the state and pass it down to the presentation. Containers are given Observables with data to display on the dumb components.
To manage our state we can pick any state management library that supports RxJS (like  NgRx) or simply use BehaviorSubjects to model our state.

The state will be propagated to multiple components and shown in multiple places, but will never be modified locally. The changes will only come from the state and the components will only reflect the state in the UI.
This way we will have the state object as  **the single source of truth**.

### NgRx
Source: [https://ngrx.io/docs](https://ngrx.io/docs)

NgRx is a framework for building reactive applications in Angular. NgRx provides state management, isolation of side effects, entity collection management, router bindings, code generation, and developer tools that enhance developers experience when building many different types of applications.

 Core Principles :
-   State is a single, immutable data structure.
-   Components delegate responsibilities to side effects, which are handled in isolation.
-   Type-safety is promoted throughout the architecture with reliance on TypeScript's compiler for program correctness.
-   Actions and state are serializable to ensure state is predictably stored, rehydrated, and replayed.
-   Promotes the use of functional programming when building reactive applications.
-   Provide straightforward testing strategies for validation of functionality.

### Store directory structure

We can find below naming conventions, and a skeleton of how an NgRx based project should be structured according to the Angular/NgRx guidelines: 

```
|-- store (A specific folder that should be put in each feature module )
|   |-- actions ( will hold all the NgRx actions related to the feature)
|	|	|-- index.ts ( exports all actions as an es6 module )
|	|	|-- example-one.action.ts
|	|	|-- example-two.action.ts
|   |-- reducers ( Will hold all the functions that will be in charge of mutating the state )
|	|	|-- index.ts ( exports all reducer as an es6 module)
|	|	|-- example-one.reducer.ts
|	|	|-- example-two.reducer.ts
|	|-- effects ( will hold all the effects that will be used to handle the side effects in the app)
| 	|-- module-name.state.ts ( The store initialisation of the feature will be defined here)
|   |-- module-name.selector.ts ( Will have all the functions that will return specific portions of the state )
```


## Additional references / Tips
### Typescript Configuration

We should go for stricter configuration regarding the use of Typescript
We should avoid types that are too permissive
Flags that should be put in the TS compiler configuration:
{  
  "forceConsistentCasingInFileNames": true,  
  "noImplicitReturns": true,  
  "strict": true,  
  "noUnusedLocals": true,  
}
