# Angular Universal + WordPress

### 1.2.2017 @ Angular Finland Meetup, in ZEF, Oulu

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

## WordPress: Install & Run

```
cd ~/Development
git clone https://github.com/jussikinnula/ \
    angular2-universal-wordpress-wp-test.git wp-test
cd wp-test
composer install
PORT=5001 heroku local
```

Based on:
- https://github.com/frc/bedrock-on-heroku
- https://github.com/frc/Frantic-WP-theme

___

## WordPress: Menus

On WordPress side bare minimum is that we to use menus built on WP customizer side.

This is achieved by using a plugin:

https://wordpress.org/plugins/wp-api-menus/

___

## WordPress: ACF

To enable more flexible content editing, we are installing ACF (Advanced Custom Fields, there's also newer and better "Pro" -version): https://www.advancedcustomfields.com/

Our theme has custom REST functionality for ACF content fetching, however, if you wish to do it with "standard" tools you can use existing ACF to REST API plugin: https://wordpress.org/plugins/acf-to-rest-api/

___

## Angular -app

```
cd ~/Development
git clone \
    https://github.com/jussikinnula/angular2-universal-wordpress.git
cd angular2-universal-wordpress
npm install
npm run dev
```

Some more instructions from: https://github.com/jussikinnula/angular2-universal-wordpress

---

# The Code

---

# Thank you!

GitHub repositories:
https://github.com/jussikinnula/angular2-universal-wordpress
https://github.com/jussikinnula/angular2-universal-wordpress-wp-test

Slides:
https://jussikinnula.github.io/angular-universal-wordpress-oulu-20170201
