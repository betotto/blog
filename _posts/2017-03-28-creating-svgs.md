---
layout: post
title: Creating SVGs for CSS
date: 2017-08-25 12:00:00 -0700
icon: code
tags: CSS
---

I notice that one important thing that every front-end developer should know, is how to create an SVG image and how to include it in the CSS file as base64 code for optimization, this way is preferred in many situations because you don't need a new "request" for the image itself. Here I present a little way to create the base64 string from an SVG code using NodeJS.

### Prerequisites

- Nodejs 6^


### The process

I have Visual Studio Code installed in my computer and I included a plugin **[SVG viewer](https://marketplace.visualstudio.com/items?itemName=cssho.vscode-svgviewer)**. This particular tool allows me to easily see all my changes to the code and fastly create the icon, This example is the icon next to the title of the post, something like this "< >".

```html
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 40 40">
  <style type="text/css">
    #code-icon {
      stroke: red;
      stroke-width: 4px;
      fill: none;
    }
  </style>
  <g id="code-icon">
    <path d="M 15 8 L 5 20 L 15 32 " />
    <path d="M 25 8 L 35 20 L 25 32 " />
  </g>
</svg>
```

One note: even if I going to use this image as code, it's important to save the code in a file, in case that you want to modify it later I named my image as *code.svg*, once you have your own image or the previous one, we can use the Buffer class from NodeJS to convert it to a base64 string, you need a javascript file like *buffersvg.js* and paste the next code:

```javascript
const fs = require('fs');

fs.readFile('PathtoSVG/code.svg', 'utf-8', (err, data) => {
  let svg = data.replace(/\n/g, '').replace(/  /g, '');
  console.log('data:image/svg+xml;base64,'.concat(new Buffer(svg).toString('base64')));
});
```

Remember to replace "PathToSVG" with the correct location of your SVG image, and now we just run it:

```bash
$ node buffersvg.js

data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCA0MCA0MCI+PHN0eWxlIHR5cGU9InRleHQvY3NzIj4jY29kZS1pY29uIHtzdHJva2U6IHdoaXRlO3N0cm9rZS13aWR0aDogNHB4O2ZpbGw6IG5vbmU7fTwvc3R5bGU+PGcgaWQ9ImNvZGUtaWNvbiI+PHBhdGggZD0iTSAxNSA4IEwgNSAyMCBMIDE1IDMyICIgLz48cGF0aCBkPSJNIDI1IDggTCAzNSAyMCBMIDI1IDMyICIgLz48L2c+PC9zdmc+
```

That's it, now you can include the base64 string in any element of HTML as background-image.

```css
.code {
  background-image: url('data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCA0MCA0MCI+PHN0eWxlIHR5cGU9InRleHQvY3NzIj4jY29kZS1pY29uIHtzdHJva2U6IHdoaXRlO3N0cm9rZS13aWR0aDogNHB4O2ZpbGw6IG5vbmU7fTwvc3R5bGU+PGcgaWQ9ImNvZGUtaWNvbiI+PHBhdGggZD0iTSAxNSA4IEwgNSAyMCBMIDE1IDMyICIgLz48cGF0aCBkPSJNIDI1IDggTCAzNSAyMCBMIDI1IDMyICIgLz48L2c+PC9zdmc+');
  background-size: 30px 30px;
  background-position: center center;
  background-repeat: no-repeat;
}
```

### Help me

I'm writing to remember important things from my job as well as to share the little things that I consider important, as you can read I'm not a native speaker so if you identify any error in grammar or code I'll correct it. Please let me now in the issues section of this little project **[blog issues](https://github.com/betotto/blog/issues)**.
