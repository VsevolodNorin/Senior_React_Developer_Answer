# Senior React Developer Answer

## Configuration and example analysis
There we can clearly see that all the variables, such as card variation, filter panel parameters and pagination type, 
are set in stone just before our component is loaded, so they won’t randomly change at runtime.

It also means that all cards are of the same type at the given moment, so the grid layout won't be overly complicated.

It’s worth mentioning that YourReactComponent is not a component in terms of react, but it surely is in application terms.
We shouldn’t face any problem mounting it in the given placeholder because it’s the basic operation in react-dom, it comes out of the box.

Data endpoint appears static, as a json file, there are no params in it, 
so I would assume that all the data loads once and for all, and any filter/sort operations happen post-load on the front-end side.

Layout can be easily separated to two semantic blocks: controls and content. By controls I mean any filter/sort/pagination options.
Visually there are clearly the header, sidebar, and content zone.

Content zone doesn't change much depending on viewport size.
There is no deep-linking in the example, but some localstorage bound list of favourite cards. Header and sidebar are not sticky and just slides up with the entire page when you scroll. 

Page is quite simple and doesn't provide any complex challenges layout-wise except pop-up fullscreen filters in mobile version.

## Project structure

There is no routing here in example, so we won't group components route-wise. I'd suggest some simple structure without any deep folder nesting because it leads to weird relative imports and this way provides no benefits in such a simple page. The example directory structure could be like this:  
![Directory structure](https://i.imgur.com/DBw87LL.png)  
> Please keep in mind that the image above contains only significant components, additional components could be placed here as needed.

High-level component responsibilities:

- CollectionPage contains all the components specific for this and only this page
  - Card is responsible for rendering single card
  - ContentLayout is a grid of cards
  - DataLoader is a component that loads JSON data and stores it inside a local state. Ideally it should provide the data to child components using props, more on that later
  - PageLayout is a layout top level component that holds everything else inside
  - SearchBar is responsible for search by text functionality
  - SortingSelector is responsible for top-right sorting dropdown
  - TagFilter is a component for handling tag checkboxes
- Reusable is just all the dumb components that could be reused later. Ideally they should live somewhere in the different package, but not in this trivial case
 
In the root folder I would place index.ts (js), that would register our "component" in the global namespace through `window` (given it will not be used as a commonjs/es module, but instead just a script tag as provided in the example).
 
## Coding style and technical solutions
 
Given there are no pre-existing constraints regarding react version or existing code, I would prefer to use mostly functional components with hooks over HOC or class components, because they tend to be less verbose and less complicated. I don't see any problems here that couldn't be resolved using hooks (Error boundary could be the one though, but error handling isn't mentioned anywhere in the example). We could use Render Props to provide the data, but the simple props-way is way more easy to maintain and optimise for me.

The next problem is the possibility to add more variations later. There is nothing difficult here, but the code can easily become messy if we were using a lot of conditions and switches in the markup, so I'd suggest to wrap variations in switcher components, in order to  achieve something like pseudocode below:
```tsx
  <Pager config={config} ... any other common pager props .../>
  ...
  const Pager = (props: PagerProps) => {
    switch(props.config.type) {
      case 'loadmore': 
        return (<LoadMorePager {...props}/>);
      case 'anotherpager':
        return (<AnotherPager {...props}/>);
      default:
        throw new Error('Invalid pager type');
    }
  };
```
It should simplify layout pages enough to not care about what variation of component should be rendered here and there.
Also we could use the very same technique to provide some component variations that differ based on viewport size (using media query api), like checkbox filter panel/dropdown (only for those that need more than just a simple css change).

Also separating those variations in different components makes (unit) testing much easier.

I see no need for any local state managing libraries here. It could be much easier (and implemented much faster) to just store filters in the page local state, or refactor it to use react context api to handle deep props-passing than to put entire redux in the bundle just for 2 or 3 values.   
We could easily use [recoil](https://recoiljs.org/) here, though.

Page layout could be easily made using css grids provided we're not expected to support IE11. This solution gives us the flexibility to easily change page layout based on viewport using [named areas](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Grid_Layout/Grid_Template_Areas).

There is no different how we'd handle our styles, but I'd suggest CSS-IN-JS way (especially [styled-components](https://styled-components.com/) because they allow the developer to write very readable react code without the need to wrap every div to its own component).

And some words about testing: I'm [not a big fan](joshribakoff.com/jest-snapshot-testing-considered-harmful/) of snapshot testing, we could easily test our components behaviour by checking if they render all the information they should, and don't render anything they should not. It should give us some regression protection with a low cost. Also we could use cypress to do some e2e-testing later.

