# The self-reactive CSS implementation for contemporary web applications

## Prologue

As the progressive involving of web applications, the requirement and complexity of UI (User Interface) is also increasing, for example, based on different states of the current application process, a certain component may have to display different visual elements or alter the visual element’s properties such as colour, shape or size etc. In order to achieve this purpose, quite often there are DOM manipulations involved to insert or replace HTML nodes or class names, which leads to quite a bit of non-business logic related codes.

To help reducing the stress of low-level labour works like DOM manipulations, there are libraries such as React.js or Vue.js that takes over the jobs like this therefore developers can focus on the logics sorely. It is good to free developers from such not-so-meaningful works, however, is it possible to take one step forward to avoid such DOM editing at all and let CSS handle the visual changes on its own?

Well, CSS is created for building visual effects, if we can maximise the capability of CSS then we no longer need the code written purely for updating DOM elements. In this case, our code would be much cleaner and easier to read. In this article, I am going to explore the possibilities to achieve this goal with some common elements can be found on nearly every web application.

## Exploration

### Checkbox

Checkbox probably is one of the most common elements in web applications, because the visually dull and dumbness of the default looking (not to mention the inconsistency across different browsers), many projects decided to build custom beautiful looking checkbox on their own. Of course, there are libraries like Bootstrap can be adopted directly to save the trouble of re-inventing the wheel, it works for the purpose, does it work elegantly? Hmm, maybe not.

Most common implementations of the customised checkboxes I’ve seen all these years in the industry commonly involves following things (depends on the actual design of UI):

1. Multiple level nested elements inside HTML tag `label`.
2. DOM manipulation to add / remove class names to render different styles.
3. Dependency upon font-icons or Unicode icons.

It’s quite a bit of additions for a fundamental simple checkbox, basically just an `input` and a related `label`. The goal to achieve here is to keep the HTML node as simple as it is and implement a custom styled checkbox that can change styles when checkbox state changes (checked, unchecked, disabled). Therefore, this is what the HTML would look like:

```html
<input type=”checkbox” class="checkbox-input" id="chekbox-id">
<label class="checkbox-label" for="checkbox-id">Checkbox label</label>
```

In order to show a customised checkbox, we have to hide the input element:

```scss
.checkbox-input {
  display: none;
}
```

For now, we only have the label text showing, next step is to build the square box and the &#x2713; icon. To achieve this, we will use pseudo element
`::before`. Here's what the code looks like:

```scss
.checkbox-label {
  cursor: pointer;

  &::before {
    content: "";
    display: inline-block;
    width: 16px;
    height: 16px;
    border: 1px solid #cccccc;
  }
}
```

At this stage we have an empty checkbox without the &#x2713; icon in it. It's a bit of challenge to build the icon without Unicode or Font-icons, but it's achievable  with a compound of few layers of linear-gradient backgrounds. Here's the code to construct the &#x2713;:

```scss
background: linear-gradient(
              45deg,
              rgba(#fff, 0) 26%,
              #007bbd 26%,
              #007bbd calc(26% + 1px),
              rgba(#fff, 0) calc(26% + 2px)
            ) no-repeat -4px -4px,
            linear-gradient(
              -45deg,
              rgba(#fff, 0) 40%,
              #007bbd 40%,
              #007bbd calc(40% + 1px),
              rgba(#fff, 0) calc(40% + 2px)
            ) no-repeat 3px -3px;
```

Now we have all the elements, how do we make the checkbox changes on click? The key is using CSS selector `:checked`, here's the code snippet of the logic:

```scss
.checkbox-input {
  // Styles when not checked
  &:checked {
    & + .checkbox-label {
      // Styles when input checked
    }
  }
  &:disabled {
    & + .checkbox-label {
      // Styles when input disabled
    }
    &:checked + .form-check-label {
      // Styles when checked & disabled
    }
  }
}
```

At last, this is a working example putting everything together: <https://codepen.io/adamyung/pen/eYQjaYO>

### Toggle button

Like the checkbox above, toggle button shares the same functional logic but with a more stylish look. We will start with the similar html elements without label text:

```html
<input id="toggle-btn" type="checkbox" class="toggle-btn-input" />
<label for="toggle-btn" class="toggle-btn-label"></label>
```

