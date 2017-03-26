# babel-plugin-transform-jsx-localize

Localize JSX as is, with very little extra markup.

The translator is inspired by [Genshi][1], [Kajiki][2], and [Tonnikala][3],
which are XML based templating languages in Python. The latter 2 compile a
template to a Python program, and while Kajiki writes source text, Tonnikala
writes code as [Abstract Syntax Trees][4]. This seemed like a good fit in the
Babel and React ecosystem, where AST parsing and rewriting is the norm.

## React

### In

```javascript
var simple = <p>Hello, World!</p>;

var placeholders = <p i18nMsg="name">Hello, { "World" }!</p>;

var multiplePlaceholders = <p i18nMsg="first, last">{ first_name } { last_name }</p>

var elements = <div i18nMsg="">
  Text content should be <strong>translated</strong>.
  <img src="/img/hello.jpg" alt="Text props should be translated" />
</div>;

var untranslated = <div lang="en">
  Text, elements, and attributes in an element with <code>lang</code> attribute
  shall be untouched. 
  <img src="/img/hello.jpg" alt="Hello, World!" />
</div>;
```

### Out

```javascript
var _babelPluginTransformJsxLocalize = require("babel-plugin-transform-jsx-localize");

var simple = React.createElement(
  "p",
  null,
  gettext("Hello, World!")
);

var placeholders = React.createElement(_babelPluginTransformJsxLocalize.Message, {
  format: "Hello, {name}!",
  component: React.createElement("p", null),
  expressions: {
    name: "World"
  },
  translator: gettext
});

var multiplePlaceholders = React.createElement(_babelPluginTransformJsxLocalize.Message, {
  format: "{first} {last}",
  component: React.createElement("p", null),
  expressions: {
    first: first_name,
    last: last_name
  },
  translator: gettext
});

var elements = React.createElement(
  _babelPluginTransformJsxLocalize.Message,
  {
    format: " Text content should be [1:translated]. [2:] ",
    component: React.createElement("div", null),
    expressions: {},
    translator: gettext
  },
  React.createElement("strong", null),
  React.createElement("img", { src: "/img/hello.jpg", alt: gettext("Text props should be translated")
  })
);

var untranslated = React.createElement(
  "div",
  { lang: "en" },
  "Text, elements, and attributes in an element with ",
  React.createElement(
    "code",
    null,
    "lang"
  ),
  " attribute shall be untouched.",
  React.createElement("img", { src: "/img/hello.jpg", alt: "Hello, World!" })
);
```

## Usage

Via `.babelrc`

### .babelrc

Without options:

```json
{
  "plugins": ["transform-jsx-localize"]
}
```

With options:

```json
{
  "plugins": [
    ["transform-jsx-localize", {
      "translator": "myTranslatorFun"
    }]
  ]
}
```

## Options

### `translator`

`string`, defaults to `gettext`.

Replace the function used when translating messages.

### `normalizeWhitespace`

`boolean`, defaults to `true`.

If `true`, replace sequences of white space characters with a single space.

## Status

Early alpha, working features are

- Simple translations.
- Text attribute translations.
- Complex translations with parameters and/or markup.

## TODO

- Tests!
- ~~Message extractor – initially target [gettext][5] catalogs.~~
- ~~Automatically inject `Message`, if needed.~~
- `Message` and other runtime parts should perhaps have their own package.
- ~~Improve `Message` format parser and add escaping support. Currently it is
  impossible to output translated text with parameters/markup that would
  contain escaped placeholders as text.~~

## Contributors

- Ilja Everilä

  [1]: https://pythonhosted.org/Genshi/
  [2]: https://pythonhosted.org/Kajiki/
  [3]: https://github.com/tetframework/Tonnikala/
  [4]: https://en.wikipedia.org/wiki/Abstract_syntax_tree
  [5]: https://en.wikipedia.org/wiki/Gettext
