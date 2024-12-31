---
title : 'The second journey to React'
date : 2024-12-31T09:53:14+08:00
layout: "post"
images: 
  - "/avatar.jpg"   # 设置封面图
description: "Learn react reactively."     
categories: ["技术"]
tags: ["前端"]
draft: false
---
## The Static Pages
<!-- ![](/avtar.jpg) -->

The Fundamental for the first Project 

- composable
- Declarative
- createRoot().render()
- the `<h1>Header</h1>` in the jsx is a object    

```jsx
Object {type: 'h1', key: null, ref: null, props: {children: 'Header'}}
```

## Props

- React Can's render the Object exclude the HTML element object. So use  the **array.map()** to return a components array use the props raw data.

### uniqueKey

Must pass a key named `key` as the unique props.

### **Three different way to pass the object prop**

```jsx
const entryElements = data.map((m, index) => {
    <Entry 
        key={m.id} // key=index
        img=d.img
        />
}) // get a array of components
```

### Pass down the entire Object as the props

```jsx
const entryElements = data.map((m, index) => {
    <Entry 
        key={m.id} // key=index
        entry={m}
        />
}) // get a array of components
```

### Use the spread object  method to pass props

```jsx
const entryElements = data.map((entry, index) => {
    <Entry 
        key={entry.id} // key=index
        {...entry} // convert the object's key to single props
        />
}) // get a array of components
```

## State(Dynamic web application)

### FormData

```jsx
// The way to handle the submit action in the react 19
<form action={handleSubmit} >
    <input defaultVlue="1490910337@main.com" type="email"/>
</form>
const handleSubmit = (formData) => {
    const data  = formData.get(data)
}
```

**The above code will do those thing automatically**

- `preventDefault` prevent the default behavior for the form submission
- `form.reset()`
- Passing the formData object directly
- Make this become more declarative

**radio type input and the checkbox**

```jsx
const getData = (formData) => {
    // get all the checkbox value
    formData.getAll('checkBoxValue')
    // when deal with the mutible value like the checbox and the select option, need set all the options a distinct value from each other
}
<label htmlFor="favColor">What is your favorite color?</label>
        <select id="favColor" name="favColor" defaultValue="" required>
          <option value="" disabled>-- Choose a color --</option>
</select>

```

**Use the `Object.fromEntries` to get the whole formdata**

```jsx
function signUp(formData) {
    console.log(Object.fromEntries(formData))
    // also it still needs to getAll to get the whole checkbox array value
    formData.getAll('checkboxName')
  }
```

### The DataFlow in the React

**Passing Data to component**: Just like the tree, there are sibling components and the parent components,. And in React **DataFlow can only pass the data from top to down (parent to children)**. So there should be the third party like **Redux** or **Context**, to pass the data around the react.

### Use the callback function to control the Components' data flow

Restore the main source in on parent  rather than pass the `pad.id` to the children components.

- React state can be a good maintainer to storage the information between others' state change, that the information won't disappear when the page re-render

## (Out)Side Effect

- Controlled components
- Functional programming 
  - Immutable
  - Pure Function
- (out)side effects

> - Those thing that react does not Control on it's own.
> - To escape from the React ecosystem, need to interactive with outside world.

``` js
useEffect(setup(callback function), dependencies?) // only after the DOM was rendered, then the callback run           
```

By **Default** the callback function  will run every single(so well run at the first time) time when the dom is rerendered , and the `dependecies` will be a **state array**, **only when the dependency change,** **the function run.**

## `useEffect` cleanup function

Where is a UseEffect callback function, React will automatically receive a function called "cleanUp function"

```jsx
useEffect(() => {
   .....
   return function() {
       // this is the useEffect clean up function when this component is unmounted
   }
})
```

## UseRef()

> Ref can be mutated directly
>
> Changing them doesn't cause a re-render

```jsx
const recipeSelection = useRef(null)
// ref can only add to the normal html elements
<input ref={recipeSelection} />
    recipeSelection = Object{current: null}
```



### Controlled Components

### CSS

- padding-block, margin-block can edit the bottom and top 
- em -> 1em = parent'font size (parent = 16px, 1em = 16px) 
- 1rem -> stands for 'root em' root element's(the <html>) size as the standard
- to edit the bullet of the li use the typically pseudo element `li::marker`

```css
/* the pseudo selector for the bullet before the li's context  */
.content-item::marker {
  color: #61DAFB;
  /* rem stands for the root em */
  font-size: 1.5rem;
}
```

- box-sizing
  - border-box: width = border + padding
  - content-box(default): width + border + padding = WIDTH
- flex
  - align-items: center (make the box's element align by center)
- 100% = parent's elements' size
- **Add the background-image**

```css
  background-image: url("./assets/react-logo-half.png");
  background-repeat: no-repeat;
  background-position: right;
  background-position: (x, y));
```

## FlexBox

FlexBox should neet a container to wrapper the whole items become the parent

```css
justify-content: sapce-between : // use the whole availiable space put it in the middle
- flex-start(default)
- flex-end
- center
- space-around (equal space)
flex-direction: column / row(default)
align-items(decide the vertical): (used in the container(parent)) stretch(defalt)
align-selfs: (used in the container's children)
```

- flex = flex-grow flex-shrink flex-basis
- flex = 1 (width = allWidth / itemNumbers)

### flex-direction 

### flex-wrap: no(default) 

Only allowed one row 

### flex

flex = 1 == flex shrink  = flex-grow = 1, flex-basis = 0 = flex = 1 1 0

flexbasis: a way to set the base width of the flex children

flexgrow: element will grow while the window's size bigger

flexshrink: the opposite from the flex grow

### source order