Also similarly, we will put input to hidden.

```scss
.toggle-btn-input {
  display: none;
}
```

Then construct the background frame of the toggle button:

```scss
.toggle-btn-label {
  position: relative;
  display: block;
  width: 50px;
  height: 25px;
  border: 1px solid rgba(#fff, 0);
  border-radius:  25px;
}
```

Next step is building the round toggle button in it, again, we will use pseudo element `::before` to achieve it:

```scss
&::before {
  content: "";
  display: block;
  position: absolute;
  top: 0;
  width: 50%;
  height: 100%;
  background-color: #fff;
  border-radius:  25px;
}
```

The style change controlled by the input state update uses the same logic code as the checkbox in previous section. This is a working example putting everything together: <https://codepen.io/adamyung/pen/erpbpK>

### Tooltip

Tooltip utility is something we see quite often in the web applications as a popular replacement for the default `title` attribute (which is fairly unnoticeable and slow to response).

The commonly adopted library `Bootstrap` uses library `popper.js` for its implementation. It works by inserting a bunch of elements to the end of html body, such as

```html
<div class="tooltip custom-tooltip bs-tooltip-auto fade show" role="tooltip" id="tooltip675661" data-popper-placement="top" style="position: absolute; inset: auto auto 0px 0px; margin: 0px; transform: translate(429px, 1159px);">
  <div class="tooltip-arrow" style="position: absolute; left: 0px; transform: translate(94px, 0px);"></div>
  <div class="tooltip-inner">This top tooltip is themed via CSS variables.</div>
</div>
```

As we can see here, the Tooltip's shape and position are calculated by JavaScript then insert into DOM with generated html contents with inline styles. It works, but quite a bit overkill for just a Tooltip in my opinion. Considering the original Tooltip we are replacing is merely a `title` attribute.

Again, our goal is keeping the DOM elements as minimum as possible and not utilising JavaScript for lightweight purpose. The very first question is, how simple the html possibly can be to the extreme? As it turns out, it can be as simple as:

```html
<div class="hover-tooltip" data-tooltip="Tooltip message">
  <span>Tooltip text</span>
</div>
```

The Tooltip content itself can be rendered inside a pseudo element such as:

```scss
&::after {
  content: attr(data-tooltip);
  display: block;
  white-space: nowrap;
  font-size: 12px;
  font-weight: 400;
}
```

Now it comes to the difficult part, how can we fit the background and position arrows all into one pseudo element `::before`? The trick here is to do a multiple layered `linear-gradient` background. Here's an example of top positioned Tooltip:

```scss
&::after {
  left: 50%;
  top: calc(-100% - 10px);
  transform: translate(calc(-50% + 0px), 0);
  padding: 4px 8px 10px 8px;
  background: linear-gradient(0deg, rgba(#fff, 0) 7px, rgba(var(--tooltip-bg), 1) 7px),
    linear-gradient(-45deg, rgba(#fff, 0) 50%, rgba(var(--tooltip-bg), 1) 50%)
      calc(50% + 4px) calc(100% - 2px) / 8px 8px no-repeat,
    linear-gradient(45deg, rgba(#fff, 0) 50%, rgba(var(--tooltip-bg), 1) 50%)
      calc(50% - 4px) calc(100% - 2px) / 8px 8px no-repeat;
}
```

The most important part to make it look correct is to calculate the padding size to be matching up with background linear-gradient's colour transition and size. In the example above there's an extra 6px padding space at the bottom of Tooltip body, which will be used to place in the Tooltip arrows as part of background. While the first linear-gradient is to render the body background, the other two is for building each of the half arrow.

For different positions it will need to calculate the padding and background separately for each of them. This is a working example putting everything together: <https://codepen.io/adamyung/pen/abQQOPd>

## Conclusion

We have discussed three different implementations that can greatly simplify the DOM manipulation in order to let developers focus on the business logics. I believe there are much more other cases can adopt similar methodologies. I know many frontend developers who detests CSS due to its irrational nature, however it actually quite logical once you grab the sense of it. As an essential element for the contemporary web development components, obtaining a certain level of skills of CSS is certainly going to contribute to the beauty of the code you are going to create.
