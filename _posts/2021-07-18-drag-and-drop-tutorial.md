---
layout: post
title: "List drag-and-drop tutorial"
date: 2021-07-18
featured-image: /assets/images/html-drag-and-drop-api.png
featured-image-alt: HTML Drag and Drop API
---

This tutorial explains the basic concepts of the HTML Drag and Drop API and shows how to implement it in a list, including a simple row-shifting mechanism.

Lately, I've been implementing Web APIs more frequently. My go-to resource for understanding them is MDN Docs. However, reading documentation can be daunting at first.

Here, I've tried to simplify the essentials.

The goal of the exercise is to reorder a list by moving a row and placing it into another position (<a href="https://www.akasharojee.codes/list-drag-and-drop-tutorial/" target="_blank">view demo</a>). For this, we need to implement a drag-and-drop functionality.

We'll create the HTML skeleton, add the basic CSS (via Sass), and get to the JS. Let's dive in!

* Do not remove this line (it will not be displayed)
{:toc}

## HTML

Let's first create the HTML skeleton.

Each list item will have a checkbox, a text, and an icon.

This will be the HTML structure of the list item.

{% highlight html %}
<li>
  <div>
    <input type="checkbox">
    <p>List item text</p>
  </div>
  <button class="material-icons">drag_indicator</button>
</li>
{% endhighlight %}

**Note:** We use a `div` to wrap the checkbox and the text, because we'll use Flexbox to position the checkbox and the text on one end of the list item, and the button on the other end.

---

**<u>TASK</u>**

**Create an HTML file named `index.html`, add the HTML skeleton with 5 list items, and link to the Google Fonts Material Icons stylesheet in the file.**

---

For the drag indicator icon, we're using Google Fonts Material Icons. To import the stylesheet, add the following line in your `<head>`.

{% highlight html %}
<link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">
{% endhighlight %}

<a href="https://github.com/AkashaRojee/list-drag-and-drop-tutorial/blob/html/index.html" target="_blank">View the HTML file in its current state.</a>

## CSS

Let's add some basic CSS.

To use some fun features, we'll do it via Sass.

---

**<u>TASK</u>**

**Create an SCSS file named `style.scss`, compile it, and link to its CSS version in the HTML file.**

---

Create an SCSS file named `style.scss`.

To compile it:

- Via the terminal

  Run the following commands.

  ```
  npm install -g sass
  sass --watch style.scss:style.css
  ```

  If you don't have npm installed, _<a href="https://www.akasharojee.codes/2021/06/20/intro-to-nodejs-and-npm.html#installation" target="_blank">view the section **Installation** in this guide about Node.js and npm</a>_.

- Via an extension (if you use VS Code)

  Install the <a href="https://marketplace.visualstudio.com/items?itemName=ritwickdey.live-sass" target="_blank">VS Code extension "Live Sass Compiler"</a>.

  In VS Code, click "Watch Sass" in the status bar.

Add the link tag for the CSS stylesheet in the head of the HTML file.

{% highlight html %}
  <!-- ... -->
  <link href="style.css" rel="stylesheet">
</head>
<!-- ... -->
{% endhighlight %}

In `style.scss`:

&nbsp;1. Reset everything.

{% highlight scss %}
* {
  padding: 0;
  margin: 0;
  box-sizing: border-box;
}
{% endhighlight %}

&nbsp;2. Turn the list item elements into a flex row.

Create a class `flex-row`.

{% highlight scss %}
.flex-row {
  display: flex;
  flex-direction: row;
}
{% endhighlight %}

Instead of applying the `flex-row` class to every list item in the HTML file, extend the `flex-row` class to the list item element.

Change `.flex-row` to `%flex-row`, and apply it to `li`.

{% highlight scss %}
%flex-row {
  display: flex;
  flex-direction: row;
}

ul {
  li {
    @extend %flex-row;
  }
}
{% endhighlight %}

&nbsp;3. Turn the checkbox wrapper into a flex row as well.

{% highlight scss %}
li {
  //...

  div {
    @extend %flex-row;
  }
}
{% endhighlight %}

