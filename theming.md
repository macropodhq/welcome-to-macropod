# Theming

Themes are where an applications main *style* lives. This can include color, font, text, background and other styles, but generally excludes structural behaviour. This document covers both **themes** and **values**.

- [Values](#values)
  - [Structure](#value-structure)
  - [Importing](#importing-values)
  - [Composing](#composing)
  - [Color values](#color-values)
- [Themes](#themes)
  - [Using themes](#using-themes)
  - [Combining themes](#combining-themes)

## Values

Values are variables, but you shouldn't re-assign them, so they're more like constants. It's the method of storing values that are used _multiple_ times in a theme or generally throughout your application.

There's some lingering methods, including scss variables and postcss-constants - but new implementations use [postcss-modules-values](https://github.com/css-modules/postcss-modules-values). A few notes:

- You shouldn't be defining values and declarations in a single stylesheet. Seperate the concerns, so you have a stylesheet for values and another for using them in declarations.
- If you feel the need to declare a value outside of the root files, ask yourself _why?_, and _does it really need to be a value?_

### Value structure

Macropod
  - colors.mcss
  - fonts.mcss

Bugherd
  - colors.mcss
  - fonts.mcss

### Importing values

There's two ways to reference values using `postcss-modules-values`. Preffered is:

```css
@value primary, secondary from "./colors.mcss"

.element {
  background: primary;
}
```

This allows you to easily see what values are used from what file in one place (top of the document) without having to trawl through all of the declarations.

### Color Values

Color naming conventions are hard - we've tried a few. Basically the aim is to **have the smallest amount of color values** used across the applications as possible. If you've only got 10 colors, it's far easier to keep track of them. **Colors should be `rgb()`**. Conventions:

  - `@value primary` - primary brand
  - `@value secondary` - secondary brand
  - `@value primaryText` - primary text (think `p`)
  - `@value secondaryText` - secondary text (think alternate `p`, introduction or abstract)
  - `@value link` - `anchor` text. `:hover` should be an `opacity` change and not _another color_.

If you've got a subset of colors, like grays, and you need to include a few. Think about the implementation, you shouldn't need more than three versions of a color:

  - `@value primaryGray`
  - `@value secondaryGray`
  - `@value tertiaryGray`

## Themes

Themes sit independent of any application (dependency and structure wise) - they exist to be an abstracted collection of styles that can be shared across multiple applications. Just like other stylesheets, themes reference values; if you've styled a h1 to be the primary brand color, you want a single entry point to change that color theme and application wide.

themes/
  - macropod.mcss
  - bugherdWebsite.mcss
  - bugherdApplication.mcss

Even though the Bugherd (section of the) website and the Bugherd applications (Board, Feedback) share some of the same styles, they are largely considered different themes. A `h1` on the website is considered an extension (or duplicate) of the `macropod.mcss` `h1`, rather than of the application `h1`.

The relationship between themes looks like this:

```
values/macropod/colors.mcss                        values/bugherd/colors.mcss
            +                                                  +
            |                                  +---------------+-----------------------+
            v                                  v                                       v
   themes/macropod.mcss +--------> themes/bugherdWebsite.mcss          themes/bugherdApplication.mcss
```

i.e. the Bugherd website theme can rely on the macropod theme, but the Bugherd application theme should be independant of any website theme.

### Using Themes

Themes are accessed by `context` which is passed invisibly (magically, probably not) from the initiator all the way down the tree without you having to explicitly pass it. This is _generally not a good idea_ (data flow is less visible), but theming is considered a good case for `context`. Read more about it [here](https://facebook.github.io/react/docs/context.html).

##### Parent

The parent component (think the `Layout` that's used as the `handler` in the _parent_ route):

```js
import theme from './theme/macropod.mcss';

export default class Layout extends React.Component {
  static childContextTypes = {
    theme: React.PropTypes.object
  }

  getChildContext() {
    return {
      theme: theme
    };
  }

  render() {
    return (
      <RouteHandler />
    );
  }
}
```

##### Any Child

Any child of the Parent (no matter if direct or great-great-n-child) will then have access to `this.context.theme` to grab a className:

```js
export default class Layout extends React.Component {
  static contextTypes = {
    theme: React.PropTypes.object
  };

  render() {
    const theme = this.context.theme;

    return (
      <h1 className={theme.h1} />
    );
  }
}
```

- The `context` is _always_ called `theme` - never `macropodTheme` or `bugherdTheme`. This lets us use components under whatever theme.
- It's always good to assign `this.context.theme` to a `const`, rather than going `className={this.context.theme.h1}`

#### Combining themes

**Theme is _mostly_ different from base**

If your theme and your base theme are completely different apart from one or two styles - have those styles _compose_ from the base:

```css
/* theme.mcss */
h1 {
  composes: h1 from './base.mcss';
}
```

**Theme is _slightly_ different from base**

If your theme is only slightly different (think a special 'christmas' theme, where you're changing a few colors) you can use _both_ themes:

```js
import theme from './theme/macropod.mcss';
import christmas from './theme/macropod-christmas.mcss';

export default class Layout extends React.Component {
  getChildContext() {
    return {
      theme: Object.assign(theme, christmas),
    };
  }
}
```






