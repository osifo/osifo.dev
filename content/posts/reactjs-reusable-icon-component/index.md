---
title: "Building A Reusable SVG-Based Icon Component In React"
date: 2017-06-19T01:40:34+02:00
draft: false # Set 'false' to publish
tableOfContents: true # Enable/disable Table of Contents
description: ''
categories:
  - Web Development
tags:
  - reactjs
  - frontend-dev
---


A few months ago, I was working on one of my side projects which i was building in React. I needed to implement icons - I didn't want to use images. I would have attempted using CSS3, but again i wasn't as knowledgeable, considering the complexity involved in crafting the kinds i needed. Then the thought came to mind: SVG!.

I followed the thought for the following reasons
- I wouldn't need to keep various versions of an icon for different screen sizes, as SVG naturally scales.
- SVGs are mostly made up of paths, I figured i could re-use them.
- Most of the configurations for SVG are passed as attributes to the comprising elements, thus if used it'd allow me easily configure the icons.


Below, I describe the implementation steps:
<!--more-->

### 1. Export SVG path(s) for each icon as an array of Strings.
{% highlight text %}
//constants.js

export const ICON_ARROW_DOWN = ['M316 334l196 196 196-196 60 60-256 256-256-256z'];

export const ICON_CANCEL = ['M69.7 68.3c.925.927-.458 2.343-1.4 1.4L50 51.4 31.7 69.7c-.938.938-2.326-.474-1.4-1.4L48.6 50 30.3 31.7c-.932-.933.468-2.332 1.4-1.4L50 48.6l18.3-18.3c.933-.932 2.331.467 1.4 1.4L51.4 50l18.3 18.3z'];

{% endhighlight %}
- *Note: For more complex Icons, the paths array would usually consist of more than one element.*

### 2. We begin building the Icon Component.
For easily reusability, we'll build our Icon component as a stateless(dumb) react component.
The purpose is to allow for more configurability(via props) from its host component.
{% highlight text %}
//Icon.js

import React, { PropTypes } from 'react';

const Icon = (props) => {
  //1. map values in props to icon attributes
  //2. build and return an svg element using the attributes gotten from the passed props.
}

Icon.propTypes = {
  // Sspecify the data-types of props this component is expecting.
}

Icon.defaultProps = {
  //as the name implies the Icon component would hold default value for some important props.
}

//return Icon as the default export from this file
export default Icon;

{% endhighlight %}


### 3. Specify proptypes for the Icon component
We'll start by specifying the data-types of the props we're expecting.
{% highlight text %}

Icon.propTypes = {
  paths: PropTypes.array.isRequired,
  color: PropTypes.string,
  size: PropTypes.number,
  viewBox: PropTypes.number,
  scaleTo: PropTypes.number,
};

{% endhighlight %}

As you would have noticed, the only required props is `paths`. For other essential attributes, we'll fall back to default values in case no values are specified.

{% highlight text %}

Icon.defaultProps = {
  size: 24,
  viewBox: 1024,
  color: '#555',
};

{% endhighlight %}


Here's what our Icon.js file should look like:
{% highlight text %}
//Icon.js

import React, { PropTypes } from 'react';

const Icon = (props) => {

};

Icon.propTypes = {
  paths: PropTypes.array.isRequired,
  color: PropTypes.string,
  size: PropTypes.number,
  viewBox: PropTypes.number,
  scaleTo: PropTypes.number,
};

Icon.defaultProps = {
  size: 24,
  viewBox: 1024,
  color: '#555',
};

export default Icon;
{% endhighlight %}


### 4. Map values in props to Icon attributes.
While SVGs can be used to achieve reasonable complex vectors, for some apps(and for my use-case), SVG-based Icons usually take a shape not too far from this:

{% highlight text %}
<svg class="icon" height="" width="" viewBox="" style="">
  <g fill="" transform="">
    <path d=""></path>
  </g>
</svg>
{% endhighlight %}

Because our objective for the Icon component is *ease of reusability*, specifying these attributes as variables(set using props) is the way to go.

Some props would be used directly as attributes of the SVG components, In other cases we'll compose the attributes values from passed props:

{% highlight text %}

const Icon = (props) => {
  const styles = {
    svg: {
      display: 'inline-block',
      verticalAlign: 'middle',
    },

    g: {
      fill: props.color,
      scale: props.scaleTo
    }
  }

  return(
    //return the build SVG.
  );
};

{% endhighlight %}


### 5. Building out the SVG structure.
We've specified our props as well as other necessary attributes to be used in our Icon component.
Now let's use those in constructing the SVG.
i.e We'll be putting some flesh to this:

{% highlight text %}
<svg class="icon" height="" width="" viewBox="" style="">
  <g fill="" transform="">
    <path d=""></path>
  </g>
