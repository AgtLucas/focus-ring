- [Rationale](#rationale)
- [API Proposal](#api-proposal)
  * [Note: Styling non-native elements which should always match `focus-ring`](#note--styling-non-native-elements-which-should-always-match--focus-ring-)
- [Example heuristic](#example-heuristic)
- [Implementation Prototype](#implementation-prototype)

## Rationale

The status quo, `:focus`, is quite problematic:

- Many developers disable the default focus ring in their CSS styles,
  others attempt to style it in concert with their design.
  The former often seems to be a result of finding the default focus ring
  both aesthetically unpleasant and confusing to users
  when applied after a mouse or touch event and introduces accessibility problems.
  The latter inevitably creates considerably more of the kind of problem that the former was trying to solve.
- Some native elements in some browsers,
  notably `<button>` in Chrome,
  have a "magic" focus style which does _not_ apply
  unless focus was received via a keyboard interaction.

To deal with this:
- It seems evident that a visual indication of what has focus
  is only interesting to a user who is using the keyboard
  to interact with the page.
  A user using any kind of pointing device
  would only be interested in what is in focus
  if they were _just about_ to use the keyboard -
  otherwise, it is irrelevant and potentially confusing.
- Thus, if we only show the focus ring when relevant,
  we can avoid user confusion
  and avoid creating incentives for developers to disable it.
- A mechanism for exposing focus ring styles
  only when the keyboard is the user's current input modality
  gives us this opportunity.

## API Proposal

```css
/* override UA stylesheet if necessary */
:focus {
  outline: 0;
}

/* establish desired focus ring appearance for appropriate input modalities */
:focus-ring {
  outline: 2px solid blue;
}
```

`:focus-ring` matches native elements that are
1. focussed; and
2. would display a focus ring if only UA styles applied

Additionally, `:focus-ring` matches non-native elements as if they were
native button elements.

### Note: Styling non-native elements which should always match `focus-ring`


This is not currently part of the spec,
but a mechanism is needed to explain the ability of native text fields
to match `:focus-ring` regardless of how focus arrived on the element.

Consider an author creating a custom element, `custom-texty-element`,
which they believe should show a focus ring on mouse click.
By default, the default `:focus-ring` user agent style
will not show a focus ring when this element receives focus via mouse click.
However, if the author were to style the element via `:focus`,
they could not recreate the browser's default `outline` style reliably:

```css
custom-texty-element:focus {
   outline: ???;
}
```

Either of the following two new primitives would allow the author to
show the default focus ring on click for this element:

1. Add a new keyword value to the outline shorthand that represents whatever the default UA `::focus-ring` is. Then authors can do:

        custom-texty-element:focus {
            outline: platform-default-focus-outline-style-foo;
        }

2. Add a new CSS property that controls "keyboard modality" vs non-"keyboard modality" behavior, e.g.

        custom-texty-element {
            show-focus-ring-foo: always | keyboard-only;
        }

_("`-foo`" placeholder indicates that these names are by no means final!)_

While either of these primitives would suffice,
having both would provide more flexibility for authors.

## Example heuristic

The heuristic used to decide the current modality should not be defined
normatively. An example heuristic is to update modality on each style recalc:
if the most recent user interaction was via the keyboard;
and the key pressed was either `Tab`, `Shift + Tab`, or an arrow key;
then the modality is keyboard. Otherwise,
the modality is not keyboard.

## Implementation Prototype

The tiny
[focus-ring.js](http://wicg.github.io/focus-ring/src/focus-ring.js)
provides a prototype intended to achieve the goals we are proposing
with technology that exists today
in order for developers to be able to try it out, understand it and provide feedback.
It sets a `.js-focus-ring` class on the body element
to provide a way to disable focus styles only when the polyfill is loaded.
It also sets a `.focus-ring` class on the active element
if the script determines that the keyboard is being used.
This attribute is removed on any `blur` event.

This allows authors to write rules
which show a focus style only when it would be relevant to the user.
Note that the prototype does not match the proposed API -
it is intended to give developers a feel for the model
rather than to provide a high-fidelity polyfill.
