﻿`Phantom-pdf` recipe is using [phantomjs](http://phantomjs.org/) screen capture feature to print html content into pdf files. This approach is very productive in defining report templates and also the most used one with jsreport.

`Phantom-pdf` recipe is capable of rendering any html and javascript you provide. This means you can also use external javascript libraries or canvas to print visual charts.

##Basic settings

- margin - px or cm specification of margin used from page borders, you can also pass an `Object` or `JSON object string` for better control of each margin side. ex: `{ "top": "5px", "left": "10px", "right": "10px", "bottom": "5px" }`
- format- predefined page sizes containing A3, A4, A5, Legal, Letter
- width - px or cm page width, takes precedence over paper format
- height - px or cm page height, takes precedence over paper format
- orientation - portrait or landscape orientation
- headerHeight - px or cm height of the header in the page
- header- header html content
- footerHeight - px or cm height of the footer in the page
- footer - footer html content
- printDelay - delay between rendering a page and printing into pdf, this is useful when printing animated content like charts
- blockJavaScript - block executing javascript
- waitForJS - true/false

These basic settings are typically stored with the template, but you can also send them through [API calls](/learn/api)  inside the `template.phantom` property.


##Configuration

Use `phantom` node in the standard [config](/learn/configuration) file.
```js
phantom: {
  numberOfWorkers: 1
  timeout: 180000,
  allowLocalFilesAccess: false,
  defaultPhantomjsVersion: '1.9.8'
}
```

##Page breaks

Css contains styles like `page-break-before` you can use to specify html page breaks. This can be used as well with phantom-pdf to specify page breaks inside pdf files.

```html
<h1>Hello from Page 1</h1>

<div style='page-break-before: always;'></div>

<h1>Hello from Page 2</h1>

<div style="page-break-before: always;"></div>

<h1>Hello from Page 3</h1>
```

##Headers and footers
Header and footer is evaluated as it would be a full jsreport template. This means you can add for example an [child template](/learn/child-templates) reference into a header and it will be extracted. You can also use main template helpers or data in the header/footer.

##Page numbers
`Phantom-pdf` is providing special tags `{#pageNum}` and `{#numPages}` to header and footer which can be used to print page numbers.
```html
<div style='text-align:center'>{#pageNum}/{#numPages}</div>
```

Note that `Phantom-pdf` also evaluates javascript in the header and footer which you can use to modify the paging start for example:

```html
<span id='pageNumber'>{#pageNum}</span>
<script>
    var elem = document.getElementById('pageNumber');
    if (parseInt(elem.innerHTML) <= 3) {
        elem.style.display = 'none';
    }
</script>
```


##National characters

`Phantom-pdf` is currently not able to print some national characters by default. To be able to print correct national characters into pdf you need to set `utf-8` charset in your html first.

```html
<html>
  <head>
    <meta content="text/html; charset=utf-8" http-equiv="Content-Type">
  </head>
  <body>
     ščřžý
  </body>
</html>

```

##Images in header and footer
You may notice image won't show up when you try to add it into phantom header in a common way:

```html
<img src='http://domain.com/foo.jpg'>
```
This is because header and footer is printed into pdf in a synchronous way. This mean any asynchronous request like getting image won't finish in time. This is current limitation of phantom.js

Solution:    
Add the same image to template content and hide it with style display:none. Then you can add it to the header and it will show up because it is already cached and no asynchronous request is needed. This is required to do for both image referenced with url as well for Data URI scheme base64 image.

##Styles in header and footer
phantomjs doesn't let you to link an external style to header and footer. You need to always inline it using `<style>` tag. If this becomes tedious, you can use [child template](https://jsreport.net/learn/child-templates) to extract and reuse it.

##Printing triggers
You may need to postpone pdf printing until some javascript async tasks are processed. If this is your case set the `phantom.waitForJS=true` in the API or `Wait for printing trigger` in the studio menu. Then the printing won't start until you set `window.JSREPORT_READY_TO_START=true` inside your template's javascript.
```html
...
<script>
    // do some calculations or something async
    setTimeout(function() {
        window.JSREPORT_READY_TO_START = true; //this will start the pdf printing
    }, 500);
    ...
</script>
```

##macOS sierra
The default phantomjs@1.9 currently doesn't work on the macOS sierra update, you need to use the phantomjs2. See the next chapter.

##phantomjs2
The recipe default installation uses phantomjs@1.9. You can additionally install other versions and use them in parallel.

1. Install additional phantomjs using
`npm install phantomjs-exact-2-1-1`
2. Use jsreport studio to switch phantomjs version in properties or set `"template.phantom.phantomjsVersion":"2.1.1"` in api call

You can also set the default phantomjs version globally in the config:

```js
"phantom": {   
    "defaultPhantomjsVersion": "2.1.1"
  }
```

Note that phantomjs2 produces different sizes of fonts. Also it doesn't support repeating `thead` when the table spawns multiple pages.


##Twitter Bootstrap
Using a responsive css framework for printing pdf may not be the best idea. However it still works. Only thing you need to keep in mind is that output pdf typically won't look the same as html because bootstrap includes different printing styles under `@media print`.

##Fonts
The fonts can be easily embeded into pdf reports using [assets](https://jsreport.net/learn/assets) extension. You can find the tutorial how to do it [here](https://jsreport.net/blog/fonts-in-pdf).

##Different sizes on Windows vs Unix
Both phantomjs 1.9.8 as well as 2.1.1 are producing different sizes of pdf elements when rendered on Windows vs Unix platform. This issue can be tracked [here](https://github.com/ariya/phantomjs/issues/12685). We recommend to design your reports on the same OS where you plan to run your production jsreport instance because of this. If this is not an option for you, you may try to apply the following css to adapt the sizes on your local or production templates.

```css
body {
  transform-origin: 0 0;
  -webkit-transform-origin: 0 0;
  transform: scale(0.654545);
  -webkit-transform: scale(0.654545);
}
```
