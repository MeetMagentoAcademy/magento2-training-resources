# Compile LESS with Grunt

The topic describes how to install, configure and use Grunt JavaScript task runner for compiling `.less` files in Magento 2.

## Prerequisites
- Node.js (LTS version are recommended)
- `grunt-cli` package instaled globaly

## Installing and configuring Grunt
1. Rename `Gruntfile.js.sample` to `Gruntfile.js`
2. Rename `package.json.sample` to `package.json`
3. Install dependecies `npm install` or `yarn`
4. Add you theme to config file `dev/tools/grunt/configs/themes.js`
5. Complie styles using `gulp less:<theme>`

There are more tasks avalivable, but... it's slow, not maintained and overcomplicated.