&nbsp;4. Remove the default styling of buttons.

**Note**: I use the button element for semantic HTML, but feel free to use a span.

{% highlight scss %}
li {
  //...

  button {
    all: initial;
  }
}

{% endhighlight %}

&nbsp;5. Add borders:
- Around the `ul`
- Between each `li`

Since the border values are the same everywhere, let's store them in a variable, and use them in the border properties.

{% highlight scss %}
$border-values: 1px solid black;

ul {
  border: $border-values;

  li {
    @extend %flex-row;

    & + li {
      border-top: $border-values;
    }

    //...
  }
}
{% endhighlight %}

&nbsp;6. Resize and position the list (center horizontally and add some margin at the top).

{% highlight scss %}
ul {
  border: $border-values;
  width: 25%;
  margin: 50px auto 0 auto;

  //...
}
{% endhighlight %}

&nbsp;7. Push the drag indicator button to the far right in the list item.

{% highlight scss %}
li {
  @extend %flex-row;
  justify-content: space-between;

  //...
}
{% endhighlight %}

&nbsp;8. Final stylings:

- Add some padding inside the list item and set the element's height

{% highlight scss %}
li {
  @extend %flex-row;
  justify-content: space-between;
  padding: 10px;
  height: 45px;

  //...
}
{% endhighlight %}

- Vertically align the children of the checkbox wrapper
- Add some margin between the checkbox and the list item

{% highlight scss %}
div {
  @extend %flex-row;
  align-items: center;
  
  input {
    margin-right: 5px;
  }
}
{% endhighlight %}

- When hovering over the drag indicator button, we want the the cursor to change to a move icon.

  Add the &:hover pseudo-class for the button.

{% highlight scss %}
button {
  //...

  &:hover {
    cursor: move;
  }
}
{% endhighlight %}


<a href="https://github.com/AkashaRojee/list-drag-and-drop-tutorial/blob/css/index.html" target="_blank">View the HTML file in its current state.</a>

<a href="https://github.com/AkashaRojee/list-drag-and-drop-tutorial/blob/css/style.scss" target="_blank">View the SCSS file in its current state.</a>

## JS

Now to the real thing!

### HTML Drag and Drop API

To use DND features in browsers, there is a Web API called <a href="https://developer.mozilla.org/en-US/docs/Web/API/HTML_Drag_and_Drop_API" target="_blank">**HTML Drag and Drop**</a>.

With the API, you can:
- Select a draggable element with a mouse.
- Drag the element to a droppable element (a.k.a. a drop zone)
- Drop the draggable element at the drop zone by releasing the mouse.

While dragging, a translucent image of the element follows the pointer.

#### Getting Started

To get started with DND, we first have to specify which elements are draggable and which elements are droppable.

For this tutorial, all list items elements should be draggable and droppable.

By default:

- Browsers allow dragging only with text selections, images, and links.
  
  To allow dragging with other elements, we should set their <a href="https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/draggable" target="_blank">`draggable`</a> attribute.

- Browsers also don't allow most elements to be droppable, because most areas of a webpage are not valid places to drop anything. So, the default handling is to not allow drops.
  
  To allow elements to become drop zones, we should prevent the default handling while we drag over them.

---

**<u>TASK</u>**

**Make all list item elements draggable.**

---

In the HTML file, add `draggable="true"` to list item elements.

{% highlight html %}
<li draggable="true">
{% endhighlight %}

#### Events

A DND operation, from start to end, consists of several event types. Some fire only once, and some continuously.

1. When you start dragging an element, the `dragstart` event is fired first - once.

2. While you drag the element, the `drag` event is fired continuously - every few 100 ms.

3. When you drag into an element, the `dragenter` event is fired first - once.

4. While you drag over an element, the `dragover` event is fired continuously - every few 100 ms.

5. When you drag out of an element, the `dragleave` event is fired - once.

    As you drag out of an element and into another element, the `dragenter` event will fire again - once. As you drag over it, the `dragover` event is fired continuously. And so on.

