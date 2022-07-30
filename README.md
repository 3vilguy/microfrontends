# Microfrontends

Playing around with microfrontends using Webpack Module Federation.

# Notes

## What are microfrontends?
- Divide a monolithic app into multiple, smaller apps
- Each smaller app is responsible for a distinct feature of the product
- There usually is some sort of "Container" app, that decides when/where to show each Microfrontend

## Why use them?
- Multiple engineering teams can work in isolation
- Eaach smaller app is easier to understand and make changes to

## Integration
(<em>How and when does the Container get access to the source code of Microfrontends</em>)

### Major Categories of Integration:
- <strong>Build-Time Integration</strong> (Compile-Time Integration) - Before Container gets loaded in the browser, it gets access to the MF source code
  - For example using MF source as dependency
- <strong>Run-Time Integration</strong> (Client-Side Integration) - After Container gets loaded in the browser, it gets access to the MF source code
  - For example MF is being deployed separately to http://my-app/something.js, and Container app loads and execute this file (using Webpack Module Federation).
- <strong>Server Integration</strong> - While sending down JS to load up Container, a server decides on whether or not to include MF source


## Implementing Module Federation (integration process)
- Designate one app as the <strong>Host</strong> and one as the <strong>Remote</strong>
  - <strong>Host</strong> is the application that is trying to make a use of code from another project (Container).
  - <strong>Remote</strong> is the project that is making the code available to other projects.
- In <strong>Remote</strong>, decide which modules (files) you want to make available to other projects.
- Setup up the Module Federation plugin to expose those files.
- In the <strong>Host</strong>, refactor the entry point to load asynchronously
- In the <strong>Host</strong>, import whatever files are needed from the remote

### Sharing dependencies between Apps
- In plugins, ModuleFederationPlugin settings, add `shared: ['library-to-share']` option, to specify array of libraries that we want to share. Add it in all apps that we want the library to be shared. If `major` version of the library is the same, then only single lib will be loaded. If not, multiple versions will be loaded.
  - For standalone testing, don't forget to add `import(./bootstrap)` in index.js, for Webpack to figure out what has to be loaded.
- In case we want to make sure only 1 instance of library can be loaded (for example React), we can use singleton flag in ModuleFederationPlugin settings:
  ```json
  "shared": {
    "library-to-share": {
      "singleton": true,
    },
  },
  ```
  And now in case of incompatible versions there would be a console warning about it.

### Executing Context
We should NOT assume that some DOM element would exist in Container app, so we should export a function that can be called from the Container app (and pass DOM element as parameter). Also for local development, some extra code would be required, just to check if we're running it in isolation.