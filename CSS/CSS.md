# CSS Notes

**This notes are based on _Learning Web Design(4th Edition)_ book by Jennifer Niederst Robbins**

## General

Generally speaking, the closer the style sheet is to the content, the more weight it is given. Embedded style sheets that appear right in the document in the style element have more weight than external style sheets. Inline styles have more weight than embedded style sheets.

External style sheets -> Embedded style sheets -> Inline styles

To prevent a specific rule from being overridden, you can assign it _“importance”_ with the `!important` indicator

Style information can come from various sources, listed here from general to specific. Items lower in the list will override items above them:

-   Browser default settings
-   User style settings (set in a browser as a “reader style sheet”)
-   Linked external style sheet (added with the link element)
-   Imported style sheets (added with the `@import` function)
-   Embedded style sheets (added with the style element)
-   Inline style information (added with the style attribute in an opening tag)
-   Any style rule marked `!important` by the author
-   Any style rule marked `!important` by the reader (user)

## Selectors

-   ID selectors are more specific than (and will override)
-   Class selectors, which are more specific than (and will override)
-   Contextual selectors, which are more specific than (and will override)
-   Individual element selectors

## Pseudo-classes

### Link pseudo-classes

_:link_ Applies a style to unclicked (unvisited) links  
_:visited_ Applies a style to links that have already been clicked

### User action pseudo-classes

_:focus_ Applies when the element is selected and ready for input  
_:hover_ Applies when the mouse pointer is over the element  
_:active_ Applies when the element (such a link or button) is in the process of being clicked or tapped

### More Pseudo-classes

_:root_ | _:only-child_ | _:nth-child()_ | _:empty_ | _:first-of-type_ | _:nth-last-child()_ | _:first-child_ | _:last-of-type_ | _:nth-of-type()_ | _:last-child_ | _:only-of-type_ | _:nth-last-of-type()_

## Font and Text Properties

### font-family

_Values:_ one or more font or generic font family names, separated by commas | inherit  
_Default:_ depends on the browser  
_Applies to:_ all elements  
_Inherits:_ yes

_“Why specify more than one font?”_  
If the first specified font is not found, the browser tries the next one, and down through the list until it finds one that works.

### font-size

_Values:_ length unit  
_Default:_ medium  
_Applies to:_ all elements  
_Inherits:_ yes

The preferred values for font-size in contemporary web design are em measurements and percentage values. For example, the font-size of the h1 ’s parent element ( body ) has been specified as 100% of the default text size (generally 16 pixels). The h1 inherits the 16px size from the body element, and applying the 150% font-size value multiplies that inherited value, resulting in an h1 that is 24 pixels. If the user has her font size set to 30 pixels, for example, to read it on a television browser from across the room, the resulting h1 would be 45 pixels, but would maintain its proportion relative to the main body text, which is the idea of using relative measurements.

_Em best practices_  
As of this writing, the most popular solution for making ems display consistently is to set the size of the body element to 100% (keeping it at the default or user’s preference), then use ems to size the text elements thereafter.

**The shortcut font property**  
_Values:_ font-style font-weight font-variant font-size/line-height font-family| inherit  
_Default:_ depends on default value for each property listed  
_Applies to:_ all elements  
_Inherits:_ yes

At minimum, the font property must include a font-size value and a font -family value, in that order. Omitting one or putting them in the wrong order causes the entire rule to be invalid. This is an example of a minimal font property value:

```
p { font: 1em sans-serif; }
```

### text-indent

_Values:_ length measurement | percentage | inherit  
_Default:_ 0  
_Applies to:_ block-level elements, table cells, and inline blocks  
_Inherits:_ yes

### text-align

_Values:_ left | right | center | justify | inherit  
_Default:_ left for languages that read left to right; right for languages that read right to left  
_Applies to:_ block-level elements, table cells, and inline blocks  
_Inherits:_ yes

### list-style-type

_Values:_ none | disc | circle | square | decimal | decimal-leading-zero | lower-alpha |
upper-alpha | lower-latin | upper-latin | lower-roman | upper-roman | lower-greek | inherit  
_Default:_ disc  
_Applies to:_ ul , ol and li (or elements whose display value is list-item )  
_Inherits:_ yes

