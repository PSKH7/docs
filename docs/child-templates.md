> Decompose templates into small reusable pieces

## Basics
Sometimes you'll get into situation where you find out that your report templates are starting to be too complicated. The html gets bigger and you need to copy paste several parts between templates quite often. You may consider using `Child templates` extension in this situation.

`Child templates` allows you to include report templates into each other even in multiple levels. The realization of this is very easy. You just create a standard template for a part you want to reuse. Let's call it `Some table`. And then in every template  you want to use in just insert special tag `{#child Some table}`.

## How it works
`Child templates` works the way that for every report rendering it searches for special tag `{#child [name]}` and replaces it content with real report output. It basicaly calls whole rendering process for child template and includes result instead of `{#child [name]}` special tag.

**It's important to understand that child templates really do the whole rendering process and you probably want to set `recipe` of child templates to `html` otherwise jsreport will not be able to render root template with `phantom-pdf` recipe.**    

> `Child templates` extension can be used to centralize your css styles into one child template and include css to all other templates.

## Set child template parameters

The root request input data cascade through child templates, but you can also pass additional data to the child template through its declaration
```html
{#child myChildTemplate @data.paramA=foo}
```

Or you can override existing child template attributes like its recipe.
```html
{#child myChildTemplate @template.recipe=html @options.language=sp}
```

## Using static content

`Child templates` really excels when you want to break a big template in separate child templates for better organization, this is the best use case for `Child templates` because it supports dynamic content just like normal templates, however if you only have static content (like a group of styles, [html layout](https://jsreport.net/blog/template-layouts), etc) the best option (simple and with more performance) is to use [assets](https://jsreport.net/learn/assets) extension to handle that kind of content.
