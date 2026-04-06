---
title: "Make custom marker displaying users location and direction with react-native-maps on iOS"
draft: false
date: "2020-09-30T00:00:00+00:00"
author: juhana.jauhiainen
tags: ["react", "react native", "maps"]
description: "If you want to display the current GPS location with your own custom graphics in React Native Maps, you need to create a custom Marker. This is done fairly easily, since the Marker component accepts a View as child."
---

**NOTICE:** This post is about showing a custom location and direction indicator with react-native-maps on **iOS**. This involves a _hack_ which probably isn't needed on Android. On Android rotation works just by adding the `rotation`prop to the Marker and setting it to the current heading. As far as I know this hack is only required for iOS.

If you want to display the current GPS location with your own custom graphics in React Native Maps, you need to create a custom Marker. This is done fairly easily, since the Marker component accepts a View as child.

```jsx
<Marker coordinate={latlng}>
  <View>
    <Airplane fill="black" />
  </View>
</Marker>
```

Here `<Airplane />` is a SVG component created with `react-native-svg`. `latlng` is a object with `latitude`and `longitude` attributes.

```jsx
import React from "react";
import Svg, { Path } from "react-native-svg";

export default function Airplane(props) {
  return (
    <Svg width={30} height={31} viewBox="0 0 305 313" {...props}>
      <Path d="M134.875 19.74c.04-22.771 34.363-22.771 34.34.642v95.563L303 196.354v35.306l-133.144-43.821v71.424l30.813 24.072v27.923l-47.501-14.764-47.501 14.764v-27.923l30.491-24.072v-71.424L3 231.66v-35.306l131.875-80.409V19.74z" />
    </Svg>
  );
}
```

To add show the heading from GPS using the marker we need to do some tricks.

We can rotate the Airplane icon by adding a rotate transform to the enclosing View component.

```jsx
<Marker coordinate={latlng} flat anchor={{ x: 0.5, y: 0.5 }}>
  <View
    style={{
      transform: [{ rotate: `45deg` }],
    }}
  >
    <Airplane fill="black" />
  </View>
</Marker>
```

![Aiplane](/images/airplane-marker.png)

Now the Marker will always point at 45 degree angle to the upper right of the screen. To rotate the marker to the current GPS heading we first need to add a Geolocation watcher.

```jsx
const [geolocation, setGeolocation] = React.useState({
  latitude: 0,
  longitude: 0,
  altitude: 0,
  heading: 0,
  speed: 0,
});

React.useEffect(() => {
  const watchId = Geolocation.watchPosition((position) => {
    setGeolocation(position.coords);
    updateCameraHeading();
  });

  return () => Geolocation.clearWatch(watchId);
}, []);
```

We add a Geolocation watcher in a useEffect call and whenever we get a new position, we update a state variable which is used to store the current location and heading.

Now we can set the rotation on the Marker using.

```jsx
<Marker coordinate={latlng} flat anchor={{ x: 0.5, y: 0.5 }}>
  <View
    style={{
      transform: [{ rotate: `${geolocation.heading}deg` }],
    }}
  >
    <Airplane fill="black" />
  </View>
</Marker>
```

Now the airplane icon is pointing at the correct direction.

**This will be enough if the map rotation is locked so up is always pointing to north. If the map can be rotated however, the Marker will again point to a wrong direction** 😞

But this can be fixed by taking to account the maps rotation!

To get the maps rotation we need to call `getCamera`on the `MapView` component.

```jsx
const mapRef = React.useRef();
const [cameraHeading, setCameraHeading] = React.useState(0);

function updateCameraHeading() {
  const map = mapRef.current;
  map.getCamera().then((info: Camera) => {
    setCameraHeading(info.heading);
  });
}

<MapView
  ref={mapRef}
  onTouchEnd={() => {
    updateCameraHeading();
  }}
  onTouchCancel={() => {
    updateCameraHeading();
  }}
  onTouchStart={() => {
    updateCameraHeading();
  }}
  onTouchMove={() => {
    updateCameraHeading();
  }}
>
  ...
</MapView>;
```

First, we need a [ref](https://reactjs.org/docs/refs-and-the-dom.html) to the `MapView` component, then we need to call `getCamera` every time the user touches the map and save the camera heading into a state variable.

Finally we calculate the Marker rotation using `currentHeading`and the GPS heading.

```jsx
<Marker coordinate={latlng} flat anchor={{ x: 0.5, y: 0.5 }}>
  <View
    style={{
      transform: [{ rotate: `${geolocation.heading - cameraHeading}deg` }],
    }}
  >
    <Airplane fill="black" />
  </View>
</Marker>
```

Now the Marker will be pointing in the right direction! This isn't an ideal solution because the Marker only updates its rotation after the user stops rotating the map 😕But it works just fine for most situations.
