---
layout: post
date: 2017-03-31 15:20:12 +0100
title: Debugging Karma tests with VSCode
---

## Introduction

After talking with one fellow programmer I decided to get back to my old mouselessNavigator chrome extension. The code was not maintained for couple of years, and it was written in good old ECMAScript 3 spirit.

The extension is working heavily with DOM, and since I'd like to also push it to browsers other than Chrome, it made sense to try out [Karma](https://karma-runner.github.io) for cross browser local testing. Since I had already some decent unit tests in the project, this meant that I'll have to port these tests to Karma.

While adjusting tests I found one skipped failing test, and since I was totally unfamiliar with the code base, I decided that the sooner I'll use debugger, the more benefits I'll have - as I really, _really_ love VSCode's debugger. In case you need more information on this see:

* https://code.visualstudio.com/docs/introvideos/debugging
* https://code.visualstudio.com/docs/editor/debugging

## Preparation

All right, so in order to get this going we'll need three things:

1. (Obviously) VSCode installed on our desktop.
1. [debugger-for-chrome](https://marketplace.visualstudio.com/items?itemName=msjsdiag.debugger-for-chrome) extension.
1. (Obviously) a project using Karma.

### VSCode Installation

You can grab your installation file at [Visual Studio Code](https://code.visualstudio.com/) home page.

### Debugger for Chrome Extension

You can install it simply by pressing `ctrl/cmd+p` and typing `ext install debugger-for-chrome` and hit `enter`.

### Karma Project

I'll use Yeoman [Karma generator](https://www.npmjs.com/package/generator-karma) to create a new project.

So first off install Yeoman and the generator:

```bash
npm install -g yo generator-karma
```

Then make a directory for your project and use generator. Note that I'm explicitly marking that I want to use Karma with Chrome in `--browsers` argument, otherwise PhantomJS would be installed. To see information on other arguments, see [generator-karma docs](https://github.com/yeoman/generator-karma/blob/master/readme.md).

```bash
mkdir karma-and-vscode-debugging
cd karma-and-vscode-debugging
yo karma --browsers "Chrome" --app-files "src/**/*.js" --test-files "test/**/*.js" --base-path ".."
```

Let it create all the necessary files and download the dependencies. Once that's done you want to open it's config with VSCode, so just use:

```bash
code . test/karma.conf.js
```

## Configuration

The whole trick is to make Chrome Debugger extension to listen to Chrome instance launched by Karma, using [Chrome Debugging Protocol](https://developer.chrome.com/devtools/docs/debugger-protocol). This is very simple, you just need to make sure you're using a correct port.

### Karma Configuration

First let's set the port in Karma's configuration.

Go back to your Karma's configuration file `test/karma.conf.js` and add `customLaunchers` property, like so:

```javascript
  customLaunchers: {
    ChromeDebugging: {
      base: 'Chrome',
      flags: [ '--remote-debugging-port=9333' ]
    }
  },
```

Note `--remote-debugging-port` flag, which sets listening port to `9333`.

Now change `browsers` property value to:

```javascript
    browsers: [
      'ChromeDebugging'
    ],
```

So that Karma will launcher settings that were specified in listing above.

### VSCode Configuration

Press `f1` to invoke command toolbox and type `> Debug: Open launch.json` and press `enter`. This will open your `.vscode/launch.json` file.

By default it will be created with some standard/preset content, we'll override it to keep it clean.

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "chrome",
      "request": "attach",
      "name": "Attach Karma Chrome",
      "address": "localhost",
      "port": 9333,
      "pathMapping": {
        "/": "${workspaceRoot}",
        "/base/": "${workspaceRoot}/test/"
      }
    }
  ]
}
```

Note how config specifies `pathMapping` property, as by default Karma will serve your files prefixed with `/base/` directory. Actually there's one lesson I learned while doing this: currently your `pathMapping` entires [must end with tailing slash](https://github.com/Microsoft/vscode-chrome-debug/issues/393).

## Testing

Configuration is done. Now it's time to use it in practice.

Add a dummy file that will create a global `myFunction` function.

```javascript
window.myFunction = function( e ) {
  if ( e >= 10 ) {
    return true;
  } else {
    return false;
  }
}
```

Save it as `src/myFunction.js`. Now let's create an actual test.

```javascript
describe( "A suite is just a function", function() {
  it( "and so is a spec", function() {
    let res = myFunction( 15 );

    expect( res ).toBe( true );
  } );
} );
```

Save it as `test/example.js` file.

Now run `npm test` to start watching tests.

### It's Debugging Time!

Enable debugger using either:

* `f5` hotkey,
* "Start debugging" button in Debug panel.

Now the fun part! Put a breakpoint in `test/example.js` file, say at 3rd line. And just press `ctrl/cmd+r` to refresh browser window attached to the debugger. As it gets to your line it will stop the execution, and let you inspect your variables and do all your debugging business.

## Conclusions

I have placed all the files in [karma-and-vscode-debugging](https://github.com/mlewand/karma-and-vscode-debugging) repository, so you can check it out for full sources.

At this point you might want to customize your paths, as for instance I like to have `karma.conf.js` file sitting in the root directory. So after moving the file I need to adjust `basePath` property of `karma.conf.js` and `pathMapping` of `.vscode/launch.json` file. In case you have troubles matching the files use [`.scripts` debugger command](https://github.com/Microsoft/vscode-chrome-debug#the-scripts-command) for easier troubleshooting.

You also might want to have karma instance automatically started if not running, but I guess I'll cover it some time later.