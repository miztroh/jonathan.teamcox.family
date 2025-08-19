---
{"publish":true,"title":"Embracing Constraints","created":"2025-02-10T09:30:59.797-06:00","modified":"2025-02-10T09:30:59","published":"2025-08-19T12:59:58.601-05:00","tags":["Angular","Firebase","Go","JavaScript","Lit","Material-Design","Node-JS","PocketBase","SQLite","Vaadin-Router","Memory-Safety","Wiz","TypeScript","Google","Microsoft","Rollup","NgRx","RxJs","IP-Development"],"cssclasses":""}
---






Recently, I've had to make some hard choices regarding application architecture.  Before I walk you through the things to come, let's take a step back and review the history of this app.

## Back in Time

![[blog/2025/02/12/embracing-constraints/huey-lewis-and-the-news.png]]

The app in question began with a [Node.js](https://nodejs.org/) and [SQLite](https://sqlite.org/) backend and web frontend comprised of [Material Design web components](https://m3.material.io/develop/web), custom [Lit](https://lit.dev/) web components, and the [Vaadin Router](https://github.com/vaadin/router).  Database maintenance (users, access, schema migrations, etc.), authentication, and routing were done manually in code.  It was far from perfect, but met the needs of the business and its users.  Eventually, poor developer ergonomics motivated me to reconsider this approach.

## Round 2

![[blog/2025/02/12/embracing-constraints/mortal-kombat.webp]]

By this point, I was tired of managing database maintenance, authentication, and routing.  It felt like reinventing the wheel.  As one does, I scoured the web for a solution to my problems.  That's when I discovered [PocketBase](https://pocketbase.io/).  If you've come across [Firebase](https://firebase.google.com/), PocketBase should feel familiar.  It offers a web server, a realtime database, authentication, file storage, and an admin dashboard.  As advertised, it's an "open source backend in 1 file".  Unlike Firebase, you can self-host PocketBase.  #SelfHostAllTheThings

With PocketBase, the database is still SQLite, but with major quality-of-life improvements.  For example, any time you create / change table schema, PocketBase creates a JavaScript migration file.  These migration files live in the repo alongside other code and ensure each app deployment is serving the correct schemas.  This is a major win over the manual schema migration processes of the past.  Authentication is built atop the database functionality and can integrate with all sorts of external providers.  Finally, the admin dashboard is a user interface for creating tables and records, managing users and auth providers, viewing logs, and configuring app settings.  It also includes a robust server API in both Go and JavaScript for interacting with data, serving custom routes, sending emails, and more.  The client-side JavaScript API supports realtime notifications when data changes.  It's a one-stop stop for quite a lot of functionality.

I immediately set about rearchitecting the app around PocketBase.  I was excited to learn Go for the server side portions.  But, there were unforeseen issues ahead.

## You've Got Your Troubles, I've Got Mine

![[blog/2025/02/12/embracing-constraints/argus-filch.png]]

The first problem was the amount of effort required to reimplement the app's API in Go.  It quickly felt like reinventing the wheel.  In truth, I was duplicating many of the efforts I had put into the Node.js backend.  While it was fun learning a new language, especially one with the benefit of [memory safety](https://en.wikipedia.org/wiki/Memory_safety), the costs outweighed the benefits.  I would need to tackle server side concerns in another way.

The second problem was one I did not see coming: [Google's Material web components library entering maintenance mode](https://github.com/material-components/material-web/discussions/5642).  Essentially, [Google decided to merge its Wiz and Angular framework teams](https://blog.angular.dev/angular-and-wiz-are-better-together-91e633d8cd5a).  That meant resources (personnel and otherwise) for other web framework initiatives within Google (such as MWC) were reallocated elsewhere.  MWC had served as the UI library for my app.  The app's entire design language was sourced from these components and Google's Material design language.  I could no longer count on future support / development of this UI library and I would need to find another.

## Looking at Things from a Different Angle

![[blog/2025/02/12/embracing-constraints/angular.png]]

I decided to tackle my client-side troubles first.  As Material is the design language of my app, I wasn't so quick to abandon that.  I learned that Google's [Angular](https://angular.dev/) framework has its own implementation of Material, suitably named [Angular Material](https://material.angular.io/).  *"Great!"*, I thought.  *"A full suite of ready-to-use Material web components.  Problem solved!"*.  Not so fast.

Angular leverages [TypeScript](https://www.typescriptlang.org/).  For years, I had deftly avoided it.  I'm something of a JavaScript purist.  #UseThePlatform, am I right?  But ... all the cool kids are using it.  Big organizations like Google and Microsoft, plus plenty of smaller open source projects.  It was time to get over myself.  I had already spent time learning Go and enjoying all of its type syntax goodness.  After spending a couple of hours experimenting with TypeScript, I had to admit it: I was sold.  I had learned enough of the Angular framework to get started and so I set about reworking my client-side code.  After getting my frontend up and running, it was time to solve my server-side problems.

## I've Got Server-Side Problems, But TypeScript Ain't One

![[blog/2025/02/12/embracing-constraints/typescript.png]]

As mentioned, PocketBase offers a couple of server side API options.  Go was out; Only JavaScript remained.  TypeScript is not directly supported, as it must be transpiled to JavaScript in order be executed by the PocketBase JavaScript virtual machine (JSVM).  The transpilation has to be part of the application's build process to ensure all required resources are available at runtime.  This is what Angular's ``ng build`` command does for client-side resources.  Another common tool for bundling is [Rollup](https://rollupjs.org/).  I was able to leverage Rollup to transpile my TypeScript PocketBase hooks (server-side API) at build time.  This enables development of PocketBase hooks in TypeScript, leveraging both the type definitions that PocketBase provides ***and*** the custom types from my client-side code.  Now we're cooking with gas!

## Back to the Future

![[blog/2025/02/12/embracing-constraints/back-to-the-future.png]]

So, where does all this leave us?  Let's recap the benefits of these changes:

- By leveraging Angular (along with [RxJs](https://rxjs.dev/) and [NgRx](https://ngrx.io/), I get a wholistic client-side framework for the application, components, shared state, reactivity, animations, and UI ... all while enjoying the benefits of TypeScript (strongly typed syntax, editor integration, etc.).
- By implementing PocketBase, I've alleviated the need to write code to manage the database and its schema / schema migrations, users, and authentication.  Additionally, we can continue to develop our server-side code in JavaScript (TypeScript), enabling reuse of existing JavaScript code from the Node.js implementation.

These benefits were the result of specific architecture choices and a lot of hard work.  Why was it necessary to revisit the architecture?  **Environmental constraints**.  The app is constrained in many ways:

- **The features it is expected to provide.**  I can't simply bail on the entire app as it exists today or push forward knowing that I can't support some of the existing features under a new architecture.
- **The interface to which users are accustomed.**  In this case, the existing UI library has entered maintenance mode.  At some point, it may cease to function or a security issue may be identified.  It's better to proactively eliminate from the environment rather than waiting until it becomes urgent.
- **The amount of time that can be spent on further development / maintenance.**  "Reinventing the wheel", "throwing the baby out with the bath water" ... pick your idiom.  Minimizing the amount of future effort required to move forward while maximizing the amount of benefit retained from previous effort is essential.

Every environment has constraints.  You can try to work around them or break through them, but you will only end up breaking yourself against them.  Instead, you must embrace the constraints.  Understand which of them are hard rules and which are merely guidelines.  Once you frame your thought process in the context of your constraints, you're free to be truly creative in solving whatever problems come your way.