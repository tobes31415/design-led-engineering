# Assumptions

Technologies - These guidelines are assuming You're using React with Typescript and Scss, and that you are using Dependency Injection to load services. The overall design recommendation still applies even in other frameworks however you may need to make adjustments for your specific project needs.

Minimal Local Interactivity - These guidelines are assuming that most of the pages of your app are not interdependent, or at the very least that each page tends to focus on one aspect of the application at a time, with a shared source of truth (which may be local or remote). If that is not the case, such as a Dashboard application that displays multiple unrelated things simultaneously, and reflects connected updates continuously, then the section on services may not be the best choice for your project.

Sharing with the team - As part of the project setup the devs bootstrapping the project should fork this document, put it in the repo under /docs (or similar) and make changes as needed for the specific projects needs.

# CSS Specificity

Whenever you need to write CSS always make sure to use the lowest specificity possible to achieve your purpose, with the exception that all of your css should be scoped at a minimum to a component. [Check out this CSS Specificity Calculator](https://polypane.app/css-specificity-calculator/). As a general rule of thumb you can count it like this, and add the numbers together to get a quick estimate:

| Selector | Score |
| -------- | ----- |
| Tag      | 1     |
| Class    | 10    |
| id       | 100   |

# State management - Services

Business state is managed using the service pattern similar to Angular. You can use State machines such as redux at the scale of an individual component, but application scale state such as session state and cached api calls are maintained via singleton services. You can use hooks to facilitate service level integration as well as function decorators to cache the output of async functions in order for react to bind correctly to the results.

# Component Categories - Business, Layout, Controls, Theme

All components fall into one specific category and each category has different restrictions.
| Category | Business Logic | Dom Logic | Styles (Internal)<sup>1</sup> | Styles (external)<sup>2</sup> |
| -------- | -------------- | --------- | ------ | ---|
| Business | ✔️ | ❌ | ❌ | ❌ |
| Controls | ❌ | ✔️ | ✔️ | ❌ |
| Theme | ❌ | ❌ | ✔️ | ❌ |
| Layout | ❌ | ❌ | ❌ | ✔️ |

<sup>1</sup>Internal Styles
: All internal css properties such as display, padding, color, as well as minimal external properties like borders, border-radius, outlines. margins of nested elements are allowed, but external margins are not allowed. Make sure to use the absolute minimum css specificity possible when defining any external properties as they will likely need to be overridden.

<sup>2</sup>External Styles
: Affects the layout container and edges of controls only: margins, grid props, flexbox props, border, shadows, outline, border-radius etc. Should not be used to recolor or change the internal rendering of any controls, use a Theme<sup>**New**</sup> instead to achieve that effect.

These restrictions allow for rapid development and design iteration by isolating design elements and logic. App wide use of this pattern also results in a highly consistent UI and reduces the chance for bugs by minimizing the amount of custom logic and binding code required on each screen

### Business

These components typically combine layout, themes, and controls together with the needed service calls to fetch and update data, and to perform actions to achieve the appearance and behaviour indicated by the wireframes. Business components MUST NOT define any css at all whatsoever. If you find yourself needing to define css on a business component then look to see if you are missing a layout or theme. Keep in mind that this is usually an indication that there is an error in the wireframes. Before you go implement new layouts and themes make sure to verify with the designer if this is intentional or if there was a typo in the wireframes. Designers are humans too and make mistakes just as easily as we do, especially on projects with high UI churn.

### Controls

These are the widgets and doodads that the user clicks on, taps, displays data, etc. They MUST NOT have any business logic, or external styles. Consider that controls may be used on any page at any time. They should be 100% agnostic of where they are being used so that they can be used easily anywhere.

### Theme

Theme wrappers allow you to declare the visual intent of a design without having to explicitly pass render specific properties down into all the controls. There are multiple ways of implementing themes and it is important to decide on a case by case basis what will produce the better result for your needs. See the section below for more details. These MUST NOT have any business logic, or external styles. An example of a theme would be DarkMode vs LightMode which you could use to invert your colors.

### Layout

Layout wrappers allow you to declare the visual layout of a design without having to have a bazillion div's everywhere in your code. Layout wrappers MUST NOT have any business logic, and must not have any DOM logic outside what is minimally necessary to control the layout (such as detecting screen size and orientation changes)

## Component Properties

- Components MAY have data properties, event properties, and may render child nodes.
- Components MUST NOT have properties that change their semantics. Fork the wrapper instead
  ex:

```html
Don't use
<button variant="primary">...</button>

use this instead
<PrimaryButton>...</PrimaryButton>
```

- Components MUST NOT have properties that change the way they render. Use Theme context instead
  ex:

```html
Don't use
<Spinner isLightColored="{true}" />

use this instead (New concept)
<DarkBackgroundTheme>
  <Spinner />
</DarkBackgroundTheme>
```

The reason we do this is so that there is a 1:1 mapping between the UX designers INTENT and the actual code that gets written. Anyone joining the project can read the code and right away understand the goal of the code, making it easier to implement new features and find bugs.

Please be fully 100% aware that following these restrictions does place a small burden on the developers and introduces a small amount of extra wrappers that could all be removed just by allowing one or two properties. _Resist the urge to add that property_. The pain you feel is INTENTIONAL and is CRITICAL to encouraging discussion with the design team. The balancing point should be such that it is not overly bothersome to support a few variants of common controls but that if the design team is going off the deep end and making customizations on every single page that the developers will scream loudly and push back on the design. A good design should feel consistent from screen to screen. The UX designers know this and they will use variations to intentionally draw the users intentions when it is advantageous. There is a maintenance cost to this and forcing these restrictions surfaces those costs early in the process instead of further down the road. This allows the dev team to make intentional choices about what variations to accept and support, and what to push back on.

To make things easier on the dev team keep in mind that you can distinguish between exported components, and components which are not exported. You can add additional properties to the NON exported components in order to make the code simple and to avoid unnecessary duplication. For example in the case of the PrimaryButton, you could have a single file called buttons.tsx that contains a genericButton component that is NOT exported, then export your UX Designed variants with names that match the wireframes or are indicative of their purpose

```tsx
// NOT exported, only accessible inside this one file
function GenericButton(params, variant) { ... }

// exported for use in the app
export function PrimarButton(params) { return <GenericButton ...params variant="primary"/> }
export function SecondaryButton(params) { return <GenericButton ...params variant="secondary"/> }
```

### Page/Screen specific components

It is common for some more complex screens to have components which do not ever get reused on other parts of the application. These components should still be treated as separate from the Page itself and follow the same design restrictions as other components, however you can put them in a subfolder of the page they're used on in order to prevent creating a lot of noise in the shared components folder. Enforcing the page design to be done modularly like this makes it easier to refactor sections of the design as well as to move components to the shared folder if they do end up being reused on other pages/screens in the future.

## Exceptions to the above

Keep in mind that some exceptions will need to apply where the effort to maintain the design pattern is too high relative to the gain. For example Typography Controls can have over a dozen different variants and new ones keep getting added as projects grow in complexity. The Time and complexity of maintaining that many wrappers outweighs the benefit that the cleaner code would provide so instead in that case it's better to expose a "variant" property on a generic Typography Component instead of making wrappers for each style. Typography is probably a safe place to start, but you should discuss with your team if you feel you need to make other exceptions based on the project's needs.

## Implementing Themes

Implementation of a theme should either use pure css fully contained under the Themes root selector, or should make use of React's useContext feature to allow specific components to opt-in to theme awareness. Regardless of which method is used to implement a theme, the developers consuming it should not need to know the details, it should just work as expected. Care must be taken to ensure nesting of themes will not cause conflicts in cases where two themes ex Lightmode vs DarkMode would overwrite the same properties. Similarly care should be taken to avoid over-using useContext for cases where we are not expecting conflicts, to avoid scope creep in the controls. Implementing themes will be a balance between supporting the visual design and keeping the code maintainable.

| **Pure CSS**                                                                 |                                                                                                                                                               |
| ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Pros                                                                         | Cons                                                                                                                                                          |
| <ul><li>Easier to write</li><li>Self contained</li></ul>                     | <ul><li>Fragile</li><li>Nested Themes will conflict with each other</li></ul>                                                                                 |
| **useContext**                                                               |                                                                                                                                                               |
| Pros                                                                         | Cons                                                                                                                                                          |
| <ul><li>Much more targeted control</li><li>Nested themes supported</li></ul> | <ul><li>Requires the use of Tier2 Specificity (classes)</li><li>Controls will slowly get larger and harder to maintain as more themes are supported</li></ul> |

## Libraries To support this Design

Here is a selection of libraries which can be used to meet the support needs of most projects. The libraries in this list are maintained by Jake Tober. Feel free to send requests or contributions.

| Name                 | Description                                                                                                                                                                                                                                                             | Link                                               |
| -------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------- |
| dependency-injection | A tiny library to handle basic dependency injection                                                                                                                                                                                                                     | https://github.com/tobes31415/dependency-injection |
| dispose              | Register callbacks on objects so that they get properly disposed at a later time even if you have complex trees of objects                                                                                                                                              | https://github.com/tobes31415/dispose              |
| storage              | Common wrappers for local web storage mechanisms that supports namespacing, serialization and encryption                                                                                                                                                                | https://github.com/tobes31415/storage              |
| console-logger       | Debugging wrapper for console.log that adds namespacing, color formatting, and simultaneously forks log events to a 3rd party log utility of your choice                                                                                                                | https://github.com/tobes31415/console-logger       |
| basic-observables    | Follows the interface for RXJS but is a tiny fraction of the size. Good for smaller projects or libraries to avoid creating large dependency trees. Compatible syntax means it's easy to migrate to the full RXJS library if you end up needing a more powerful library | https://github.com/tobes31415/basic-observables    |

## Copyright

<p xmlns:cc="http://creativecommons.org/ns#" xmlns:dct="http://purl.org/dc/terms/"><a property="dct:title" rel="cc:attributionURL" href="https://github.com/tobes31415/design-led-engineering">Design Led Engineering - Design Guidelines</a> © 2021 by <a rel="cc:attributionURL dct:creator" property="cc:attributionName" href="https://www.macadamian.com/">Macadamian Technologies</a> is licensed under <a href="https://creativecommons.org/licenses/by/4.0/?ref=chooser-v1" target="_blank" rel="license noopener noreferrer" style="display:inline-block;">CC BY 4.0<img style="height:22px!important;margin-left:3px;vertical-align:text-bottom;" src="https://mirrors.creativecommons.org/presskit/icons/cc.svg?ref=chooser-v1"><img style="height:22px!important;margin-left:3px;vertical-align:text-bottom;" src="https://mirrors.creativecommons.org/presskit/icons/by.svg?ref=chooser-v1"></a>
To view a copy of this license, visit <a href="https://creativecommons.org/licenses/by/4.0/?ref=chooser-v1" target="_blank" rel="license noopener noreferrer" style="display:inline-block;">https://creativecommons.org/licenses/by/4.0</a>
</p>
