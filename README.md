# React Native Tab View

[![npm version](https://img.shields.io/npm/v/react-native-tab-view-dgjoy.svg)](https://www.npmjs.com/package/react-native-tab-view-dgjoy)
[![travis](https://img.shields.io/travis/react-native-community/react-native-tab-view-dgjoy.svg)](https://travis-ci.org/react-native-community/react-native-tab-view-dgjoy)
[![license](https://img.shields.io/github/license/react-native-community/react-native-tab-view-dgjoy.svg)](https://opensource.org/licenses/MIT)

A cross-platform Tab View component for React Native.

This is a JavaScript-only implementation of swipeable tab views. It's super customizable, allowing you to do things like coverflow.

- [Run the example app to see it in action](https://expo.io/@satya164/react-native-tab-view-dgjoy-demos).
- Checkout the [example/](https://github.com/react-native-community/react-native-tab-view-dgjoy/tree/master/example) folder for source code.


## Features

- Smooth animations and gestures
- Scrollable tabs
- Both top and bottom tab bars
- Follows Material Design spec
- Highly customizable
- Fully typed with [Flow](https://flow.org/)


## Demo

<a href="https://raw.githubusercontent.com/satya164/react-native-tab-view-dgjoy/master/demo/demo.mp4"><img src="https://raw.githubusercontent.com/satya164/react-native-tab-view-dgjoy/master/demo/demo.gif" width="360"></a>


## Installation

```sh
yarn add react-native-tab-view-dgjoy
```


## Example

```js
import React, { PureComponent } from 'react';
import { View, StyleSheet } from 'react-native';
import { TabViewAnimated, TabBar, SceneMap } from 'react-native-tab-view-dgjoy';

const FirstRoute = () => <View style={[ styles.container, { backgroundColor: '#ff4081' } ]} />;
const SecondRoute = () => <View style={[ styles.container, { backgroundColor: '#673ab7' } ]} />;

export default class TabViewExample extends PureComponent {
  state = {
    index: 0,
    routes: [
      { key: '1', title: 'First' },
      { key: '2', title: 'Second' },
    ],
  };

  _handleIndexChange = index => this.setState({ index });

  _renderHeader = props => <TabBar {...props} />;

  _renderScene = SceneMap({
    '1': FirstRoute,
    '2': SecondRoute,
  });

  render() {
    return (
      <TabViewAnimated
        style={styles.container}
        navigationState={this.state}
        renderScene={this._renderScene}
        renderHeader={this._renderHeader}
        onIndexChange={this._handleIndexChange}
      />
    );
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
});
```


## API

The package exposes the following components,

### `<TabViewAnimated />`

Container component responsible for managing tab transitions.

#### Props

- `navigationState` - the current navigation state
- `onIndexChange` - callback for when the current tab index changes, should do the `setState`
- `onPositionChange` - callback called with position value as it changes (e.g. - on swipe or tab change), avoid doing anything expensive here
- `canJumpToTab` - optional callback which accepts a route, and returns a boolean indicating whether jumping to the tab is allowed
- `lazy` - whether to load tabs lazily when you start switching
- `initialLayout` - optional object containing the initial `height` and `width`, can be passed to prevent the one frame delay in rendering
- `renderPager` - optional callback which returns a react element to handle swipe gesture and animation
- `renderHeader` - optional callback which returns a react element to use as top tab bar
- `renderFooter` - optional callback which returns a react element to use as bottom tab bar
- `renderScene` - callback which returns a react element to use as a scene

Any other props are passed to the underlying pager.

### `<TabViewPagerPan />`

Pager component based on `PanResponder`.

#### Props

- `configureTransition` - optional callback which returns a configuration for the transition
- `animationEnabled` - whether to enable page change animation
- `swipeEnabled` - whether to enable swipe gestures
- `swipeDistanceThreshold` - minimum swipe distance to trigger page switch
- `swipeVelocityThreshold` - minimum swipe velocity to trigger page switch
- `onSwipeStart` - optional callback when a swipe gesture starts
- `onSwipeEnd` - optional callback when a swipe gesture ends
- `children` - React Element(s) to render

### `<TabViewPagerScroll />`

Pager component based on `ScrollView` (default on iOS).

#### Props

- `animationEnabled` - whether to enable page change animation
- `swipeEnabled` - whether to enable swipe gestures
- `children` - React Element(s) to render

### `<TabViewPagerAndroid />`

Pager component based on `ViewPagerAndroid` (default on Android).

#### Props

- `animationEnabled` - whether to enable page change animation
- `swipeEnabled` - whether to enable swipe gestures
- `children` - React Element(s) to render

### `<TabBar />`

Material design themed tab bar. Can be used as both top and bottom tab bar.

#### Props

- `getLabelText` - optional callback which receives the current scene and returns the tab label
- `renderIcon` - optional callback which receives the current scene and returns a React Element to be used as a icon
- `renderLabel` - optional callback which receives the current scene and returns a React Element to be used as a label
- `renderIndicator` - optional callback which receives the current scene and returns a React Element to be used as a tab indicator
- `renderBadge` - optional callback which receives the current scene and returns a React Element to be used as a badge
- `onTabPress` - optional callback invoked on tab press which receives the scene for the pressed tab, useful for things like scroll to top
- `pressColor` - color for material ripple (Android >= 5.0 only)
- `pressOpacity` - opacity for pressed tab (iOS and Android < 5.0 only)
- `scrollEnabled` - whether to enable scrollable tabs
- `tabStyle` - style object for the individual tabs in the tab bar
- `indicatorStyle` - style object for the active indicator
- `labelStyle` - style object for the tab item label
- `style` - style object for the tab bar


Check the [type definitions](src/TabViewTypeDefinitions.js) for details on shape of different objects.


## Caveats

`<TabViewAnimated />` is a `PureComponent` to prevent unnecessary re-rendering. As a side-effect, the tabs won't re-render if something changes in the parent's state/props. If you need it to trigger a re-render, put it in the `navigationState`.

For example, consider you have a `loaded` property on state which should trigger re-render. You can have your state like the following:

```js
state = {
  index: 0,
  routes: [
    { key: '1', title: 'First' },
    { key: '2', title: 'Second' },
  ],
  loaded: false,
}
```

Then pass `this.state` as the `navigationState` prop to `<TabViewAnimated />` or `<TabViewTransitioner />`.

```js
<TabViewAnimated
  navigationState={this.state}
  renderScene={this._renderPage}
  renderHeader={this._renderHeader}
  onIndexChange={this._handleIndexChange}
/>
```


## Optimization Tips

### Avoid unnecessary re-renders

The `renderScene` function is called every time the index changes. If your `renderScene` function is expensive, it's good idea move each route to a separate component if they don't depend on the index, and apply `shouldComponentUpdate` to prevent unnecessary re-renders.

For example, instead of:

```js
renderScene = ({ route }) => {
  switch (route.key) {
  case 'home':
    return (
      <View style={styles.page}>
        <Avatar />
        <NewsFeed />
      </View>
    );
  default:
    return null;
  }
}
```

Do the following:

```js
renderScene = ({ route }) => {
  switch (route.key) {
  case 'home':
    return <HomeComponent />;
  default:
    return null;
  }
}
```

Where `<HomeComponent />` is a `PureComponent`.

If you are using the `SceneMap` helper, the scenes are already optimized with `PureComponent` and won't re-render if parent's state changes.

### Avoid one frame delay

We need to measure the width of the container and hence need to wait before rendering some elements on the screen. If you know the initial width upfront, you can pass it in and we won't need to wait for measuring it. Most of the time, it's just the window width.

For example, pass the following `initialLayout` to `TabViewAnimated`:

```js
const initialLayout = {
  height: 0,
  width: Dimensions.get('window').width,
};
```

The tabview will still react to changes in the dimension and adjust accordingly to accommodate things like orientation change.


### Optimize large number of routes

If you've a large number of routes, especially images, it can slow the animation down a lot. You can instead render a limited number of routes.

For example, do the following to render only 2 routes on each side:

```js
renderScene = ({ route }) => {
  if (Math.abs(this.state.index - this.state.routes.indexOf(route)) > 2) {
    return null;
  }

  return <MySceneComponent route={route} />;
};
```


## Contributing

While developing, you can run the [example app](/example/README.md) to test your changes.

Make sure the tests still pass, and your code passes Flow and ESLint. Run the following to verify:

```sh
yarn test
yarn run flow
yarn run lint
```

To fix formatting errors, run the following:

```
yarn run lint -- --fix
```

Remember to add tests for your change if possible.
