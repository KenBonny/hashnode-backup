---
title: "Branding an Angular site"
datePublished: Mon May 07 2018 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0ipqbr00xizfs1erit1ow9
slug: branding-an-angular-site
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617380707442/04dzU9h1g.jpeg
tags: css, angular, scss, branding, styling

---


For a client, I'm working on a portal for their customers. This portal needs to be branded according to the logged-on client. If Microsoft does business with my client and can log into the portal, then the Microsoft logo should appear and all highlight colours should be red, green, blue and yellow.

In the back-end, there is a _brand-template.scss_ file. The CSS in this file will overwrite the CSS from the Bootstrap theme used in the portal. To be able to overwrite the CSS in the Angular app, the style should be included after the Bootstrap CSS is loaded. The CSS attributes that contain branding colours need to be placed in the _style.scss_ file that is served from the Angular static resources. If the brand specific CSS is placed in a .component.css file, the colours will not be overwritten because the .component.css file is loaded after the _brand-template.scss_ CSS. I also recommend having default colours that look nice. In case the branded CSS does not load, the site will still look decent instead of broken.

To get the branded CSS, there is a branding controller that exposes an endpoint /branding/style.css. This will first fetch the contents of the _brand-template.scss_ file and retrieve the settings for the logged on customer that contain the brand colours. The code then replaces all placeholders in the _brand-templace.css_ file with the colour properties from the settings. These properties are selected by the presence of the the _Color_ keyword. Besides replacing the placeholders, no further processing will take place. So I cannot use SCSS features in the _brand-templace.scss_ file, because that will generate an incorrect CSS file.

```
private static string ReplacePlaceholders(string template, Settings settings)
{
    var colorProperties = settings.GetType()
                                  .GetProperties()
                                  .Where(x => x.Name.Contains("Color"))
                                  .ToArray();
    foreach (var property in colorProperties)
    {
        var color = property.GetValue(settings).ToString();
        template = template.Replace($"${property.Name}", color);
    }
    return template;
}
```

At the top of the _brand-template.scss_, I list the available placeholders so I can easily check which properties I can use. I added them manually, so when there is a new colour setting, it needs to be added here as well. The reason I used a .scss file, is so I can place variables in the file such as _$BackgroundColor_ without generating errors from my IDE and still have auto completion when writing CSS.

```
/*
 * available variables
 * $TitleColor
 * $UrlColor
 */
h1 {
  color: $TitleColor;
}
```

Also, to overwrite the CSS classes, they need to appear in the correct order. `.active .btn-primary` is different from `.btn-primary .active`. Check the order in the Chrome/FireFox/Edge/BrowserOfChoice developer tools to be sure.

To get the branded image and logo, I have a separate endpoint for each image. Currently there are two, a small and a big logo. I have an /branding/logo-small (and also large) endpoint. This will look at the same settings as before to find the location of the logos. The controller then returns the image as a byte array. Do not forget to set the content type. Most browsers recognise the type from the stream, except, of course, Internet Explorer (IE). If I don't set the content type to the correct type (such as `image/png` or `image/jpeg`), IE will not display the image.

I'm not entirely happy with this approach for the CSS because it feels like I'm abusing an SCSS file. It looks and feels like an SCSS file, but it isn't. Unfortunately, I don't know a better way at the moment. If there is a better way of branding an Angular site, I'm not aware of it. If any of my readers know of a better way, do [get in touch](http://kenbonny.net/contact/) and let me know. I always like an opportunity to learn.

**Update**: A reader of this blog pointed out that there are [CSS variables](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_variables) now. I'd like to thank [Filip Blondeel](https://twitter.com/filipblondeel) for bringing this to my attention. Thanks for the teaching moment.

%[https://twitter.com/filipblondeel/status/993441792682004481]

This means that I can move the CSS in the _brand-template.scss_ to a statically served CSS in the front-end. First benefit, this style.css file can be cached both server and client side. It will look something like this:

```
h1 {
  color: var(--TitleColor, green);
}
```

Notice that I replaced the `$TitleColor` by `var(--TitleColor, green)`. This indicates that a parsing is necessary. The CSS will look for a definition of `--TitleColor` and will fall back to the colour green if no variable is found. Side note, the variables are case sensitive. A wrong case will mean the fallback will be used.

In the _brand-template.css_ (notice I'm not using the _.scss_ file extension anymore, this will be a legitimate CSS file), I define the variables. This can in a separate file. The front-end file will overwrite the colours and the /branding/style.css will now serve the specific colours.

```
:root {
  --TitleColor: $TitleColor;
}
```

The string replace is still being used to put the actual color in the _brand-template.css_ file. Since this is now a valid CSS file, no more ambiguity is possible. All CSS tips and tricks can be safely used and developers need no warnings not to use certain techniques. This is the part I like the best: standard practices are used.

The only down side I can find about this is that [IE11 does not support this](https://caniuse.com/#search=variables). At all. It won't even recognise the fallback colour. Ugh, when will we all be rid of this monstrosity.
