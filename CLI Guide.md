# CLI Guide

### Table of Contents

* [Angular CLI Guide](#angular)
* [Spring CLI Guide](#spring)
* [GIT CLI Guide](#git)
* [Bash CLI Guide](#bash)
* [PWS CLI Guide](#pws)

## Angular

- Must install [Angular CLI](https://angular.io/cli) tool first.

- Must install [node](https://github.com/coreybutler/nvm-windows) first.

- To show current version `nvm` OR `node -v` OR `npm -v`

#### Setup:

`npm install -g @angular/cli` **installs latest version** (-g globally in your machine).

`ng` OR `ng help` **shows available commands**.

#### Create:

`ng new project-name` creates a **new angular project**.

`ng generate component component-name` OR `ng g c component-name` generates a **new component**.

`ng generate service service-name` OR `ng g s service-name` generates a **new service**.	

`npm install --save @angular/material @angular/cdk @angular/animations` **installs angular material**.

#### Execute:

`ng serve` **serves an Angular project** via a development server.

`ng serve --port 4201` serves an Angular project on the **specified port**.

`ng test` runs **tests**.

`npm start` **runs whatever you have defined** for the `start` command of the `scripts` object in your `package.json` file.

## Spring

#### Gradle Projects

`./gradlew clean` **delets the build** directory.

`./gradlew build` **builds** the application

`./gradlew bootRun` **runs** the application.

`./gradlew clean build bootRun` does **all of the previous**.

`java -jar build/libs/gs-testing-web-0.1.0.jar` **runs the Jar file**, similar to `bootRun`

`./gradlew test` runs **tests**.

#### Maven Projects

`./mvnw clean` **delets the build** directory.

`./mvnw package` **builds** the application

`./mvnw spring-boot:run` **runs** the application.

`./mvnw clean package spring-boot:run` does **all of the previous**.

`java -jar target/gs-testing-web-0.1.0.jar` **runs the Jar file**, similar to `bootRun`

`./mvnw test` runs **tests**.

## GIT:

`Git clone [[link]]` **clones** the remote repository to a local folder using the provided `[[Https link]]`. 

`Git checkout -b [[branchName]]` : **creates a new branch** called `[[branchName]]`. To move to an existing branch we can remove `-b`.

`Git add -A` : **adds files to be committed** to the index. `-A` is used to add all modified files. Instead, we can also specify a folder or a file by name.

`Git commit -m [[commitMessage]]` **commits** the staged files with the given `[[commitMessage]]`. 

`Git push origin [[branchName]]` **pushes** commits to the specified `[[branchName]]` of the remote repository.

`Git pull origin [[branchName]]` **pulls** from `[[branchName]]` of the remote repository

`Git reset --hard` **git rid** of all unwanted modified files.

`Git help` shows all **available commands**.

## BASH:

`cd ~` navigates to the **home directory**.

`cd ..` navigates **up one directory**.

`cd [[childfolder]]` navigates to the **child directory**.

`mkdir [[newFolder]]` creates a **new folder**.

`touch [[newFile]]` creates a **new file**.

`ls` **list all contents** of the current directory.

`pwd` shows **path** of the current directory.

`explorer` opens the **windows explorer**.

## PWS:
	
`cf help` shows all **available commands**.

`cf login -a api.run.pivotal.io` **logs in** to the PWS account.

`cf push [[your_app]]` **deploys** app to PWS servers.