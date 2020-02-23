---
title: Styling Components
description: Styling Components
url: /docs/styling
contributors:
  - jthoms1
  - shreeshbhat
  - danbucholtz
---

# Styling Components

Styling and theming are first class citizens in Stencil with support built directly into the compiler. Components use standard CSS for styling. The CSS file to be used is specified in the [Component Decorator](). Common CSS post processors like Sass or less are supported via [plugins]().

Since components use CSS, they're simple and fun to style. By default in Stencil, CSS works just like it does in the browser or on any website - with global selectors that apply to the entire document. To make components more robust, Stencil offers two methods of "scoping" CSS or encapsulating styles to a specific component. The recommended method of scoping CSS is called Shadow DOM.


## Shadow DOM

Shadow DOM is the recommended approach for styling Stencil components. It is a part of the Web Component specification and provides a great developer experience by enabling developer's to write very simple selectors for their components without worrying about collisions between components.

### What is Shadow DOM

[Shadow DOM](https://developers.google.com/web/fundamentals/web-components/shadowdom) is an API built into the browser that allows for DOM encapsulation and style encapsulation. Shadow DOM shields our component from its surrounding environment. What this means is that the interal implementation of a component remains just that - internal. CSS from the outside world does not cross over or interfere with the Shadow DOM CSS. Consumers of a component are discouraged from 

### Using Shadow DOM

Shadow DOM is not currently turned on by default for web components built with Stencil. To turn on Shadow DOM in a web component built with Stencil, you can use the `shadow` param in the component decorator. Below is an example of this:

```tsx
@Component({
  tag: 'my-component',
  styleUrl: 'my-component.css',
  shadow: true
})
export class MyComponent {

}
```

From there, in `my-component.css`, write CSS as you normally would for a component. Since styles from the outside world do not impact the Shadow DOM, things like fonts, sizings, colors, etc are typically set in the `:host` selector using CSS variables. Here is an example:

```css

:host {
  display: block;
  font-family: var(--fonts);
  box-sizing: border-box;
  contain: content;
}

```

The css above is a typical `:host` selector you'll use. It sets the host element to be `display: block` so it acts like any other div. It passes in typography or font-family information from the `--fonts` CSS variable. It sets the `box-sizing` model to `border-box`. It also sets CSS contaiment to `content` for improved performance.

Let's take our above example a step further.

```tsx
@Component({
  tag: 'my-component',
  styleUrl: 'my-component.css',
  shadow: true
})
export class MyComponent {

  render() {
    return (
      <div class="container">
        My name is <span>Neo</span>
      </div>
    )
  }
}
```

The corresponding css would look like this:

```css

:host {
  display: block;
  font-family: var(--fonts);
  box-sizing: border-box;
  contain: content;
}

.container {
  display: flex;
  justify-content: center;
  align-items: center;
}

span {
  color: red;
}

```

As you can see above, the CSS selectors can be very simple because Shadow DOM keeps other CSS from colliding.

### Using CSS Variables with Shadow DOM

CSS Variables, or [CSS Custom Properties](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_variables), are a part of the CSS 4 spec. CSS Variables enable developers to style internal details of the component from outside of the component. Here's an example

Here's a sample global CSS file included in an app's html page.

```css

--primary-color: red;

```

Looking at the `my-component.css` again, here's how it could be implemented to use CSS variables to be more flexible:

```css

:host {
  display: block;
  font-family: var(--fonts);
  box-sizing: border-box;
  contain: content;
}

.container {
  display: flex;
  justify-content: center;
  align-items: center;
}

span {
  color: var(--primary-color);
}

```

In the above snippet, the `color: red` code was replaced with `color: var(--primary-color)`. This makes the code more flexible and enables any consumer of the component to override the style of the component from outside by simply setting the value of the `--primary-color` variable.

#### CSS Variable Best Practices

Typically, components add a `:host` style selector that is very similar for all components. In addition to initializing things like fonts, `:host` is the perfect place to specify the default values of all of the CSS Variables that your component uses.

```css

:host {
  display: block;
  font-family: var(--fonts);
  box-sizing: border-box;
  contain: content;
  --text-color: var(--primary-color);
  --font-size: 24px;
  ---padding: 10px;
}

.container {
  padding: var(--padding);
}

span {
  color: var(--text-color);
  font-size: var(--font-size);
}

```

In the above example, we set the default values of `--text-color`, `--font-size` and `--pading` variables.

The Stencil team recommends creating a CSS file at `src/global/variables.css` to hold all of your common, top-level variables such as colors `--primary-color`, `--secondary-color`, etc and other commonly used variables (such as `--font` in the above examples). After this file exists, we recommend setting `globalStyle: 'src/global/variables.css'` in the stencil.config.ts` file.

As design systems grow, all branding colors, typography information, margins, borders, etc are exposed as CSS Variables to make it easy to build robust, maintainable components.


### Shadow DOM Considerations

Shadow DOM is designed to keep the private implementation details of your component shielded or hidden from outside consumers. That is the intent of the API. In simpler terms, that means that when attempting to use an element like `querySelector` to query an node inside of the Shadow DOM, it will not work. You would first need to query the component's `shadowRoot` element. In the case of `my-component` above, you'd be able to do something like this.

```js

const myComponentRoot = document.querySelector('my-component').shadowRoot;
const containerElement = myComponentRoot.querySelector('.container');
```

While the above code works, it is generally considered an anti-pattern for one component to know about another component's internal implementation details. If one component needs to know or check if a different component is in a given state, [Events]() are the recommended way of achieving that.

### Fallback to Scoped CSS

While Shadow DOM is available in all modern, evengreen browsers, it is not available without a polyfill in older browsers like Internet Explorer 11. When Stencil compiles your source code, it automatically translates Shadow DOM CSS selectors into CSS that is scoped to a specific component *without* Shadow DOM for older browsers. Note: Stencil will automatically determine whether to load the Shadow DOM bundle, or fallback to the Scoped CSS bundle. As a developer, this is not something you need to worry about because Stencil takes care of it for you automatically.

If you don't wish to use Shadow DOM and would prefer to use scoped CSS, the property `scoped: true` can be set in the Component Decorator, though this is not recommended.