6. When you release the mouse, if it's on a valid drop target, the `drop` event is fired - once.

7. Then, regardless of whether the target is valid, the `dragend` event is fired - once.

For more information, <a href="https://developer.mozilla.org/en-US/docs/Web/API/DragEvent#event_types" target="_blank">_view the MDN docs about drag event types_</a>.

In this tutorial, we'll use only `dragstart`, `dragover` and `drop`, but feel free to explore further to add more functionality.

Now that we understand the flow of a DND operation from start to end, let's start implementing it into the list.

### Implementation

#### Setup

If you haven't followed the steps in the HTML and CSS sections above, download the necessary HTML and CSS files from here.

---

**<u>TASK</u>**

**Create a JS file named `index.js` as the entry point of the JS code, and link to it in the HTML file.**

---

Create a JS file named `index.js`.

Add the script tag before the closing body tag in the HTML file.

{% highlight html %}
  <!-- ... -->
  <script src="index.js"></script>
</body>
<!-- ... -->
{% endhighlight %}

#### Initialisation

To detect any DND operation on the list items, we should attach DND event listeners to them. However, we want the code to run only after the window has fully loaded.

Let's use a promise for that.

---

**<u>TASK</u>**

**In `index.js`, create a promise that resolves after the window has loaded, and calls a function `init`.**

---

In `index.js`, add the following code.

{% highlight js %}
function init() {
  //code to initialise DND event listeners
}

let windowLoad = new Promise(function(resolve) {
  window.addEventListener('load', resolve);
});

windowLoad.then(
  function(result) {
    init();
  }
);
{% endhighlight %}

`init` will initialise the DND event listeners, but, first, let's look at how we want to structure the DND implementation.

#### Class

For better structure, the DND implementation will be in a class of its own.

---

**<u>TASK</u>**

**Create a JS file named `DND.js` as the class file of the DND implementation, and link to it in the HTML file.**

---

Create a JS file named `DND.js`.

Add the corresponding script tag in the HTML file.

{% highlight html %}
  <!-- ... -->
  <script src="index.js"></script>
  <script src="DND.js"></script>
</body>
<!-- ... -->
{% endhighlight %}

**Note:** Adding several JS script tags in the HTML is not ideal, because loading too many scripts can cause a network bottleneck. It is best to use a module bundler, like Webpack. In this tutorial, we include each script individually for the sake of simplicity.

---

**<u>TASK</u>**

**In `DND.js`, create a class `DND` for the DND implementation.**

**In `index.js`, in `init`, create an object using that class.**

---

In `DND.js`, create a class named `DND`.

{% highlight js %}
class DND {
  //code for DND implementation
}
{% endhighlight %}

In `index.js`, in the function `init`, create a `DND` object using that class.

{% highlight js %}
function init() {
  let dnd = new DND();  
}
{% endhighlight %}

#### Event Listeners

The first thing that we want to do is to set the DND event listeners.

---

**<u>TASK</u>**

**In the `DND` class, create a method `setEventListeners`.**

**In  `index.js`, in `init`, call the method on the `DND` object that you created earlier.**

---

In the `DND` class, create a method called `setEventListeners`, which will take care of setting the DND event listeners.

{% highlight js %}
class DND {

  setEventListeners() {
    //code to set event listeners for DND operation
  }

}
{% endhighlight %}

In `index.js`, call the method `setEventListeners` on the `DND` object that you created.

{% highlight js %}
function init() {
  let dnd = new DND();
  dnd.setEventListeners();  
}
{% endhighlight %}

Next: we want to attach the DND event listeners to all list item elements, remember?

---

**<u>TASK</u>**

**In `setEventListeners`, add event listeners for `dragstart`, `dragover` and `drop` to all list item elements.**

**Name the listener functions `start`, `over` and `drop`, respectively.**

**The listener functions should be set up as class methods.**

---

- Get all the list item elements from the HTML, using `querySelectorAll`, and save them to a variable, `listItems`.

- Since we want to attach the event listeners to all of them, let's loop through them, using `forEach`.

