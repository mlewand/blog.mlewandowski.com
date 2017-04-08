---
layout: post
date: 2017-04-08 19:03:20 +0100
title: Quick CKEditor 4 plugin creation with Yeoman generator
tags: [ckeditor,yeoman]
image:
  feature: construction.jpg
  teaser: construction-teaser.jpg
---

For last couple of weeks I've been playing a little with [Yeoman](http://yeoman.io) and so far I'm enjoying the philosophy of this project. While working on CKEditor 4 we sometimes need to create some smaller plugin, to quickly mock something up.

In the past I've already created some Python scripts for generating a proper plugin for me, however it was pretty much untested XP thing, not really maintained in a way I'd like it to be. So I asked myself: why not create a yeoman generator for CKEditor 4?

tl;dr I'll only describe how to create a plugin with [CKEditor 4 Generator](https://github.com/mlewand/generator-ckeditor4), as there already are many great tutorials on how to write a generator.

## Installation

You need to install yeoman and the generator itself:

```bash
npm install -g yo generator-ckeditor4
```

And that's all!

## Creating a Plugin

Since in generator CKEditor is referenced as a npm module, you don't have to worry about downloading CKEditor or anything.

Just go to your workspace and type:

```bash
yo ckeditor4:createPlugin myplugin --dialog --button
```

Where `myplugin` is the name of your plugin. It will automatically create a subdirectory in your cwd.

## Testing

A great benefit of this is that you have test platform set out of the box!

You can run server with `npm run test-server` and open `http://localhost:1030`.

You can also run all the tests at once using `npm test`.

More information on testing platform in [CKEditor Testing Environment](http://docs.ckeditor.com/#!/guide/dev_tests).

## Dialogs and Buttons

As seen in example from "Creating a Plugin" section, you can create dialogs and buttons so you don't have to worry about file name typo or anything! However these are optional.

## Type Definitions

By default it will also put a proper typings package into `package.json` so that you get a good code completion in Visual Studio Code.

## Conclusions

All in all this generator is a big help in setting up a complete CKEditor 4 plugin infrastructure.

Looking into a future there are couple of things I'd like to improve:

* Definitely building process. As current CKE4 build process could be greatly simplified and modernized (integrating it with npm, git etc).
* More modular approach. I should split more things into sub generators. This is something I have learned down the road, so for instance you'll see that `createPlugin` sub generator has a lot of responsibilities. More recent parts have it's things already extracted.

Hope you'll find this generator helpful. If you have any suggestions feel free to drop an issue on [GitHub](https://github.com/mlewand/generator-ckeditor4).

Keep in mind that this is my pet project, it's not supported by CKSource. So if you see any issues the best way is to drop a pull request 🍺!