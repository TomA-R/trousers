<p align="center">
  <img width="300" height="300" src="assets/trousers-logo.png" alt="trousers, a little library for CSS-in-JS, without the mess">
</p>

# Trousers 👖
Give React Components some style with Trousers!

[Try it here](https://danieldelcore.github.io/trousers/)

Think of Trousers like `styled-components` + `classnames` + `BEM`, wrapped in a lovely React Hook API <3. Trousers is designed to help you co-locate CSS and JS but opinionated in that it helps you avoid using JavaScript where CSS can take over. It loosely follows a BEM-like philosiphy, borrowing the concept of Blocks (the component), Elements (the child node you want to apply styles to) and Modifiers (apply styles when your component has particular props or state) to reduce the complexity that normally comes with CSS-in-JS.

## Get started 🏗

**Installation**

`npm install --save trousers` or `yarn add trousers`

**Basic Example**

Creating a trousered component

`app/components/button.jsx`

```jsx
import { trousers, useTrousers } from 'trousers';

const styles = trousers()
    .element`
        background-color: ${theme => theme.backgroundColor};
        border: none;
        color: ${theme => theme.textColor};
        margin: 0 10px;
        padding: 10px 20px 14px 20px;

        :hover {
            background-color: ${theme => theme.hoverColor};
            color: rgba(255, 255, 255, 0.8);
        }
    `
    .modifier(props => !!props.primary)`
        background-color: #f95b5b;
        color: #ffffff;

        :hover {
            background-color: #e45454;
        }
    `;

const spanStyles = trousers()
    .element`
        font-size: 20px;
        font-style: italic;
    `;

const Button = props => {
    const buttonClassNames = useTrousers('button', props, styles);
    const spanClassNames = useTrousers('button-span', props, spanStyles);

    return (
        <button className={buttonClassNames}>
            <span className={spanClassNames}>
                {props.children}
            </span>
        </button>
    );
};

export default Button;
```

`app/MyApp.jsx`

```jsx
import { ThemeProvider } from 'trousers';

import Button from './components/button';

const theme = {
    backgroundColor: 'blue',
    textColor: 'white',
    hoverColor: 'lightblue',
};

const MyApp = props => {
    return (
        <ThemeProvider theme={}>
            <Button primary>
                How do I look?
            </Button>
        </ThemeProvider>
    );
};

export default Button;
```

## Motivation
Unlike some of the more popular (and great!) CSS-in-JS libraries, Trousers has made the concious decision to avoid letting you directly apply Props to your CSS properties like this:

```jsx
const Button = styled.button`
  /* Adapt the colors based on primary prop */
  background: ${props => props.primary ? "palevioletred" : "white"};
  color: ${props => props.primary ? "white" : "palevioletred"};

  font-size: 1em;
  margin: 1em;
  padding: 0.25em 1em;
  border: 2px solid palevioletred;
  border-radius: 3px;
`;
```

It's quite hard to see at a glance which state will trigger which styles. The logic used to evaluate the CSS property is JavaScript, which means it _probably should_ be tested in some meaningful way. The logic for a particular state can also tend to be duplicated across mutlitple properties. This is a simple example, consider the same example with multiple states like disabled, loading etc.

Trousers, instead encourages you to group properties for different states. It leverages the C (cascade) in CSS to determine which styles are applied to an element when a particular state is active.

```jsx
const buttonStyles = trousers()
    // Base styles, these are static and every modifier will be applied on top
    .element`
        background-color: blue;
    `
    // A modifier for primary buttons. Note that the `cascade` will handle the color
    .modifier(props => props.primary)`
        background-color: red;
    `
    // Second modifier that will override the prior background-color rules
    .modifier(props => props.disabled)`
        background-color: grey;
    `;
```

Notice that you can localise the logic for a particular state in one place, which makes it more obvious to see which conditions will need to be met before a particular style set is applied.

Under the hood, Trousers will generate a [hash](https://github.com/perezd/node-murmurhash), mount styles to the `<head>` of the page and return a human readable class name. Then on, we are simply dealing with class names.

### Enter Hooks
Hooks is a hot new feature in React, which allows Trousers to access context and state while abstracting the messy details away from the consumer.
Our `useTrousers` hook accepts a name, some props and an instance of `trousers()`. It will then evaluate everything for you and return a human readable class name, which you can then apply to your desired element.
For example, here we define a style for the button and inner span and apply the resulting classes to their respective elements.

```jsx
const Button = props => {
    const buttonClassNames = useTrousers('button', props, buttonStyles);
    const spanClassNames = useTrousers('button-span', props, spanStyles);

    return (
        <button className={buttonClassNames}>
            <span className={spanClassNames}>
                {props.children}
            </span>
        </button>
    );
};
```

### Theme Support
Theming is achieved via React's Context API, which provides a lot of flexibility. You can even choose to nest themes and present a section of your app in a different way.
It looks a little something like this:

```jsx
import { ThemeProvider } from 'trousers';

const lightTheme = {
    primaryColor: 'white',
    secondaryColor: 'blue',
    disabledColor: 'grey',
};

const darkTheme = {
    primaryColor: 'black',
    secondaryColor: 'purple',
    disabledColor: 'grey',
};

const MyApp = () => {
    return (
        <ThemeProvider theme={lightTheme}>
            <h1>Hello World</h1>
            <p>Rest of my app lives here and has access to the light theme!</p>
            <ThemeProvider theme={darkTheme}>
                <p>This subtree will have access to the dark theme!</p>
            </ThemeProvider>
        </ThemeProvider>
    );
};
```

When a Trousers component is mounted within a new theme, it will render new styles for that component and apply them to the component.

You can define how your component handles themes like this:

```jsx
const buttonStyles = trousers()
    .element`
        background-color: ${theme => theme.secondaryColor};
    `
    .modifier(props => props.primary)`
        background-color: ${theme => theme.primaryColor};
    `
    .modifier(props => props.disabled)`
        background-color: ${theme => theme.disabledColor};
    `;
```

Now your component will render different styles based on the context it is mounted in.

## API

### `trousers()`

The `trousers()` function is designed to collect style definitions and provide some portability. If you deside to define CSS in another file, you can do and reimport it into your component.

- `trousers().element`: Because the component is the block. And it is made up of elements
- `trousers().modifier`: Elements can be modified by props. This allows you to extend the look of the element, it also helps you seperate your JS logic from your CSS

### `useTrousers`

### `<ThemeProvider />`

Accepts a `theme` and provides it to children components via context

**Example:**

```jsx
import { ThemeProvider } from 'trousers';

const theme = {
    primaryColor: 'red',
    secondaryColor: 'blue',
};

const App = () => (
    <ThemeProvider theme={theme}>
        {* Every child node will have access to the theme *}
    </ThemeProvider>
);
```

## Unit Testing

- Simplify testing. Classnames can be attached to dom and snapshot tested
- Trousers uses hashes, so the outputted classnames can be snapshot tested

## TypeScript
`// TODO: ...`

## FAQs

**Why BEM?**

**Why not styled components?**

**Runtime? Can't we do all of this at compiletime?**

**My modifier overrides another modifier, why?**

## TODO:
- [ ] `attachGlobalStyle` function
- [ ] Server Side Rendering support
- [ ] Pass `state` into predicates
- [ ] CSS syntax highlighting for VSCode + Atom
- [ ] Unit tests
- [ ] Pass theme to trousers via a generic
- [ ] Fallback mechanism for components without context?

## Resources
- [Creating a TypeScript library with minimal setup](https://michalzalecki.com/creating-typescript-library-with-a-minimal-setup/)
- [How styled-components works](https://medium.com/styled-components/how-styled-components-works-618a69970421)

## Tools
- [light – weight css preprocessor ](https://github.com/thysultan/stylis.js)
- [Optimized JavaScript implementation of the MurmurHash algorithms](https://github.com/perezd/node-murmurhash)