### list-style-position

_Values:_ inside | outside | inherit  
_Default:_ outside  
_Applies to:_ ul , ol , and li (or elements whose display value is list-item )  
_Inherits:_ yes

The list-style-position property allows you to pull the bullet inside the content area so it runs into the list content.

### list-style-image

_Values:_ url | none | inherit  
_Default:_ none  
_Applies to:_ ul , ol , and li (or elements whose display value is list-item )  
_Inherits:_ yes

You can also use your own image as a bullet using the list-style-image property.

### CSS Review: Font and Text Properties

| Property        | Description                                                               |
| --------------- | ------------------------------------------------------------------------- |
| font            | A shorthand property that combines font properties                        |
| font-family     | Specifies a typeface or generic font family                               |
| font-size       | The size of the font                                                      |
| font-style      | Specifies italic or oblique fonts                                         |
| font-variant    | Specifies a small-caps font                                               |
| font-weight     | Specifies the boldness of the font                                        |
| letter-spacing  | Inserts space between letters                                             |
| line-height     | The distance between baselines of neighboring text lines                  |
| text-align      | The horizontal alignment of text                                          |
| text-decoration | Underlines, overlines, and lines through                                  |
| text-direction  | Whether the text reads left-to-right or right-to-left                     |
| text-indent     | Amount of indentation of the first line in a block                        |
| text-shadow     | Adds a drop shadow under the text                                         |
| text-transform  | Changes the capitalization of text when it displays                       |
| vertical-align  | Adjusts the vertical position of inline elements relative to the baseline |
| white-space     | How whitespace in the source is displayed                                 |
| word-spacing    | Inserts space between words                                               |

## Colors and Backgrounds

### color

_Values:_ color value (name or numeric) | inherit  
_Default:_ depends on the browser and user’s preferences  
_Applies to:_ all elements  
*Inherits:*yes

### background-color

_Values:_ color value (name or numeric) | transparent | inherit  
_Default:_ transparent  
_Applies to:_ all elements  
_Inherits:_ no

### opacity

_Values:_ number (0 to 1)  
_Default:_ 1  
_Applies to:_ all elements
_Inherits:_ no

### background-image

_Values:_ url (location of image) | none | inherit  
_Default:_ none  
_Applies to:_ all elements  
_Inherits:_ no

The image starts in the top lefthand corner and tiles horizontally and vertically until the entire element is filled. (although you’ll learn how to change that in a moment)

### background-repeat

_Values:_ repeat | repeat-x | repeat-y | no-repeat | inherit  
_Default:_ repeat  
_Applies to:_ all elements  
_Inherits:_ no

### background-position

_Values:_ length measurement | percentage | left | center | right | top | bottom | inherit  
_Default:_ 0% 0% (same as left top )  
_Applies to:_ all elements  
*Inherits:*no

The background-position property specifies the position of the origin image
in the background. You can think of the origin image as the first image that
is placed in the background from which tiling images extend. Here is the
property and its various values.

### background-attachment

_Values:_ scroll | fixed | local (new in CSS3) | inherit  
_Default:_ scroll  
_Applies to:_ all elements  
_Inherits:_ no

**_fixed_**  
The background is fixed relative to the viewport. Even if an element has a scrolling mechanism, the background doesn't move with the element. (This is not compatible with background-clip: text.)  
**_local_**  
The background is fixed relative to the element's contents. If the element has a scrolling mechanism, the background scrolls with the element's contents, and the background painting area and background positioning area are relative to the scrollable area of the element rather than to the border framing them.  
**_scroll_**  
The background is fixed relative to the element itself and does not scroll with its contents. (It is effectively attached to the element's border.)

### The Shorthand background Property

_Values:_ background-color background-image background-repeat background-
attachment background-position | inherit  
_Default:_ see indiviual properties  
_Applies to:_ all elements  
_Inherits:_ no

### Multiple backgrounds

To apply multiple values for background-image , put them in a list separated
by commas.

```
#example1 {
  background-image: url(img_flwr.gif), url(paper.gif);
  background-position: right bottom, left top;
  background-repeat: no-repeat, repeat;
}
```
