# Angular Universal + WordPress

### 27.1.2017 @ Frantic Tech Meeting

## Jussi Kinnula

---

# Core Concepts

___

## Angular Universal

- Set of tools and components to create Angular application to be ran on server side and in browser (e.g. hybrid, isomorphic application)
- It's currently a separate component, but should be later integrated more deeply to Angular (on Angular 4.x version)

___

## WordPress

- Very popular CMS (Content Management Software) used to build blogs and websites
- Wordpress 4.7.x started to include WP-API (e.g. WordPress REST API) version 2
- WP-API is mainly built to extend WordPress page rendering with JavaScript, for example by lazy-loading some part of the content after the initial page was rendered on WordPress side

___

## Server-side Rendering

- Use of "headless browser" or DOM rendering framework to render and evaluate JavaScript on server side
- Most popular project which currently is used for rendering JavaScript-enriched content on server-side is PhantomJS (PhantomJS itself is fork of WebKit)
- Server environment doesn't have all capacities of real a real browser, for example `window` and `screen` are completely missing

---

# Building Blocks

___

## WordPress: Install

```
cd ~/Development
git clone https://github.com/frc/bedrock-on-heroku.git wp-test
cd wp-test/web/app
mkdir themes
git clone git clone https://github.com/frc/Frantic-WP-theme.git
```

Read instructions from:

https://github.com/frc/bedrock-on-heroku

___

## WordPress: Menus

On WordPress side bare minimum is that we to use menus built on WP customizer side. This is achieved by using a plugin:

https://wordpress.org/plugins/wp-api-menus/

___

## WordPress: ACF

To enable more flexible content editing, we are installing ACF (Advanced Custom Fields Pro):

https://www.advancedcustomfields.com/

Also, to access ACF-originated content via WP-API, we need to install ACF to REST API plugin:

https://wordpress.org/plugins/acf-to-rest-api/

___

## Angular -app

```
cd ~/Development
git clone \
    https://github.com/jussikinnula/angular2-universal-wordpress.git
```

Read instructions from:

https://github.com/jussikinnula/angular2-universal-wordpress

Note! The instructions are for older WordPress 4.6.2. The started will be updated shortly, but just a reminder that WP-API doesn't need to be installed as plugin anymore.

---

# Frantic.com

- Custom WP-API end points & extending the default ones
- A lot of different ACF modules, which are transformed to "fat models" on Angular side to format and normalize the data for Angular template use
- Custom multi-level ACF traversing to preload all the content on single load (opposite of "lazy-loading")
- Multi-language with Polylang -- a lot of hassle of setting the Polylang language "early enough" on WP-API side, so that taxonomy-relations work properly for all ACF content

---

# Thank you!

GitHub repository:
https://github.com/jussikinnula/angular2-universal-wordpress

Slides:
https://jussikinnula.github.io/angular-universal-wordpress-frantic-20170127
