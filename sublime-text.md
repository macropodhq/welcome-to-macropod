# Configuring Sublime Text

[Sublime Text 3](http://www.sublimetext.com/3) is the de facto editor of choice at Macropod. Included here are some of the things we use to integrate it into our workflow.

_**Note**: After following these instructions, it is likely that you will need to restart Sublime Text to make use of it._

## Basic Editor Configuration

Most of our projects provide an `.editorconfig` to set basic configuration consistently despite each developer's own preferences. We use [editorconfig-sublime](https://github.com/sindresorhus/editorconfig-sublime) to integrate this with Sublime Text.

1. Open the command palette with Command+Shift+P (Control+Shift+P on Windows and Linux)
2. Select "Package Control: Install Package" from the command palette, and wait for the package list to load
3. Type `EditorConfig` into the palette and select it, then wait for the package to install

## For JavaScript and React

For our projects, we use some language features which are newer than Sublime Text's default syntax highlighting supports.

Following these instructions will help you set up your copy of Sublime Text to work consistently with our projects' modern features, and give you output from our linting setup.

### ES6 and JSX support

For ES6 and JSX support, we use [babel-sublime](https://github.com/babel/babel-sublime).

1. Open the command palette with Command+Shift+P (Control+Shift+P on Windows and Linux)
2. Select "Package Control: Install Package" from the command palette, and wait for the package list to load
3. Type `Babel` into the palette and select it, then wait for the package to install

### Preparing for Linting

For linting, we use [eslint](http://eslint.org) and for ES6 and JSX linting support, [babel-eslint](https://github.com/babel/babel-eslint). For Sublime Text integration, we use [SublimeLinter](https://github.com/SublimeLinter/SublimeLinter3) and [SublimeLinter-eslint](https://github.com/roadhump/SublimeLinter-eslint).

1. Install the command-line dependencies system-wide
   `npm install -g eslint eslint-plugin-react babel-eslint`
2. In Sublime Text, open the command palette with Command+Shift+P (Control+Shift+P on Windows and Linux)
2. Select "Package Control: Install Package" from the command palette, and wait for the package list to load
3. Type `SublimeLinter` into the palette and select it, then wait for the package to install
4. Repeat steps 2-3, then type `SublimeLinter-contrib-eslint` into the palette and select it, then wait for the package to install
