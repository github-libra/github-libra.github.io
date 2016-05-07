---
title: CSS stacking order

---


## Stacking without `z-index`

When no element has a z-index, elements are stacked in **default** order (from bottom to top):

1. Background and borders of the root element
2. Descendant blocks in the normal flow, in order of appearance (in HTML)
3. Descendant positioned elements, in order of appearance (in HTML)

Consider the HTML structure below:

```html
<div id="absdiv1">
  <br /><span class="bold">DIV #1</span>
  <br />position: absolute; </div>
<div id="reldiv1">
  <br /><span class="bold">DIV #2</span>
  <br />position: relative; </div>
<div id="reldiv2">
  <br /><span class="bold">DIV #3</span>
  <br />position: relative; </div>
<div id="absdiv2">
  <br /><span class="bold">DIV #4</span>
  <br />position: absolute; </div>
<div id="normdiv">
  <br /><span class="bold">DIV #5</span>
  <br />no positioning </div>
```

It resolves to the stacking order like this:

![stacking without z-index](/images/stacking-without-zindex.png)

1. Root element(HTML) backgrounds first (white).
2. DIV #5 next because it's unpositioned.
3. DIV #1, #2, #3, #4 covers one another because of their appearance order.


## Stacking with `z-index`


`z-index` value is an integer(negative or positive), it only has effects when element is *positioned*.

Consider the layers below:

- bottom: furthest from the observer
- ...
- Layer -3
- Layer -2
- Layer -1
- Layer 0 **default** rendering layer
- Layer 1
- Layer 2
- Layer 3
- ...
- top: closest to the observer

note that:

1. when no `z-index` is specified, elements are rendered on the default rendering layer 0.

2. If several elements share the same z-index value (i.e. they are placed on the same layer), stacking rules explained in the section Stacking without z-index apply.

## Stacking Context

> Stacking context is the three-dimensional conceptualization of HTML elements along an imaginary z-axis relative to the user who is assumed to be facing the viewport or the webpage. HTML elements occupy this space in priority order based on element attributes.

A stacking context is formed, anywhere in the document, by any element which is either

- the root element (HTML),
- positioned (absolutely or relatively) with a z-index value other than "auto",
- a flex item with a z-index value other than "auto",that is the parent element display: flex,inline-flex,
- elements with an opacity value less than 1,
- elements with a transform value other than "none",
- elements with a mix-blend-mode value other than "normal",
- elements with a filter value other than "none",
- elements with a perspective value other than "none",
- elements with isolation set to "isolate",
- position: fixed
- specifying any attribute above in will-change even if you don't specify values for these attributes directly
- elements with -webkit-overflow-scrolling set to "touch"

Within a stacking context, child elements are stacked according to the same rules previously explained. Importantly, the z-index values of its child stacking contexts only have meaning inside this parent.

## Examples

### example 1-1

![stacking-order-ex1](/images/stacking-order-ex1-1.png)

According to the stacking context creation rule, there's only one stacking context for example 1-1. So the *default* rule apply. DIV #3 comes behind DIV #1 so #3 is on top of #1 and its children.

### example 1-2

![stacking-order-ex1](/images/stacking-order-ex1-2.png)

Since DIV #2 has a z-index with position, it creates a new stacking context. So the stacking context tree looks like this:

- root
    - DIV #2

all other DIVs kinda belongs to root stacking context while DIV #2 in its own stacking context. So DIV #2 is on top of other DIVs.

### example 1-3

![stacking-order-ex1](/images/stacking-order-ex1-3.png)

DIV #4 also creates a new stacking context with z-index. The stacking context tree looks like this:

- root
    * DIV #2
    * DIV #4

For #2 and #4, they are inside same stacking context: root. Since #4's z-index is larger than #2's, #4 is on the top, then #2, then #4, #3, #1.

### example 2

![stacking-order-ex2](/images/stacking-order-ex2.png)

DIV #2 and DIV #3 both create a stacking context and #2's `z-index` is larger than #3's `z-index`, so #2 is on top of #3 and all its children even #4 has `z-index` value 10.

### example 3

![stacking-order-ex3](/images/stacking-order-ex3.png)

Level1 doesn't create stacking context, level2 need to be on top of level1 so it need `z-index` 1 to create new stacking context. If every item in level2 creates a new stacking context, then level3 will overlap with levels items because level3 items are children of one level2 item, which is under other level2 items because appearance order.

To fix this, no need to create stacking order for all items in each level, create stacking contexts for each level container. For example level2 container has `z-index` value 1, level3 container has `z-index` value 1(in fact, both container has `z-index` value will do). This is because level2 items belong to level2 container stacking context, level3 items belong to level3 container stacking context which is child stacking context of level2's, so it's on top of level3.




