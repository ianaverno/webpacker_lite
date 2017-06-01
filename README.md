# Webpacker Helpers
![Gem Version](https://badge.fury.io/rb/webpacker_helpers.svg)

**VERSION 8 of [React on Rails](https://github.com/shakacode/react_on_rails) is super close!!!** [VERSION 8.0.0-beta.3](https://rubygems.org/gems/react_on_rails/versions/8.0.0.beta.3) has shipped with [webpacker_helpers](https://github.com/shakacode/webpacker_helpers) support! Please try [the 8.0.0-beta.3 beta](https://rubygems.org/gems/react_on_rails/versions/8.0.0.beta.3) and please report issues! We're **SUPER** close as we've also upgraded the [shakacode/react-webpack-rails-tutorial](https://github.com/shakacode/react-webpack-rails-tutorial) with [PR #395](https://github.com/shakacode/react-webpack-rails-tutorial/pull/395). That PR shows the changes needed to go to Webpacker Helpers.


*A slimmer version of Webpacker*

Webpacker Helpers provides similar webpack enabled view helpers from [Webpacker](https://github.com/rails/webpacker).
[React on Rails](https://github.com/shakacode/react_on_rails) version 8 and greater defaults to using Webpacker Helpers.

For example, these view helpers allow your application's layout to easily reference JavaScript and CSS files created by your Webpack setup, taking into account differences in the Rails environments. With these helpers, there is no reason for Webpack created assets to run through the [Asset Pipeline](http://guides.rubyonrails.org/asset_pipeline.html), as was done in React on Rails 7.x and earlier.

If you like this project, show your support by giving us a star!

# Why Fork?

> Everything should be made as simple as possible, but not simpler.

[Albert Einstein on Wikiquote](https://en.wikiquote.org/wiki/Albert_Einstein)

Why did [ShakaCode](http://www.shakacode.com) fork [rails/webpacker](https://github.com/rails/webpacker)? For [react_on_rails](https://github.com/shakacode/react_on_rails), we wanted a simpler configuration to get the core functionality needed. You configure 2 things:

1. Name of the manifest file.
2. The directory within `/public` where Webpack will create the manifest and output file.

Then you need to configure your Webpack to generate a simple manifest that maps the base output names to the possibly fingerprinted versions. Note, unlike Webpacker, Webpacker Helpers wants your manifest to **NOT** contain any host information.

Optionally, you can configure the name of the server and port for hot reloading, and if hot reloading is the default for a given Rails.env.

For more details on how this project differs from Webpacker and why we forked, please see [Webpacker Helpers: Why Did  We Fork Webpacker?](https://medium.com/@railsonmaui/webpacker-lite-why-did-we-fork-webpacker-ee3305688d66)

# NEWS
 
* 2017-05-03: React on Rails 8.0.0 beta defaults to using webpacker_helpers.

## Installation

The best way to see the installation of webpacker_helpers is to use the generator for React on Rails 8.0.0 or greater. Otherwise, add the gem and create the configuration file described below.

## Overview

1. Configure the `config/webpacker_helpers.yml` file, as described below. You will specify the name of the manifest file and the output directory used by step 2.
2. Use the [webpack-manifest-plugin](https://www.npmjs.com/package/webpack-manifest-plugin) to generate a manifest
   in the output directory (`webpack_public_output_dir`) that you configured in your `/config/webpacker_helpers.yml` file. 
3. Use the view helpers on your layouts to provide the webpack generated files. Note, these are the same names used by [rails/webpacker](https://github.com/rails/webpacker).
   These `output` names are **NOT** the actual file names, as the file name may have a [fingerprint](http://guides.rubyonrails.org/asset_pipeline.html#what-is-fingerprinting-and-why-should-i-care-questionmark).
   ```erb
   <%# app/views/layouts/application.html.erb %>
   <%= javascript_pack_tag('main') %>
   <%= stylesheet_pack_tag('main') %>
   ```
4. When hot-reloading, the extract-text-plugin (extracted CSS from being inlined in the JavaScript)is not supported. Therefore, all your hot-reloaded Webpack-compiled CSS will be inlined and we will skip the CSS file by default. If you're not worried about hot-reloading for your CSS, use the `enabled_when_hot_loading: true` option. 

   ```erb
   <%= stylesheet_pack_tag('main', enabled_when_hot_loading: true) %> <% # Default is false %>
   ```
   
For more details on the helper documentation, see the Ruby comments in [lib/webpacker_helpers/helper.rb](lib/webpacker_helpers/helper.rb) and please submit PRs here to help us improve the docs!

## Configuration
Webpacker Helpers takes one configuration file: `config/webpacker_helpers.yml` used to configure two required values and a couple optional values. Note, this file is configured like `config/database.yml` in that you place the values beneath the name of the Rails.env. 

### Mandatory Configuration within `config/webpacker_helpers.yml` 

1. `manifest`: The manifest file name 
1. `webpack_public_output_dir`: The output directory of both the manifest and the webpack static generated files within the `/public` directory.

Note, placing output files within the Rails `/public` directory is not configurable.

### Optional Configuration within `config/webpacker_helpers.yml` 
1. `hot_reloading_host`: The name of the hot reloading `webpack-dev-server` including the port
2. `hot_reloading_enabled_by_default`: If hot reloading should default to true

### Hot Reloading Notes

Do not put the output server in your `manifest.json` file. The rails view helpers will automatically prepend the hot_reloading_host to the asset path.

### Example Configuration `/config/webpacker_helpers.yml`

This example config shows how we use different output directories for the webpack generated assets per the type of environment. This is extremely convenient when you want to log redux messages in development but not in your tests.

```yaml
# /config/webpacker_helpers.yml
# Note: Base output directory of /public is assumed for static files
default: &default
  manifest: manifest.json  
  # Used in your webpack configuration. Must be created in the
  # webpack_public_output_dir folder.
  
development:
  <<: *default
  # generated files for development, in /public/webpack/development
  webpack_public_output_dir: webpack/development
  
  # Default is localhost:3500. You can specify the protocol if needed. Defaults to http://.
  hot_reloading_host: localhost:3500
  
  # Developer note: considering removing this option so it can ONLY be turned by using an ENV value.
  # Default is false, ENV 'HOT_RELOADING' will always override 
  hot_reloading_enabled_by_default: false 
  
test:
  <<: *default
  # generated files for tests, in /public/webpack/test   
  webpack_public_output_dir: webpack/test

production:
  <<: *default
  # generated files for tests, in /public/webpack/production
  webpack_public_output_dir: webpack/production
```

## Example for Development vs Hot Reloading vs Production Mode

**erb file**

```erb
  <% # app/views/layouts/application.html.erb %>
  <%= javascript_pack_tag('main') %>
  <%= stylesheet_pack_tag('main') %>
```

**html file**

```html
  <!-- In test mode -->
  <script src="/webpack/test/main.js"></script>
  <link rel="stylesheet" media="screen" href="/webpack/test/main-0bd141f6d9360cf4a7f5.js">
  
  <!-- In development mode -->
  <script src="/webpack/development/main.js"></script>
  <link rel="stylesheet" media="screen" href="/webpack/development/main-0bd141f6d9360cf4a7f5.js">
  
  <!-- In development mode with hot reloading, using the webpack-dev-server -->
  <script src="http://localhost:8080/webpack/development/main.js"></script>
  <!-- Note, there's no stylesheet tag by default, as your CSS should be inlined in your JS. -->
  
  <!-- In production mode -->
  <script src="/webpack/production/main-0bd141f6d9360cf4a7f5.js"></script>
  <link rel="stylesheet" media="screen" href="/webpack/production/main-dc02976b5f94b507e3b6.css">
```

## Other Helpers: Getting the asset path

The `asset_pack_path` helper provides the path of any given asset that's been compiled by webpack.
Note, the real file path is the subdirectory of the public.

For example, if you want to create a `<link rel="prefetch">` or `<img />`
for an asset used in your pack code you can reference them like this in your view,

```erb
<img src="<%= asset_pack_path 'calendar.png' %>" />
<% # => <img src="/webpack/calendar.png" /> %>
<% # real file path "public/webpack/calendar.png" /> %>
```

## Webpack Helper
You may use the [React on Rails NPM Package](https://www.npmjs.com/package/react-on-rails), [react-on-rails/webpackConfigLoader](https://github.com/shakacode/react_on_rails/blob/master/webpackConfigLoader.js) to provide your Webpack config with easy access to the YAML settings. Even if you don't use the NPM package, you can use that file to inspire your Webpack configuration.

## Rake Tasks

### Examples

To see available webpacker_helpers rake tasks:

```
rake webpacker_helpers
```

If you are using different directories for the output paths per RAILS_ENV, this is how you'd delete the files created for tests: 
```
RAILS_ENV=test rake webpacker_helpers:clobber
```

## Differences from Webpacker

1. Configuration setup of an optional single file `/config/webpacker_helpers.yml`
2. Webpacker helpers expect the manifest to contain the server URL when hot reloading. Webpacker Helpers expects the manifest to never contain any host information.

## Hot Reloading

1. Tell Rails and Webpacker Helpers that you're hot reloading by setting the ENV value of `HOT_RELOADING=YES` if you are not hot reloading by default by setting the `hot_reloading_enabled_by_default` key in your config file.
1. By default, the `stylesheet_pack_tag` helper will not create a tag when hot reloading is enabled. Per the note above, when hot-reloading, the extract-text-plugin (extracted CSS from being inlined in the JavaScript)is not supported. Therefore, all your hot-reloaded Webpack-compiled CSS will be inlined and we will skip the CSS file by default.

   ```erb
   <%= stylesheet_pack_tag('main', enabled_when_hot_loading: true) %> <% # Default is false %>
   ```
   


## Prerequisites
* Ruby 2+
* Rails 4.2+
