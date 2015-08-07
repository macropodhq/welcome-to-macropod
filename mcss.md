# `.mcss`

`mcss` is Macropods stylesheet extension. It stands for `modular-cascading-style-sheet`; where locally scoped modules are enabled by default - with some cool other features. The main driving force behind `mcss` is [css-modules](https://github.com/css-modules/css-modules).

Take a read of `css-modules` to understand how standard modularity works. We've enabled [local scoping](https://github.com/webpack/css-loader#local-scope) through `css-loader`, so none of our classes will bleed.

- [Formatting](#formatting)
  - [Naming conventions](#naming-conventions)
  - [Importing the class object](#importing-the-class-object)
  - [Composing](#composing)
  - [Styling elements like `p`](#element-styling)
- [Structuring re-usable components](#structuring-re-usable-components)
  - [Components with conditional classes](#components-with-conditional-classes)
- [Constants (variables)](#constants)

----

## Formatting

Because all of our classes are local, we can use simplistic class names and not have to worry about other components having the same class.

### Naming conventions

Because the stylesheets are component based and explicit, there's no need to use a convention like SUIT. The class names are then referenced in Javascript, so to avoid an ugly `styles['link--blue']`, we use camel case:

*`component.mcss`*
```css
.linkBlue {
  color: blue;
}
```

*`component.js`*
```js
import styles from './component.mcss';
<a href="#" className={styles.linkBlue} />
```

#### Note:

If you're referring to a component (`<Avatar />`) begin the class with a capital (`.Avatar`). If you're referring to an element (`<img alt="avatar" />`) begin the class with a lower case letter (`.avatar`).

### Importing the class object

The default object is referred to as `styles` (plural). There are cases where you can rename this - but that's for higher level styling (think themes). Component based objects are always referred to as `styles`. Even though our pipeline will automatically resolve the `.mcss` extension - it's best practice to include this as we resolve the `js` extension too:

*`component.js`*
```js
import styles from './component.mcss';
```

### Composing

[Composing](https://github.com/webpack/css-loader#composing-css-classes) is powerful, and it's how two classes share the same style with different names. It's similar to `.scss`'s `@extend`. If you have a modified state of a component, compose the original state within the modifier:

*`component.mcss`*
```css
.link {
  color: red;
  font-weight: bold;
  text-decoration: none;
}

.linkBlue {
  composes: link;
  color: blue;
}
```

### Element styling

There's a drawback with being so explicit about class names - and that's being explicit about class names that re-occur often. Take for example, a bunch of paragraphs:

*`view.mcss`*
```css
.paragraph {
  margin: 2em 0;
}
```

*`view.js`*
```js
import styles from './view.mcss';

render() {
  return (
    <div>
      <p className={styles.paragraph}>Lorem</p>
      <p className={styles.paragraph}>Lorem</p>
      <p className={styles.paragraph}>Lorem</p>
      <p className={styles.paragraph}>Lorem</p>
    </div>
  )
}
```

You can see that the `DOM` is getting polluted incredibly fast - it's making it quite difficult to read. On a page with more paragraphs (think 20!), passing the `className` prop to every `p` gets tedious. We can't just style `p {}` because that will be a global style - so we need to restrict it:

*`view.mcss`*
```css
.container :global(p) {
  margin: 2em 0;
}
```

*`view.js`*
```js
import styles from './view.mcss';

render() {
  return (
    <div className={styles.container}>
      <p>Lorem</p>
      <p>Lorem</p>
      <p>Lorem</p>
      <p>Lorem</p>
    </div>
  )
}
```

While it is harder to trace from a file point of view - it is easily traced in inspector (`.hashed-name p`).

**Note:** please don't abuse this. If you feel like you're adding too many of the same classes, you probably are. If you're being lazy and just using `:global()` instead of a few cmd+c shortcuts - you're doing it wrong.

## Structuring re-usable components

Our library of reusable components, [macropod-components](https://github.com/macropodhq/macropod-components) come with a default style that may need to be _extended_ when implemented within a product (like Stack). This was traditionally done with a separate stylesheet in Stack, and then using the class name to override styles. This is incredibly hard to trace - and the method can't work with `mcss` because the class names are randomly generated (well, at least the `-[hash]` is). **Never target a element using a compiled class name**.

Let's take, for example, how we'd structure a re-usable `Link` component. This component has a default style, but then gets a custom color when we implement it in Stack:

*`macropod-components/link.mcss`*
```css
.Link {
  color: gray;
  text-decoration: none;
}
```

*`macropod-components/link.js`*
```js
import styles from './link.mcss';

render() {
  return (
    <a href="#" className={this.props.className || styles.Link}>{this.props.children}</a>
  )
}
```

As you can see, the styling for `Link` is controlled by an `or` statement. If the parent has passed a `className` prop, the component will not use the _default_ style (`styles.Link`). **So how do we _over-rule_ a property, but still inherit the default styles from `Link`?** This is handled by the parent, who _owns_ this styling:

*`stack/navigation.mcss`*
```css
.Link {
  composes: Link from 'macropod-components/packages/link/link.mcss';
  color: blue;
}
```

*`stack/navigation.js`*
```js
import Link from 'macropod-components/packages/link';
import styles from './navigation.mcss';

render() {
  return (
    <nav>
      <Link href="/menu" className={styles.Link}>Menu</Link>
    </nav>
  )
}
```

In the above example, `Link` retains the `text-decoration: none;` by composing the default style, but overrules the color by defining the property below the composition.

### Components with conditional classes

What if we have a component that has `states` which can be conditionally met? We need to use [`classnames`](https://github.com/JedWatson/classnames) to control the generated names:

*`button.mcss`*
```css
.Button {
  background: green;
  color: white;
}

.Button:global(.disabled) {
  opacity: 0.7;
  cursor: wait;
}
```

*`button.js`*
```js
import cx from 'classnames';
import styles from './button.mcss';

render() {

  const buttonClass = cx({
    [styles.Button]: !this.props.className,
    'disabled': this.state.processing,
    [this.props.className]: this.props.className,
  })

  return (
    <button className={buttonClass}>Save</button>
  )
}
```

This _looks_ bad, because we're throwing a class into globals with `.Button:global(.disabled)`, meaning that `.disabled` never gets processed to a modular name. It **doesn't _really_ matter**. `disabled` is a state of `.Button`, therefor it is chained, preventing the global `.disabled` from leaking. This could be the output:

```css
.stack---Button---1n_w5.disabled
```

You can see above that `.disabled` can't hurt any other selector, because it's tied to a generated name. The reverse can still happen though, someone _could_ `:global(.disabled)` in another file and not chain it to anything and impact all `.disabled` classes across the application. This could happen with any element though (`:global(div)`).

Because we've defined the class name as 'disabled' in the component, it let's us override in the parent if required:

*`stack/navigation.mcss`*
```css
.Button {
  composes: Button from './button.mcss';
}

.Button:global('disabled') {
  background: gray;
}
```

*`stack/navigation.js`*
```js
import Button from '/button';
import styles from './navigation.mcss';

render() {
  return (
    <nav>
      <Button className={styles.Button}>submit</Button>
    </nav>
  )
}
```

Our `Button`'s disabled state will now have a gray background for this implementation only! You can see one drawback - we need to define `.Button` if we're defining a conditional version of it. Otherwise, because `this.props.className` is defined, it wouldn't apply the default style.

## Constants

Please see [`postcss-local-constants`](https://github.com/macropodhq/postcss-local-constants) for conventions. **This will be updated when we roll out some form of theming across our products. Stay tuned.**