</svg>
{% endhighlight %}

In the Icon.js file, we set the attributes for the SVG element above:

{% highlight text %}
//Icon.js

const Icon = (props) => {
.
.
.

return(
  <svg className='icon' style={ styles.svg } height={`${props.size}px`} width={`${props.size}px`} viewBox={ `0 0 ${ props.viewBox } ${ props.viewBox }` }>
    <g fill={ styles.g.fill } transform={ ( props.scaleTo ? `scale(${ styles.g.scale }, ${ styles.g.scale })` : '') }>
      <path d=""></path>
    </g>
  </svg>
);

};

.
.
.

{% endhighlight %}


### 5. Constructing the SVG paths.
When specifying our props, we defined the paths data type as an array. This is because in many cases, Icon SVGs would consist of several paths.

Following best practice, we wouldn't be writing the path-building logic directly in the JSX, as its multiline and could get really twisted if we do.
We'll wrap that in a function a function that returns the array of paths we need.

Here's what the SVG would look:

{% highlight text %}

<svg className='icon' style={ styles.svg } height={`${props.size}px`} width={`${props.size}px`} viewBox={ `0 0 ${ props.viewBox } ${ props.viewBox }` }>
  <g fill={ styles.g.fill } transform={ ( props.scaleTo ? `scale(${ styles.g.scale }, ${ styles.g.scale })` : '') }>
    { buildSVGPaths(props.paths) }
  </g>
</svg>

{% endhighlight %}


Our path building function - buildSVGPaths - takes as a parameter, the paths array passed in as props to the component. We'll make use of this to draw out the paths that form the icon to be built.

{% highlight text %}

const buildSVGPaths = (iconPaths) => {
  let key = 0;
  return iconPaths.map((path) => {
    key++;
    return (<path key={key} d={path}></path>);
  });
};

{% endhighlight %}

What we just did is really simple:
We map on the metthods argument(an arry) to build out `path` elements, which we return. So this  function would be returning an array of `paths`, which we interpolate into the JSX above to achieve a configurable SVG-based Icon component!

Our completed Icon.js file should look like this:


{% highlight text %}
//Icon.js Completed version.

import React, { PropTypes } from 'react';

const buildSVGPaths = (iconPaths) => {
  let key = 0;
  return iconPaths.map((path) => {
    key++;
    return (<path key={key} d={path}></path>);
  });
};

const Icon = (props) => {
  const styles = {
    svg: {
      display: 'inline-block',
      verticalAlign: 'middle',
    },

    g: {
      fill: props.color,
      scale: props.scaleTo
    }
  }

  return(
    <svg className='icon' style={ styles.svg } height={`${props.size}px`} width={`${props.size}px`} viewBox={ `0 0 ${ props.viewBox } ${ props.viewBox }` }>
      <g fill={ styles.g.fill } transform={ ( props.scaleTo ? `scale(${ styles.g.scale }, ${ styles.g.scale })` : '') }>
        { buildSVGPaths(props.icon) }
      </g>
    </svg>
  );
};

Icon.propTypes = {
  icon: PropTypes.array.isRequired,
  color: PropTypes.string,
  size: PropTypes.number,
  viewBox: PropTypes.number,
  scaleTo: PropTypes.number,
};

Icon.defaultProps = {
  size: 24,
  viewBox: 1024,
  color: '#555',
};

export default Icon;

{% endhighlight %}


### Using the Icon component we just built.

The icon component can now easily be imported into any client code and used thus:

{% highlight text %}
//some other component
import Icon from '<path_to_icon_component>';
import { ICON_FOLLOWERS } from '<path_to_constants>';

.
.
.

<Icon icon={ ICON_FOLLOWERS } size={ 36 } color='#ffa600' scaleTo={ 6 } viewBox={ 1148 } } />

.
.
.

{% endhighlight %}

There are free SVG icons sets on the internet for use, however some of the really good ones require attribution. **Its important we adhere to these**.

While this solved the icon challenges i had on my project, i'm certain its not a silver bullet. I'd appreciate if you do share your thoughts in the comments section below.

In the next few days, I would be packaging and publishing this component to npm. I hope it finds good use.



### Additional Resources:
Some really great icon sets:
* [Flaticon](http://www.flaticon.com)
* [The Icomoon.io App](https://icomoon.io/app/)

Another great article on building icons:
* [Icons as React Components](https://medium.com/@david.gilbertson/icons-as-react-components-de3e33cb8792)

ReactJS Resources
* [JSX In depth](https://facebook.github.io/react/docs/jsx-in-depth.html)
* [Typechecking With PropTypes](https://facebook.github.io/react/docs/typechecking-with-proptypes.html)