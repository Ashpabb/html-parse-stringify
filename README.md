# html-parse-stringify

This is an *experimental lightweight approach* to enable quickly parsing HTML into an AST and stringify'ing it back to the original string.

As it turns out, if you can make a the simplifying assumptions about HTML that all tags must be closed or self-closing. Which is OK for *this* particular application. You can write a super light/fast parser in JS with regex.

"Why on earth would you do this?! Haven't you read: http://stackoverflow.com/questions/1732348/regex-match-open-tags-except-xhtml-self-contained-tags ?!?!"

Why yes, yes I have :)

But the truth is. If you *could* do this in a whopping grand total of ~600 bytes (min+gzip) as this repo shows. It potentially enables DOM diffing based on a HTML strings to be super light and fast in a browser. What is that you say? DOM-diffing?

Yes. 

React.js essentially pioneered the approach. With Reach you render to a "virtual DOM" whenever you want to, and the virtual DOM can then diff against the real DOM (or the last virtual DOM) and then turn that diff into whatever transformations are necessary to get the *real* DOM to match what you rendered as efficiently as possible.

As a result, when you're building a single page app, you don't have to worry so much about bindings. Instead, you simple re-render to the virtual DOM whenever you know something's changed. All of a sudden being able to have `change` events for individual properties becomes less important, instead you can just reference those values in your template whenever you think something changed.

Cool idea, right?!

## So why this? 

Well, there are other things React expects me to do if I use it that I don't like. Such as the custom templating and syntax you have to use. 

If, hypothetically, you could instead diff an HTML string (generated by *whatever* templating language of your choice) against the DOM, then you'd get the same benefit, sans React's impositions. 

This may all turn out to be a bad idea altogether, but initial results seem promising when paired with [virtual-dom](https://github.com/Matt-Esch/virtual-dom).

But you can't just diff HTML strings, as simple strings, very easily, in order to diff two HTML node trees you have to first turn that string into a tree structure of some sort. Typically, the thing you generate from parsing something like this is called an AST (abstract syntax tree).

This lib does exactly that. 

It has two methods:

1. parse
2. stringify

## `.parse(htmlString, options)`

Takes a string of HTML and turns it into an AST, the only option you can currently pass is an object of registered `components` whose children will be ignored when generating the AST.

## `.stringify(AST)`

Takes an AST and turns it back into a string of HTML.

## What does the AST look like?

See comments in the following example:

```js
var HTML = require('html-parse-stringify')

// this html:
var html = '<div class="oh"><p>hi</p></div>';

// becomes this AST:
var ast = HTML.parse(html);


console.log(ast);
/*
{
    // can be `tag`, `text` or `component`
    type: 'tag',

    // name of tag if relevant
    name: 'div',
    
    // parsed attribute object
    attrs: {
        class: 'oh'
    },

    // whether this is a self-closing tag
    // such as <img/>
    voidElement: false,

    // an array of child nodes
    // we see the same structure
    // repeated in each of these
    children: [
        {
            type: 'tag',
            name: 'p',
            attrs: {},
            voidElement: false,
            children: [
                // this is a text node
                // it also has a `type`
                // but nothing other than
                // a `content` containing
                // its text.
                {
                    type: 'text',
                    content: 'hi'
                }
            ]
        }
    ]
}
*/
```

## the AST node types

### 1. tag

properties:

- `type` - will always be `tag` for this type of node
- `name` - tag name, such as 'div'
- `attrs` - an object of key/value pairs. If an attribute has multiple space-separated items such as classes, they'll still be in a single string, for example: `class: "class1 class2"`
- `voidElement` - `true` or `false`. Whether this tag is a known void element as defined by [spec](http://www.w3.org/html/wg/drafts/html/master/syntax.html#void-elements).
- `children` - array of child nodes. Note that any continuous string of text is a text node child, see below.

### 2. text

properties:

- `type` - will always be `text` for this type of node
- `content` - text content of the node

### 3. component

If you pass an object of `components` as part of the `options` object passed as the second argument to `.parse()` then the AST won't keep parsing that branch of the DOM tree when it one of those registered components.

This is so that it's possible to ignore sections of the tree that you may want to handle by another "subview" in your application that handles it's own DOM diffing.

properties:

- `type` - will always be `component` for this type of node
- `name` - tag name, such as 'div'
- `attrs` - an object of key/value pairs. If an attribute has multiple space-separated items such as classes, they'll still be in a single string, for example: `class: "class1 class2"`
- `voidElement` - `true` or `false`. Whether this tag is a known void element as defined by [spec](http://www.w3.org/html/wg/drafts/html/master/syntax.html#void-elements).
- `children` - it will still have a `children` array, but it will always be empty.


## credits

If this sounds interesting you should probably follow [@HenrikJoreteg](http://twitter.com/henrikjoreteg) and [@Philip_Roberts](twitter.com/philip_roberts) on twitter to see how this all turns out. 

## license

MIT
