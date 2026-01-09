# Angular Project structure

> [!Note]
>
> He started the lecture talking about localizing the project, and how to change the language of the project but he mentioned that it will be covered later.

> [!TIP] Angular History
>
> - AngularJS: was released in 2010 by Google.
> - It depends on JavaScript at the beginning.
> - It was cahnged later to a complete different thing which is Angular 2.
> - Angular 2 was released in 2016.
> - Angular 2 was written in TypeScript.
> - there is a new version every 6 months (nearly).

## Angular Structure

- Angular is a platform and framework for building single-page client applications using HTML and TypeScript.

> [!Question] what is SPA?
>
> - SPA stands for Single Page Application.
> - It's a web application that interacts with the user by dynamically rewriting the current page rather than loading entire new pages from a server (replacing components of the page).

> [!Question] What architecture does Angular use?
>
> - Angular uses the MVC architecture.
> - later it was changed to MVVM (Model-View-ViewModel) architecture.
> - Angular uses the MVVM architecture because it's more suitable for web applications.

> [!Question] What is the advantage of the MVVM architecture?
>
> - The advantage of the MVVM architecture is that it separates the view from the model, and the view model is responsible for the communication between the view and the model.
> - for more illustration check the following example of a viewModel:
>
> ```typescript
> export class UserViewModel {
>   user: User;
>   constructor() {
>     this.user = new User();
>   }
> }
> ```
>
> - in this example, the `UserViewModel` is responsible for the communication between the `User` model and the view.
> - it's about exposing only the neccessary data to the view.

> [!TIP] Starting a new Angular project
>
> - To start a new Angular project you can use the Angular CLI.
> - The Angular CLI is a command-line interface tool that you use to initialize, develop, scaffold, and maintain Angular applications.
> - To install the Angular CLI you can use the following command:
>
> ```bash
> npm install -g @angular/cli
> ```
>
> - To create a new Angular project you can use the following command:
>
> ```bash
> ng new my-app
> ```

> [!TIP] Design patterns you should know in Angular
>
> - Singleton.
> - Factory.
> - Decorator.
> - Observer.
> - Dependency Injection.

> [!TIP] Tsconfig.json
>
> - The `tsconfig.json` file is a configuration file that TypeScript uses to compile your project to JavaScript.
> - The `tsconfig.json` file is used to specify the root files and the compiler options required to compile the project.
> - we usually doesn't need to change the `tsconfig.json` file but we can change it if we need to.

> [!TIP] Angular.json
>
> - The `angular.json` file is a configuration file that Angular uses to build and serve your project.
> - The `angular.json` file is used to specify the project name, the project root, the source root, the output path, the assets, the styles, the scripts, the configurations, etc.

> [!TIP] src/index.html
>
> - The `src/index.html` file is the main HTML file that Angular uses to bootstrap the application.
> - The `src/index.html` file is the file that the browser loads when the application starts.
> - The `src/index.html` file is the file that contains the `<app-root>` tag that Angular uses to bootstrap the application.

> [!NOTE] the only place i can write javascript code in an Angular project is in the `src/index.html` file.

> [!Note] In `angular.json` file you will find array of styles and scripts, you can add your own styles and scripts to this array to be included in the project.

> [!TIP] Angular project lifecycle.
>
> - The Angular project consists of some modules and components.
> - once we run the application the browser opens the main file specified in the `angular.json` file.
> - usually the main file is the `src/main.ts` file.
> - main.ts file is the file that bootstraps the application.
> - the first thing that the main.ts file does is to import the `AppModule` module.
> - the `AppModule` module is the main module of the application.
> - the `AppModule` module is the module that contains all the components of the application.
> - the `AppModule` by default have a component called `AppComponent`.

> [!tip] All the imports in angular are classes.
