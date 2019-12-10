# Angular

Angular is a typescript based framework for developing component based applications, it is NOT the same as AngularJS.

## Basics

The basic building block of an angular app is a component, which is comprised of 4 files.

*   An .html file which is commonly referred to as a template.
    
*   A .ts file which is commonly referred to as the component.
    
*   A .css (or other type of cascading style sheet file such as .scss) which is used for styling.
    
*   A .spec file which is used for testing.
    
Other notable files are module.ts and service.ts files.

## Getting Started

Angular has 2 prerequisites needed before it can be used, node.js and an npm package manager. For more thorough getting started information [Angular’s getting started page](https://angular.io/guide/quickstart)

### Updating NodeJS:

Go to `nodejs.org` and download the latest version - uninstall (all) installed versions on your machine first.

### Updating npm:

Run `[sudo] npm install -g npm`  (sudo  is only required on Mac/ Linux)

### Updating the CLI

`[sudo] npm uninstall -g angular-cli @angular/cli`
	
`npm cache clean`
	
`[sudo] npm install -g @angular/cli`

*Some knowledge of typescript, html and css is highly encouraged.*

### Installing Bootstrap CSS

`npm install --save bootstrap@3` Where 3 is the version we would like to install. *This only installs the Bootstrap locally in the Angular app `node_modules` folder.*

We also need to add bootstrap to `angular.json` under Projects > architect > styles:

	"styles": [
		  "node_modules/bootstrap/dist/css/bootstrap.min.css",
		  "src/styles.css"
		]

## Terms

1.  directive: [Angular’s directive description](https://angular.io/api/core/Directive#directive)
    
2.  service: [Angular’s service information](https://angular.io/tutorial/toh-pt4)
    
3.  module: [Angular’s module information](https://angular.io/guide/architecture-modules)
    
4.  router: Angular uses routing navigation to establish a single page application, [Angular’s routing information](https://angular.io/guide/router)
    
5.  bootstrap: the first component loaded by your application
    

## Commonly Used Aspects

1.  [Angular Material](https://material.angular.io/)
    
2.  [Template Driven Forms](https://angular.io/guide/forms)
    
3.  [Reactive Forms](https://angular.io/guide/reactive-forms)
    
4.  [Tables](https://material.angular.io/components/table/overview)
    

### Useful Resources

1.  [Angular’s Homepage](https://angular.io/)
    
2.  [Alligator.io extensive resource list](https://alligator.io/resources/)
    
3.  [Pluralsight’s Angular Fundamentals Course - PAID](https://app.pluralsight.com/library/courses/angular-fundamentals/table-of-contents)
    
4.  [Scrimba Angular Tutorial](https://scrimba.com/g/gyourfirstangularapp)
    
5.  [Udemy’s Angular 2 Course - PAID](https://www.udemy.com/the-complete-guide-to-angular-2/)
    
6.  [Coursera Angular Course](https://www.coursera.org/learn/angular)
    
7.  [Dzone angular getting started](https://dzone.com/articles/getting-started-with-angular-70)
    
8.  [Tutorialspoint Angular 4 tutorial](https://www.tutorialspoint.com/angular4/index.htm)
    
9.  [A youtube tutorial](https://www.youtube.com/watch?v=WWQZCDegWHg&list=PL6n9fhu94yhWqGD8BuKuX-VTKqlNBj-m6)
    
10.  [A youtube tutorial](https://www.youtube.com/watch?v=htPYk6QxacQ)