- For each of them, we want to attach:

  1. An event listener for `dragstart`.
  2. An event listener for `dragover`.
  3. An event listener for `drop`.
<br><br>
- Each of these events will have their own listener functions. Let's name those listener functions as `start`, `over`, and `drop`.

{% highlight js %}
setEventListeners() {

  let listItems = document.querySelectorAll('li');
  
  listItems.forEach((listItem) => {

    listItem.addEventListener('dragstart', this.start);
    listItem.addEventListener('dragover', this.over);
    listItem.addEventListener('drop', this.drop);

  });

}
{% endhighlight %}

The listener functions will be methods of the `DND` class, hence the use of `this` to call them.

In your `DND` class, below the `setEventListeners` method, create the method for each listener function.

{% highlight js %}
setEventListeners() {
  //...
}

start() {
  //code to run for dragstart event
}

over() {
  //code to run for dragover event
}

drop() {
  //code to run for drop event
}
{% endhighlight %}

The structure of the `DND` class is now set up.

### Drag Data

Let's take a moment to think again about the goal.

We want to reorder the list by dragging a row and dropping it into the desired position.

To be more specific:

- We want to drag a list item element from a source row to a destination row.

- At the destination row, we want to drop the list item element there.

- Between the source row and the destination row, every time we drag over a new row, we want that row to shift up/down depending on the direction in which we are dragging.

_**Before we proceed further, let's ask ourselves a few questions.**_

**<u>1. When we reach the destination row, how do we know what data to place in there?</u>**

We need to store the data from the source row.

For this, the HTML Drag and Drop API provides us with a <a href="https://developer.mozilla.org/en-US/docs/Web/API/DataTransfer" target="_blank">`DataTransfer`</a> object. This object can be used to store the data involved in a DND operation, a.k.a. the drag data.

This data can then be accessed during DND events throughout the DND operation.

**<u>2. What use do we have of this?</u>**

During `dragstart`, we'll store the data from the source row.

During `drop`, we'll place that data into the destination row.

**<u>3. What data do we want to store?</u>**

The text and the checkbox status of the list item element being dragged.

**<u>4. How do we access the drag data?</u>**

DND events have a property called <a href="https://developer.mozilla.org/en-US/docs/Web/API/DragEvent/dataTransfer" target="_blank">`dataTransfer`</a>, which is a `DataTransfer` object.

- To write data to it, we use the method `setData`.

  ```
  event.dataTransfer.setData(format, data)
  ```

- To read data from it, we use the method `getData`.

  ```
  event.dataTransfer.getData(format);
  ```

For more information, <a href="https://developer.mozilla.org/en-US/docs/Web/API/DataTransfer#methods" target="_blank">_view the MDN docs about the DataTransfer methods_</a>.

**_A note about the drag data:_**

The drag data can only be accessed during `dragstart` and `drop`. This means that:

- During `dragstart`, you can _write_ and _read_ drag data.
- During `drop`, you can _only read_ drag data.
- During all other DND events, you _cannot read nor write_ drag data.

If you want to access drag data outside of `dragstart` and `drop`, you will need a separate variable to store the drag data. We will see this later on in the tutorial.

Now that we better understand the workings of a DND operation - along with the drag data store - let's plan the logic for the DND listeners, before we write any of their code.

### Logic for Event Listeners

**1. At first, during `dragstart`**

**GOAL: Set drag data to values of text and checkbox status.**

- Get text and checkbox status of list item in source row.

  - Get the text from the paragaph located inside the event target.

  - Get the checkbox status from the checkbox located inside the event target.

- Set drag data using the `setData` method mentionned earlier.

**2. Next, during `dragover`**

**GOAL: Shift current row being dragged over to previous row which we just finished dragging over.**

For this, we'll need to keep track of which row was the previous one, and which row is the current one. To do so, we'll use 2 variables: `prevRow` and `currRow`.

Each will store a reference to the respective list item element.

**Note:** `prevRow` should be initialised to store a reference to the source row during `dragstart`.

