# Angular Introduction

Table of Contents

*   [Introduction to the Shadow DOM](#introduction-to-the-shadow-dom)
*   [Introduction to Angular Modules](#introduction-to-angular-modules)
*   [Introduction to Component Design](#introduction-to-component-design)
*   [Creating Components in Angular](#creating-components-in-angular)
*   [Template Binding](#template-binding)
*   [Wrapping Up](#wrapping-up)

Historically, front-end design was somewhat of an afterthought in software development. In a traditional legacy system, the majority of complexity, logic and effort was placed on the server, on which the **_User Interface (UI)_** relied. In this model, known as **_server-side rendering_**, an HTTP GET request is made to an API, which returns a fully-rendered HTML page to the client. The benefit of this approach is that the client browser has very little work to do, because it simply displays the HTML that was pre-rendered by the server. However, the downside is that page load times are high, due to a complete HTML tree re-render that is dependent on a network request.

An alternative approach, known as **_Single-Page Applications (SPA’s)_** has arisen in the past few years. These applications are powered by JavaScript in the browser, and get their name from the fact that all the HTML needed for the application to function is included in the network request made when the user first visits the site. Each 'page' inside the application is rendered dynamically by client-side JavaScript, rather than reaching out to an API to request a new HTML file.

## Introduction to the Shadow DOM

* * *

There are several popular single-page application frameworks used by developers today. The most popular include Angular and React.js. Though the general concepts described before apply to both of these single-page application frameworks, there are key differences that distinguish them, including conventions and application structure. Here at Cognizant, we primarily focus on Angular, which is a Single-Page Application framework developed by Google that utilizes TypeScript and other popular JavaScript libraries to organize your HTML and JavaScript.

Angular functions by creating something called a **_Shadow DOM_**. The Shadow DOM is a cached snapshot of the physical DOM in the browser, used to efficiently query and update the physical DOM. At its core, Angular creates and updates pages by accepting input, processing it through JavaScript, applying any changes to the Shadow DOM, and finally applying those changes to the physical DOM. This process can be illustrated by the diagram below:

{{Insert Diagram Here}}

## Introduction to Angular Modules

* * *

Angular is very opinionated in its implementation. It relies heavily on convention over configuration, meaning that if you follow their conventions, there’s very little configuration that needs to be done. Conversely, customization is difficult because it requires you to work around the existing configuration that it provides. This is due to the fact that it is a full framework; most of what you need in order to scale up an enterprise-scale application is available out of the box. In this sense, it’s similar to other frameworks like Spring Boot and .NET, which provide features like Dependency Injection, routing and more by default.

Angular implements some of these features through special classes known as **_Modules_**. A module is, in essence, a scoped application context for any classes that are managed by it. These modules provide functionality for dependency injection, annotation implementation and more. When you create a new Angular application, it will come with two modules by default: The **_App Module_** and the **_App Routing Module_**. The App Module serves as the context for the entire application by default, and the app routing module resolves routes in the browser and returns a specific component as the value of that route.

The name module comes from the fact that Angular is designed to be modular, meaning that a single application can use multiple modules, and modules can be added or removed in order to add specific, scoped functionality.

## Introduction to Component Design

* * *

Another convention Angular follows is known as **_component-based design_**. At its core, component design encapsulates view logic into individual classes that, when used in conjunction, form a full view.

For instance, take a page that contains a user registration form. In traditional server-side rendered page, this form would be one HTML document composed of div’s, inputs, labels and buttons. This approach is no longer recommended do to the fact that it breaks common Object-Oriented Programming principles such as inheritance and encapsulation. The traditional approach leads to a single HTML page knowing and defining the implementation and functionality of all its pieces, and fails to make code reusable. In this approach, you either use the entire page or none of it.

Component-based design revolves around the principle that each component (think class definition in classic OOP) should accept input (class properties) and return **one** HTML element as a result. Instead of a typical class in OOP that does logic and returns values through methods, component classes return an HTML template. Therefore, when defining a component, you create an HTML template that can be reused across your application without re-defining it over and over. This design pattern brings object-oriented principles to the HTML layer, which makes your view layer code easier to both develop, test and reuse.

Going back to the login form from earlier, Angular encourages a different approach to its design. Instead of one HTML page that contains the entire view, the recommended approach is to create several same-level components that represent elements on the page. For instance, you may create a textbox component, a label component, a button component and so-on. These components would all be incorporated to form the entire page, but abstract the implementation into manageable pieces, making the creation of other forms in the future much faster. The diagram below illustrates the restructuring of the form into components:

{{Insert Diagram Here}}

## Creating Components in Angular

* * *

With the understanding of how component design brings modularity and object-oriented practices to the view layer, it’s important to dive into how to implement component design in the Angular framework. Fortunately, Angular makes the creation and maintenance of components very simple by giving access to the **_Angular CLI_**, a command line interface that helps to generate different parts of an Angular application. A component can be created by running the 'ng generate component' command in the CLI. Once created, you’ll notice a few important details about your component class:

*   The class is decorated with an @Component annotation
    
*   Four files were created inside a folder: HTML, CSS, Typescript and a Test File
    

The Component annotation is used to mark the class as a special kind of Angular class, similar to the way modules are reserved in Angular. The annotation provides several properties, but three of them are supplied by default: Selector, TemplateUrl and StyleUrls.

The **_selector_** property sets the name of the HTML tag that the component should be invoked by in the template (more on this later). For example, a HelloWorld component would have be invoked with the <app-hello-world></app-hello-world> tag. Since this is not a tag that the HTML DOM can render, the Angular Module must specifically target the selector and replace it with the contents of the component it represents.

The **_TemplateUrl_** property gives the Angular module the path to the HTML template file that represents the content of the component. The same principle applies to the **_StyleUrls_** property, except that it points to one or more CSS files for the component. These files are generated as part of the CLI script discussed earlier.

## Template Binding

* * *

What makes components dynamic and reusable is the fact the HTML returned from the component is not static. That is to say, it is dynamically generated by Angular through the TypeScript component file. This is accomplished through something known as **_Template Binding_**. Template Binding is the process by which the Angular runtime takes a component class, binds its data to an associated template, then registers that compiled HTML template to the Shadow DOM.

The data binding necessary to create a dynamic HTML template happens through specific syntax provided by Angular. There are three main types of data binding:

*   Interpolation
    
*   One-Way Data Binding
    
*   Two-Way Data Binding
    

**_Interpolation_** is an Angular feature that outputs the value of a class property in the HTML template. It can be utilized by wrapping the property name in two curly braces. For example, if you wanted to output the value of a property called foo, the syntax would be: {{foo}}, which would be rendered as a string in the HTML template. Any values that are bound to the template will be monitored by Angular for changes and dynamically updated on the web page. When the value changes, Angular updates the Shadow DOM with the new value of the property, which eventually renders to the physical DOM.

Interpolation is one form of a data binding known as **_One-Way Data Binding_**, which allows a component or template to read the value of a property, but not modify it. Interpolation is a type of one-way data binding. Alternatively, **_Two-Way Data Binding_** allows both reading and writing to the value of a given property. Two-way data binding is accomplished through using a property called **_NgModel_**. NgModel is a property on a class that is dynamically managed by Angular. An example of an NgModel is a textbox value that a user creates. The value displayed in the textbox is read from a class property, and whenever the user types in text, the property is updated to that new value. NgModel can be applied to an input in the using the following syntax:  
<input type="text" \[(ngModel)\]="componentPropertyName"/>  
That special syntax tells Angular to take control of that form control and bind its state to whatever component property is specified in the quotation marks.

## Wrapping Up

* * *

Angular is a powerful JavaScript framework that abstracts much of the complexity of managing the DOM and makes and efficient platform for web applications. Always remember that behind the scenes Angular is still implementing JavaScript to manage state and manipulate the DOM. Often times, you can resolve bugs and issues in your application by understanding the JavaScript running beneath it. Though the framework is large and there are many niches, if you can understand these fundamentals and the way that Angular implements the functionality, it will help you understand the more advanced concepts that come later.