---
layout: post
title: A Stylish Dropdown With Mostly CSS
summary: Learn how to make simple, but stylish dropdown menus with mostly CSS
reposted_from: http://labs.teecom.com/blog/a-stylish-dropdown-with-mostly-css
reposted_from_title: TEECOMlabs blog
---

In my experience, dropdown tutorials usually fall into one of two categories:

1. Fairly small in scope, but very dated looking

2. Very pretty, but overcomplicated with a lot of JavaScript

Thankfully, I find there’s a nice middle ground for simple but stylish dropdown
menus. In this example, we’ll use mostly CSS with a sprinkle of JavaScript to
glue everything together. You can view the final example on
[Codepen](https://codepen.io/tommyschaefer/pen/NzvJvg).

![](https://dl.dropboxusercontent.com/s/4p8xo9gkpa69fjp/Stylish%20Dropdown.gif?dl=0)

## HTML

To start, we add the HTML structure of our navigation bar with dropdown. We
include a dropdown toggle and a list of dropdown items.

```html
<nav class="nav">
    <span class="logo">A Company</span>

    <div class="dropdown">
        <div class="dropdown__toggle">
        <span>User Name</span>
      <i class="fas fa-angle-down toggle__indicator"></i>
      </div>

    <ul class="dropdown__items">
        <li><a href="#" class="dropdown__item">Profile</a></li>
      <li><a href="#" class="dropdown__item">Settings</a></li>
      <li><a href="#" class="dropdown__item">Log Out</a></li>
      </ul>
    </div>
</nav>
```

## JavaScript

Now we add our JavaScript to act as the glue between our CSS and HTML.

When a visitor clicks on the dropdown toggle, classes are applied to the
dropdown indicator and  items list. When the window is clicked, as long as the
user isn't clicking on the dropdown, these active classes are removed.

```javascript
var dropdown = document.querySelector(".dropdown");
var toggle = document.querySelector(".dropdown__toggle");
var indicator = document.querySelector(".toggle__indicator");
var items = document.querySelector(".dropdown__items");

toggle.addEventListener("click", () => {
  indicator.classList.toggle("toggle__indicator_active");
  items.classList.toggle("dropdown__items_active");
});

window.addEventListener("click", (e) => {
  if (dropdown.contains(e.target)) {
    return;
  }

  indicator.classList.remove("toggle__indicator_active");
  items.classList.remove("dropdown__items_active");
});
```

## CSS

After defining some styles for the page and the navigation bar, we dig into the
dropdown styles. We define base styles for the toggle, indicator, and items.
Additionally, we add styles for the active classes mentioned in the previous
section. These styles define what an active, or open, dropdown menu looks like.

A few areas of note:

- In the `body` styles, we set `-webkit-font-smoothing` to `antialiased` to
  prevent jittering animation when revealing the dropdown items in WebKit
  browsers

- In the dropdown items active class, `.dropdown__items_active`, we set
  `transform` to `translateY(5px)` to give the feeling that the dropdown items
  are dropping out of the toggle

- In the indicator active class, `.toggle__indicator_active`, we set `transform`
  to `rotate(180deg)` to rotate the downward facing arrow 180 degrees

```css
html,
body {
  -webkit-font-smoothing: antialiased;
  background: #fff;
  padding: 4rem;
}

.nav {
  display: flex;
  justify-content: space-between;
  margin: 0 auto;
  max-width: 400px;
  text-align: center;
}

.dropdown {
  position: relative;
}

.dropdown__toggle {
  align-items: center;
  cursor: pointer;
  display: flex;
  justify-content: space-between;
}

.toggle__indicator {
  -webkit-transform: rotate(0);
  margin-left: 0.5rem;
  transform: rotate(0deg);
  transition: all 0.25s ease;
}

.toggle__indicator_active {
  -webkit-transform: rotate(180deg);
  transform: rotate(180deg);
}

.dropdown__items {
  -webkit-padding-start: 0;
  background: #fff;
  border-radius: 3px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  list-style: none;
  margin-top: 5px;
  min-width: 8rem;
  opacity: 0;
  pointer-events: none;
  position: absolute;
  right: 0;
  transition: all 0.25s ease;
  white-space: nowrap;
  z-index: 1;
}

.dropdown__items_active {
  -webkit-transform: translateY(5px);
  opacity: 1;
  pointer-events: auto;
  transform: translateY(5px);
}

.dropdown__item {
  color: #000;
  display: block;
  padding: 1rem;
  text-decoration: none;
  transition: all 0.3s ease;
}

.dropdown__item:hover {
  background: #eef6fc;
  color: #2753a5;
}
```