- **When we drag over any row:**

  - Set `currRow` to store a reference to the current list item element being dragged over.

  - Allow drop at this location.

- **When we drag over a different row:**

  - Update the text and checkbox status in the previous row with the text and checkbox status of the current row.

  - Clear the content of the current row, since we have copied its content to the previous row.

  - Update `prevRow` to store a reference to the current row, so that when dragging over the next row, we can repeat the entire process.

This process repeats every time we drag over another row.

**3. Finally, during `drop`**

**GOAL: Update destination row with drag data**

- Get drag data using the `getData` method mentionned earlier.

- Set text and checkbox status of list item in destination row using values saved in the drag data.

The entire logic is planned. Let's get to coding!

### Code for Event Listeners

Now that we know that we'll access the `event` object in the listeners, let's modify the listener calls and declarations accordingly.

{% highlight js %}
setEventListeners() {
  //...
  listItems.forEach((listItem) => {

    listItem.addEventListener('dragstart', (e) => this.start(e));
    listItem.addEventListener('dragover', (e) => this.over(e));
    listItem.addEventListener('drop', (e) => this.drop(e));

  });

}
{% endhighlight %}

#### Code for `dragstart`

**<u>Step 1: Get the values for the drag data</u>**

- Get the HTML and the checkbox status of the list item element being dragged.

{% highlight js %}
start(e) {
  let HTMLContent = e.target.innerHTML;
  let checkboxStatus = e.target.querySelector('input').checked;
}
{% endhighlight %}

**<u>Step 2: Set the drag data to those values</u>**

For now, let's store the HTML and the checkbox status as separate data pieces inside the `DataTransfer` object.

- We'll call the data types `html-content` and `checkbox-status` respectively.

- The data for each are the values we got in the previous step.

{% highlight js %}
start(e) {
  //...
  e.dataTransfer.setData('html-content', HTMLContent);
  e.dataTransfer.setData('checkbox-status', checkboxStatus);
}
{% endhighlight %}

#### Code for `dragover`

**<u>Step 1: Store a reference to the current list item element being dragged over.</u>**

This could be as simple as `let currRow = e.target;`.

However, as opposed to `dragstart`, `dragover` also fires on the children of the element to which it's attached.

As a result, if the cursor drags over the list item's children elements (like the checkbox, the text, or the icon), the event target actually becomes that child element, instead of the list item element itself.

Wherever the cursor drags over in the row, the intended target is the list item element itself.

Remember our HTML structure for the list element.

{% highlight html %}
<li>
  <div>
    <input type="checkbox">
    <p>List item text</p>
  </div>
  <button class="material-icons">drag_indicator</button>
</li>
{% endhighlight %}

What does this mean?

- When the cursor drags over the button, the intended target is actually its parent.

- When the cursor drags over the checkbox or the text, the intended target is actually the parent of their parent.

Instead of checking which of each element the cursor is dragging over, let's check its parent.

{% highlight js %}
over(e) {
  let currRow;

  if (e.target.parentNode.tagName === 'LI') currRow = e.target.parentNode;
  else if (e.target.parentNode.tagName === 'DIV') currRow = e.target.parentNode.parentNode;
  else currRow = e.target;
}
{% endhighlight %}

**<u>Step 2: Allow drop at this location.</u>**

Prevent the default handling at the row being dragged over, which is to not allow drops.

{% highlight js %}
over(e) {
  //...
  e.preventDefault();
}
{% endhighlight %}

**<u>Step 3: Update the text and the checkbox status in the previous row with the text and the checkbox status of the current row.</u>**

In addition to the current row, we said that we need to keep track of the previous row, and that this variable, `prevRow`, will be initialised during `dragstart`.

Therefore, let's set `prevRow` as a property in the `DND` class. Create a constructor and declare `prevRow`.

{% highlight js %}
class DND {
  constructor() {
    this.prevRow;
  }

  //...
}
{% endhighlight %}

Then, in `start`, initialise `prevRow` to store a reference to the source row, which is the event target.

{% highlight js %}
start(e) {
  this.prevRow = e.target;
  //...    
}
{% endhighlight %}

