<h1 align="center">
<img width="40" valign="bottom" src="https://angular.io/assets/images/logos/angular/angular.svg">
Angular Architecture Guidelines
</h1>
<p align="center">A cohesive guidelines for building Angular applications</p>

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
	- NgRx Effects
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

