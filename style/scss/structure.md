# SCSS file and directory structure
SCSS files should be organized consistently across the entire Zetkin project.
The correct way varies slightly depending on accompanying technologies.

## Mixins, variables and other reusables
SCSS content that is reused across many different parts of an application must
be located in SCSS files in the `src/scss` folder of the repository. These must
not result in any styling in themselves, i.e. they may not contain selectors.

The structure of `src/scss` may look like this:

* `src/scss/`
  * `_mixins.scss`
  * `_vars.scss`

## Responsive breakpoints
Media queries for responsive design must be implemented as mixins so that they
can be reused across the project. The pixel width breakpoints must be defined
in variables so that they can be changed globally.

The mixed in media queries should appear in context with the selectors they
affect, so that styling for all widths for a particular selector can be found
in the same place. More about this in the SCSS style guide.

## Using SCSS with React
When SCSS is used alongside React.js to style HTML rendered by React components
SCSS files will be located in two different locations depending on their scope
and purpose.

### Styling React components
SCSS styles for React components should be per-component, located in SCSS files
next to their respective component JSX file, and named to match. For example,
a `PersonList` component may compose a `PersonListItem` component and exist in
a folder callde `personlist`. In this case, the SCSS source code for the two
components should be in the same folder, in files called `PersonList.scss` and
`PersonListItem.scss`, like so:

* `src/components/personlist/`
  - `PersonList.jsx` (component declaration)
  - `PersonList.scss` (component styling)
  - `PersonListItem.jsx`
  - `PersonListItem.scss`

The React components are responsible for assigning a `className` to the
top-level HTML element that matches the component name, i.e. for `PersonList` a
valid CSS selector will be `.PersonList`.

### Class names for sub-elements
Most components will have more or less complex HTML content with regular HTML
sub-elements below the top-level element. Such elements must have a class name
consisting of the component name, followed by a dash and a descriptive element
name in camelCase, e.g. `MyComponent-submitButton`.

### Structure of a component SCSS file
The content of a SCSS file for a React component should only affect the named
component, or other components appearing within the context of the named
component.

In other words, styling defined in `MyComponent.scss` is not allowed to affect
the style or layout of `OtherComponent`, except in cases where `OtherComponent`
is composed within `MyComponent`.

```scss
.MyComponent {
  // Base styling for MyComponent
}

.MyComponent-submitButton {
  // Base styling for some element found specifically in MyComponent:
  // <input type="submit" class="MyComponent-submitButton">
}

.MyComponent .OtherComponent {
  // Base styling for OtherComponent, when it appears within MyComponent
}

@include respond-to(medium) {
  .MyComponent {
    // Responsive styling for MyComeponent
  }

  .MyComponent .OtherComponent {
    // Responsive styling for OtherComponent instances within MyComponent
  }
}
```