Now, in `over`, whenever we drag over a different row, we can update the content of the previous row with the content of the current row.

{% highlight js %}
over(e) {
  //...
  if (this.prevRow !== currRow) {
    this.prevRow.innerHTML = currRow.innerHTML;
    this.prevRow.querySelector('input').checked = currRow.querySelector('input').checked;
  }
}
{% endhighlight %}

**Note:** we set the `checked` attribute of the checkbox separately because the attribute is not part of the innerHTML.

<details>
  <summary><strong>TRY IT OUT!</strong></summary>

  <br>
  Drag a row over any other row, and see the content of the source row change to the content of the row being dragged over.
  <hr>
</details>
<br>
**<u>Steps 5 & 6: Clear the content of the current row, and update the previous row reference to the current row.</u>**

We still need to clear the content of the current row, and update `prevRow` to store a reference to the current row.

{% highlight js %}
over(e) {
  //...    
  if (this.prevRow !== currRow) {
    //...
    currRow.innerHTML = '';
    this.prevRow = currRow;
  }
}
{% endhighlight %}

<details>
  <summary><strong>TRY IT OUT!</strong></summary>

  <br>
  Drag a row over any other row and see the latter shift up/down.
  <hr>
</details>

#### Code for `drop`

**<u>Step 1: Get the drag data</u>**

Get the text and the checkbox status of the list item element being dragged, from the `DataTransfer` object.

{% highlight js %}
drop(e) {
  const HTMLContent = e.dataTransfer.getData('html-content');
  const checkboxStatus = e.dataTransfer.getData('checkbox-status');
}
{% endhighlight %}

**<u>Step 2: Set the HTML and the checkbox status in the destination row to the values in the drag data</u>**

{% highlight js %}
drop(e) {
  //...
  e.target.innerHTML = HTMLContent;
  e.target.querySelector('input').checked = (checkboxStatus === 'true');
}
{% endhighlight %}

**Note:** we compare `checkboxStatus` with `'true'` because the `DataTransfer` object stores data in String format.

<details>
  <summary><strong>TRY IT OUT!</strong></summary>

  <br>
  Drop a row at any other row and see the latter be updated with the values of the source row.
  <hr>
</details>
<br>
<a href="https://github.com/AkashaRojee/list-drag-and-drop-tutorial/blob/js/index.html" target="_blank">View the HTML file in its current state.</a>

<a href="https://github.com/AkashaRojee/list-drag-and-drop-tutorial/blob/js/index.js" target="_blank">View the `index.js` file in its current state.</a>

<a href="https://github.com/AkashaRojee/list-drag-and-drop-tutorial/blob/js/DND.js" target="_blank">View the `DND.js` file in its current state.</a>

## Demo & Source Code

<a href="https://www.akasharojee.codes/list-drag-and-drop-tutorial/" target="_blank">View a demo of the tutorial</a>

<a href="https://github.com/AkashaRojee/list-drag-and-drop-tutorial" target="_blank">View the source code</a>

## Wrapping up DND - for now

This tutorial is meant to explain the basic concepts of the HTML Drag and Drop API and to show how to implement it in a list, including a simple row-shifting mechanism. The functionalities can definitely be extended, and improved!

More code has to be written for the following scenarios. Try them out to see what happens, and try to solve them.

- Drag over at least another row, then drop outside of the list
- Drag over at least another row, then drag outside of the list, then drop at a non-adjacent row
- Drag outside of the list immediately, then drop inside the list, but over a child element

In future articles, we'll see how to refactor the existing code, and how to solve the above scenarios.

I hope that this article helped you to get a better overview of the fundamentals of the DND mechanism.

How easy was it to undertand? I would love to know your opinions.

Anything unclear? Ask away!

Implemented drag-and-drop in a project? Share the link in the comments below!

Cheers and happy coding!

## References

* <a href="https://html.spec.whatwg.org/multipage/dnd.html#the-drag-data-store" target="_blank">HTML Standard - Drag and Drop</a>