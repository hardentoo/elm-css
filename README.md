[![Logo](https://raw.githubusercontent.com/rtfeldman/elm-css/c78bacb4420696b8d22d9429616a21c7ab20def3/assets/logo.png)](http://package.elm-lang.org/packages/rtfeldman/elm-css/latest)

# elm-css [![Version](https://img.shields.io/npm/v/elm-css.svg)](https://www.npmjs.com/package/elm-css) [![Travis build Status](https://travis-ci.org/rtfeldman/elm-css.svg?branch=master)](http://travis-ci.org/rtfeldman/elm-css) [![AppVeyor build status](https://ci.appveyor.com/api/projects/status/0j7x0mpggmtu6mms/branch/master?svg=true)](https://ci.appveyor.com/project/rtfeldman/elm-css/branch/master)

`elm-css` lets you define CSS in Elm. (For an Elm styling system that is a
complete departure from CSS, check out [style-elements](http://package.elm-lang.org/packages/mdgriffith/style-elements/latest).)

Here's an example of how to define some `elm-css` styles:

```elm
module MyCss exposing (..)

import Css exposing (..)
import Css.Elements exposing (body, li)
import Css.Namespace exposing (namespace)


type CssClasses
    = NavBar


type CssIds
    = Page


css =
    (stylesheet << namespace "dreamwriter")
    [ body
        [ overflowX auto
        , minWidth (px 1280)
        ]
    , id Page
        [ backgroundColor (rgb 200 128 64)
        , color (hex "CCFFFF")
        , width (pct 100)
        , height (pct 100)
        , boxSizing borderBox
        , padding (px 8)
        , margin zero
        ]
    , class NavBar
        [ margin zero
        , padding zero
        , children
            [ li
                [ (display inlineBlock) |> important
                , color primaryAccentColor
                ]
            ]
        ]
    ]


primaryAccentColor =
    hex "ccffaa"
```

Here's what you can do with this code:

* You can *generate a `.css` file from it.*
* You can *use it to generate type-checked inline styles.*
* You can *share `NavBar` and `Page`* with your Elm view code, so your classes and IDs can never get out of sync due to a typo or refactor.
* You can *move this code into your view file* and have your styles live side-by-side with your view functions themselves.

`elm-css` works hard to prevent invalid styles from being generated; for example,
if you write `color "blah"` or `margin (rgb 1 2 3)`, you'll get a type mismatch. If you write `(rgb 3000 0 -3)` you'll get a build-time validation error (RGB values must be between 0 and 255) if you try to compile it to a stylesheet.

`elm-css` draws inspiration from the excellent [Sass](http://sass-lang.com/), [Stylus](http://stylus-lang.com/), and [CSS Modules](http://glenmaddern.com/articles/css-modules). It includes popular features like:

* [style batching](http://package.elm-lang.org/packages/rtfeldman/elm-css/11.2.0/Css#Style) (basically mixins)
* [namespaces](http://package.elm-lang.org/packages/rtfeldman/elm-css/latest/Css-Namespace#namespace)
* [nested media queries](https://davidwalsh.name/write-media-queries-sass) (and nested selectors in general, like how [Sass](http://sass-lang.com/) does them)

There are two popular approaches to using it; you can use either or combine both of these, depending on your needs and preferences.

#### Approach 1: Inline Styles

One way to use `elm-css` is for inline styles, using the `asPairs` function:

```elm
styles =
    Css.asPairs >> Html.Attributes.style

button [ styles [ position absolute, left (px 5) ] ]
    [ text "Whee!" ]
```

This approach is the simplest way to get started with `elm-css`. One advantage of inline styles is that these can be dynamically changed at runtime; a limitation is that CSS pseudo-classes and pseudo-elements cannot be used with inline styles.

#### Approach 2: Generating CSS files

You can also use `elm-css` as a CSS preprocessor which generates separate .css files.

To do this, you will need to install both the node module and the Elm library:

    npm install -g elm-css
    elm package install rtfeldman/elm-css

Then you'll need a special module with a port for `elm-css` to access:

```elm
port module Stylesheets exposing (..)

import Css.File exposing (CssFileStructure, CssCompilerProgram)
import MyCss


port files : CssFileStructure -> Cmd msg


fileStructure : CssFileStructure
fileStructure =
    Css.File.toFileStructure
        [ ( "index.css", Css.File.compile [ MyCss.css ] ) ]


main : CssCompilerProgram
main =
    Css.File.compiler files fileStructure
```

Run `elm-css` on the file containing this `Stylesheets` module.
Then include that css file in your web page.

The above `elm-css` stylesheet compiles to the following .css file:

```css
body {
    overflow-x: auto;
    min-width: 1280px;
}

#dreamwriterPage {
    background-color: rgb(200, 128, 64);
    color: #CCFFFF;
    width: 100%;
    height: 100%;
    box-sizing: border-box;
    padding: 8px;
    margin: 0;
}

.dreamwriterNavBar {
    margin: 0;
    padding: 0;
}

.dreamwriterNavBar > li {
    display: inline-block !important;
    color: #ccffaa;
}
```

Try it out! (make sure you already have [elm](http://elm-lang.org) installed, e.g. with `npm install -g elm`)

```
$ npm install -g elm-css
$ git clone https://github.com/rtfeldman/elm-css.git
$ cd elm-css/examples
$ elm-css src/Stylesheets.elm
$ less homepage.css
```

A gentle introduction to some of the features of `elm-css` is also available in
[the tutorial](Tutorial.md).

### Examples

There are a few examples to check out!

- [json-to-elm](https://github.com/eeue56/json-to-elm) which can see be seen live [here](https://noredink.github.io/json-to-elm)
- the [examples](https://github.com/rtfeldman/elm-css/tree/master/examples) folder, which contains a working project with a README
- the example above

#### Using `elm-css` with Elm view code

Here's how to use `elm-css` in your projects:

In your Elm code, use the same union types to represent classes and ids. Then they can't get out of sync with your CSS. To do this, you'll need special versions the of `id`, `class`, and `classList` functions from `elm-html`.

Install the handy package that combines `elm-html` and `elm-css`:

     elm package install rtfeldman/elm-css-helpers

The `Elm.CssHelpers.withNamespace` returns a record full of handy functions. Use that, and then construct Html using classes and ids defined in your union types. For example:

```elm
module MyView exposing (..)


import Html.CssHelpers
import MyCss


{ id, class, classList } =
    Html.CssHelpers.withNamespace "dreamwriter"


view =
    Html.div []
        [ Html.div [ class [ MyCss.NavBar ] ] [ Html.text "this has the NavBar class" ]
        , Html.div [ id MyCss.Page ] [ Html.text "this has the Page id" ]
        ]

```

#### Missing CSS properties

`elm-css` is still in development. Not all CSS properties have been added yet.
If you run into this problem, `elm-css` includes the `property` function. It takes
two `Strings`; the property key, and its value.

**e.g.**

We want `z-index`, but suppose `elm-css` did not implement it. We would define it ourselves:

```elm
import Css exposing (..)

zIndex : Int -> Style
zIndex i =
    property "z-index" <| toString i
```

Now `zIndex 9999` is available to use inside our `Stylesheet`.

## Related Projects

* [Elm CSS Normalize](https://github.com/scottcorgan/elm-css-normalize)
