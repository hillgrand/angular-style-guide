# Angular Style Guide @ [HILLGRAND](https://hillgrand.com/)

App projects must follow the [Angular Style Guide](https://angular.io/guide/styleguide) , make good use of [TypeScript](http://typescriptlang.org/) types and the provided information on this page:

1. Project Architecture:

    1. Module Types:

        * **App Module** (one / no directory) - the main project module.

        * **Core Module** (one / has directory) - contains core functionality that is used globally inside the project.
        
            One application can have **ONLY ONE CORE MODULE**.

        * **Shared Module** (one or many / has directory) - contains shared functionality that is re-used across different ((sub)feature) modules. 

            One application must have at least **ONE CORE MODULE**. In some cases feature modules can have a shared module containing feature specific shared functionality that is used only inside the feature module and sub modules (think of it as lexical scope in JS).

            Specifics:

            * **Providers** in the shared module: 
            
              If an application has only one shared module the providers should be put inside the **core module**.

              If an application has one shared module and feature shared module(s) and certain providers need to be used across the feature module you can create a static `forRoot` method on the nearest possible shared module that returns the module with the providers. Use the `forRoot` method in the top most module that imports the current.
        
        * **Feature Module** (zero or many / has directory) - a module containing all the necessary components/directives/pipes/services for a feature.

            The modules should represent the different features that the application can have. For example if an application deals with user management it should have a user module that contains all the components/directives/pipes/services that are used for this feature.

            If a specific service has to be used in a different feature module(s) it has to be hoisted to the core module or if we have shared modules with providers to nearest shared module.

        * **Routing Module** (zero or many / no directory) - a module containing the routing configuration for a module.

    2. State Management - [NGRX](https://ngrx.io/): Every ((sub)feature) module must have one `+store` directory that contains:

          * **Actions** (one / has directory / no barrel necessary) - contains all the actions for the individual reducers

          * **Effects** (one / has directory / no barrel necessary) - all the effects for the individual actions for a reducer

          * **Reducers** (one / has directory / must have barrel) - all the different reducers for the individual stores. The barrel constructs the main reducer function combining the individual reducer functions and adds the meta reducers.

          * **Selectors** (one / has directory / must have barrel) - all the different selectors for the individual stores. The barrel file should contain all the feature selectors and should connect them with the selectors created in the files in the selectors directory.

          * **Models (Non NGRX)** (one / has directory / no barrel necessary) - Must contain services for each individual store. The models are used for component/directive/pipe <-> store communication. All communication should be done through the models.

    3. Resolving Data - [HG Resolvers](https://github.com/IliaIdakiev/hg-resolvers) (**DO NOT USE ANGULAR RESOLVERS FOR LOADING ASYNC DATA**): Every ((sub)feature) module must have one `-resolvers` directory that contains all the resolver directives used for resolving data that will be presented inside the views. Resolvers **CAN BE SHARED** and if so they should be put inside the nearest shared module inside a `-resolvers` directory.

    4. Routing - All routing and navigation should done through NGRX and the router service should only be used to get state that is not present in the `routerModel`.

          * **Guards** (zero or many / has directory / no barrel necessary) - contains all the canActivate/canDeactivate/canLoad/canLoadChild guards for the ((sub)feature) module.

          * **Router Module** (zero or many / no directory ) - every user interface (including dialogs) should have a unique route (some exceptions may apply) and all not mandatory modules should be lazy loaded (make sure that you haven't imported the ((sub)feature) module somewhere else otherwise the lazy loading won't work). The routing module (file) should be located inside the ((sub)feature) itself.


    5. Providers - In most cases use the providers array of the nearest module, in some cases you might want to use the providers on the component/directive decorator. Never use the @Injectable providedIn property because of unit testing issues.


2. Application Elements:

    1. **Components/Directives/Pipes** should:
    
        * inject the different models (as **private** properties) in order to present the data from the store and connect the user actions with the UI.

        * the models **SHOULD NOT** be used directly inside the templates (for components and the structural directives). All the individual action dispatchers and selectors should exist as own properties on the component and the UI should interact with these properties.

        * if necessary create the required derived/modified store streams.

        * can contain minor state **THAT IS REACTIVE** to the data flowing from the store (either NGRX or the [query params store](https://github.com/IliaIdakiev/query-param-store)).

        * use derived/modified store streams instead of pipes when possible.

        * have simplified and optimized templates (use directives, pipes and rxjs operators).

        * (Components should) not have methods that are called inside the template (use Pipes) nor heavy calculations inside the templates. Every event should be handled inside the component class inside a method.

        * (Components should) not have unnecessary subscriptions and state. Make good use of the `async` pipe and limit the amount of subscriptions.
    
    2. **Services** should: 

        * only only contain the necessary remote resource calls and should not do anything else.

        * should only be used inside the NGRX effects

    3. **Models** should:

        * Contain all the necessary selector streams and action dispatchers needed for interaction with the given part of the store. 

        * Can contain derived/modified store streams if needed in more than one place, otherwise the derived/modified store streams should be created inside the components/directives/pipes.

    4. **Selectors** should:

        * be used to make state derivations but it's preferred to modify the data before saving it inside NGRX store (this can be inside the reducer or inside the NGRX effects)

3. Application State - Think of the application as a state machine. The input is the url it the output are the UI elements. Given the same input the UI elements should always be in the same state as they were before (the only exception is the data that is loaded from the database and is presented in the UI - these are the effects, everything else should be pure).

    1. **State from the server** should:
    
        * be kept **ONLY** inside the NGRX store in the proper format. If needed use [NGRX entity](https://ngrx.io/guide/entity).

    2. **UI (User) state**

        * should be kept inside the url - either route parameters or query parameters using the [query params store](https://github.com/IliaIdakiev/query-param-store) in order to be persistent. Do not use `localStorage`/`sessionStorage`/`cookies` to keep persistent state.

        * in some minor cases when the user state should not be persistent we can keep it inside the NGRX store.

    3. **Component/Directive/Pipe state**

        * can contain minor state **THAT IS REACTIVE** to the data flowing from the store (either NGRX or the [query params store](https://github.com/IliaIdakiev/query-param-store)).

4. Inner Communication

    * All communication should be done via NGRX. Usually NGRX is used only for state management but in our case we use it as a messaging system. Some actions might not get caught by a reducer in order to modify the state but something might be listening for this action in order to preform something else.

5. User Input:

    * For simpler forms use the template driven approach

    * For complex forms use the reactive forms approach.

    * When creating validators always make sure that they can be used in both form types which for the template driven case results in a directive and for the reactive approach results in a function. Create the function and reuse it inside the directive.

    * Don't forget to track the form changes using [form-control-change-tracker](https://github.com/IliaIdakiev/form-control-change-tracker) module.

    * Make sure that you present the `unsaved changes` dialog if there are any unsaved changes.

6. Reminders:

    * Always make sure that the streams complete and that there are not memory leaks! 

    * Do not use the **any** type. Always define the type of the variable or create an interface!

    * Do not use Promises nor async/await! Only RxJS.

    * Put $ after every rxjs stream name (e.g. source$)

    * Use Subjects as private properties only! If you need to export them in order to observe the values use asObservable and create a public stream.

    * Don't use setTimeout, setInterval, and so on... use RxJS.

---

# About JavaScript / TypeScript

1. Variables

    * Names should always be camelCase.

    * Names should start with a letter.

    * Names can start with _ only if they should not be touched from outside.

    * Always use `const` whenever the variable won't change otherwise use `let`.

2. Code Indentation
   
    * **2 Spaces**

3. Semicolons

    * **Always!**
