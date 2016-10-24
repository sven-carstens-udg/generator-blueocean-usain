# Overview

This is a [Yeoman] generator for creating a very simple Jenkins plugin (HPI) project that contains a Jenkins [Blue Ocean] UI client-side (JavaScript) Extension Point implementation.

The Extension Point being implemnented is `jenkins.pipeline.run.result`, which allows plugins to contribute to the "Run Details" page.
In the case of the plugin generated by this generator, the Run Details page gets some animations while the build is running and then after run is complete.

![Usain](https://raw.githubusercontent.com/tfennelly/generator-blueocean-usain/master/images/running.gif)

# Prerequisites

1. [Node and NPM](https://nodejs.org/). Required by [Yeoman]. You'll need these anyway if you're developing Blue Ocean plugins/extensions. 
1. [Yeoman] - `sudo npm install -g yo`.
1. Java, Maven etc. Basically, the prerequisites for building any Jenkins HPI plugin.

# Install

Installing this [Yeoman] Generator is very easy:

```
sudo npm install -g generator-blueocean-usain
```

Don't forget to install [Yeoman] itself i.e. `sudo npm install -g yo`. See Prerequisites.

# Run

With everything installed, running the generator is very simple:

```
yo blueocean-usain
```

And then just follow the on-screen instructions in your console, using a Pipeline script similar to the
following (optional).

```groovy
node {
   stage 'Stage 1'
   echo 'Hello World'
   sleep 10
}
```

After running the pipeline job and going to the Run Details page (follow on-screen instructions),
you should see something similar to what is shown in the animated GIF at the top of this page.

# Plugin Anatomy
 
The plugin generated by this generator demonstrates a number of things that should be useful to Plugin Developers interested in developing/porting plugins to work in Blue Ocean.

![Anatomy](https://raw.githubusercontent.com/tfennelly/generator-blueocean-usain/master/images/anatomy.png)

The following sections explain the relevance of the different parts.

## How to Implement a Client-Side (JavaScript) Extension Point

Two resources need to be added to your plugin when implementing an Extension Point:
 
1. The plugin's Extension Point definition file where all Extension Points in the plugin are defined. This file needs to be named `jenkins-js-extension.yaml` and needs to be placed in `src/main/js` (the root of your JavaScript source).   
1. The `.jsx` component file that implements the Extension Point. This can be named whatever you like (so long as it's a `.jsx` file), but must be placed relative to `jenkins-js-extension.yaml`. In this case, the `.jsx` file contains a [React] component (that gets rendered in the view), but the `.jsx` component can also contain other non-rendered Extension Point implementations. That, however, is a topic outside the scope of this tutorial. 

![JavaScript](https://raw.githubusercontent.com/tfennelly/generator-blueocean-usain/master/images/js-components.png)

As stated earlier, the plugin generated by this generator implements the `jenkins.pipeline.run.result` Extension Point (contributing content to the Run Details page).
The implementation of that Extension Point is obviously contained in [Usain.jsx], but we must declare/define that in `jenkins-js-extension.yaml` in order for that
Extension Point to be "picked up" by Blue Ocean. 

```yaml
#
# Extension point implementations in this plugin.
#
# This file tells Blue Ocean what Extension Point components are in this
# plugin + what extension points they implement.
#

extensions:
  - component: Usain
    extensionPoint: jenkins.pipeline.run.result
```

> __NOTE__: We may add support for other formats (other than `.yaml`) in the future. We could also look at discovery mechanisms ala the `@Extension` annotation.

## How to Add Style, images etc for your Plugin Extension Point using LESS

Adding style/CSS, images etc for your plugin Extension Point implementations is easy. We are currently using [LESS] for processing all CSS (we might add support for SASS etc in future).

Simply create a folder named `src/main/less` and into place a file named `extensions.less`. All other web assets (fonts, images etc) should also be placed under this folder.

![Style](https://raw.githubusercontent.com/tfennelly/generator-blueocean-usain/master/images/style-components.png)

## How to use SSE events in your Client-Side (JavaScript) Extension Point

An important part of how the Blue Ocean UI works (and is different from classic Jenkins) is in how it uses [Server Sent Events (SSE)](https://github.com/jenkinsci/sse-gateway-plugin) pushed to it from the Jenkins server/backend. SSE helps Blue Ocean UI to become more real-time, as well as removing the overhead of clients constantly/needlessly polling the Jenkins backend.

The plugin generated by this generator uses SSE events to drive the Usain Bolt animation, using "job" channel events pushed from the backend to modify the Extension Point's state ([React] component state)
based on the running state of the pipeline run being observed on the Run Details page. See the animated gif at the top of this page.

Blue Ocean plugins __do not__ create and maintain their own connections to the [SSE Gateway Plugin](https://github.com/jenkinsci/sse-gateway-plugin). Instead, they use
and share a single `SSEConnection` via the `@jenkins-cd/blueocean-core-js` i.e.
 
```javascript
import { sseConnection } from '@jenkins-cd/blueocean-core-js';
```

With the connection in hand, you can `subscribe` and `unsubscribe` event listeners for different events e.g. as done in [Usain.jsx]. For more on this, see the [SSE Gateway API](https://github.com/jenkinsci/sse-gateway-plugin) docs.

# NPM Scripts

The `package.json` in the generated plugin contains a number of [NPM scripts] that are useful to know. You mostly need to know these scripts in order to streamline your development process.
As is typical with [NPM scripts], they are run by executing `npm run <script-name>` on the command line e.g. `npm run build`.

| name  | Description |
|-------|-------------|
| `build` | Build JavaScript artifacts and add them to your HPI + run tests. Same as running `bundle`, `test` and `lint`. |
| `bundle` | Build JavaScript artifacts and add them to your HPI, but do not run tests or linting. |
| `test` | Run tests only (`src/test/js/**/*-spec.js`). |
| `lint` | Run linting only. |
| `lint:fix` | Run linting and fix fixable lint errors. |
| `bundle:watch` | Run `bundle` continuously, rerunning on source changes. |

The `package.json` in the generated plugin also contains the `mvnbuild` and `mvntest` scripts that get executed as part of the maven build.

# More Advanced Builds

The builds in these plugins are built on top of [@jenkins-cd/js-builder], which in turn is built on top of
[`Gulp`](http://gulpjs.com/) (and others). The [NPM scripts] (see above) use the [@jenkins-cd/js-builder] CLI (`jjsbuilder`) by default (but can use whatever you like).

`jjsbuilder` relies on an internal/"default" `gulpfile.js` when it executes, building Blue Ocean extensions etc. This is fine in many cases, but sometimes you need to
customize the plugin build a little. To do this, simply add a `gulpfile.js` in the root dir of the plugin. When doing this however, be sure to at least `require` the
`@jenkins-cd/js-builder` package, allowing it the opportunity to register its default gulp tasks. 


[Blue Ocean]: https://jenkins.io/projects/blueocean/
[Yeoman]: http://yeoman.io/
[React]: https://facebook.github.io/react/
[LESS]: http://lesscss.org/
[SASS]: http://sass-lang.com/
[NPM scripts]: https://docs.npmjs.com/misc/scripts
[@jenkins-cd/js-builder]: https://www.npmjs.com/package/@jenkins-cd/js-builder
[Usain.jsx]: https://github.com/tfennelly/generator-blueocean-usain/blob/master/app/templates/src/main/js/Usain.jsx
