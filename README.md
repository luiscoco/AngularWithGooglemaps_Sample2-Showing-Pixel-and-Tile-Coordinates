# Angular with GoogleMaps integration.

Here's an Angular application that integrates the provided TypeScript code to show a simple Google Maps:

1.Create a new Angular blank application.
```
ng new googlemaps_simplemap
```

2.Install the required dependencies:
```
npm install @types/googlemaps
```

3.Create a new Angular component. Let's call it MapComponent:
```
ng generate component Map
```

4.Open the generated map.component.ts file and replace the content with the following code:
```typescript
import { AfterViewInit, Component } from '@angular/core';
declare var google: any;

const TILE_SIZE = 256; // Add this line

@Component({
  selector: 'app-map',
  template: `<div id="map"></div>`,
  styles: [`#map { height: 400px; }`],
})
export class MapComponent implements AfterViewInit {
  ngAfterViewInit() {
    this.initMap();
  }

  initMap(): void {
    const chicago = new google.maps.LatLng(41.85, -87.65);

    const map = new google.maps.Map(document.getElementById('map'), {
      center: chicago,
      zoom: 3,
    });

    const coordInfoWindow = new google.maps.InfoWindow();

    coordInfoWindow.setContent(
      this.createInfoWindowContent(chicago, map.getZoom())
    );
    coordInfoWindow.setPosition(chicago);
    coordInfoWindow.open(map);

    map.addListener('zoom_changed', () => {
      coordInfoWindow.setContent(
        this.createInfoWindowContent(chicago, map.getZoom())
      );
      coordInfoWindow.open(map);
    });
  }

  createInfoWindowContent(latLng: google.maps.LatLng, zoom: number) {
    const scale = 1 << zoom;

    const worldCoordinate = this.project(latLng);

    const pixelCoordinate = new google.maps.Point(
      Math.floor(worldCoordinate.x * scale),
      Math.floor(worldCoordinate.y * scale)
    );

    const tileCoordinate = new google.maps.Point(
      Math.floor((worldCoordinate.x * scale) / TILE_SIZE),
      Math.floor((worldCoordinate.y * scale) / TILE_SIZE)
    );

    return [
      'Chicago, IL',
      'LatLng: ' + latLng,
      'Zoom level: ' + zoom,
      'World Coordinate: ' + worldCoordinate,
      'Pixel Coordinate: ' + pixelCoordinate,
      'Tile Coordinate: ' + tileCoordinate,
    ].join('<br>');
  }

  project(latLng: google.maps.LatLng) {
    let siny = Math.sin((latLng.lat() * Math.PI) / 180);

    // Truncating to 0.9999 effectively limits latitude to 89.189. This is
    // about a third of a tile past the edge of the world tile.
    siny = Math.min(Math.max(siny, -0.9999), 0.9999);

    return new google.maps.Point(
      TILE_SIZE * (0.5 + latLng.lng() / 360),
      TILE_SIZE *
        (0.5 -
          Math.log((1 + siny) / (1 - siny)) /
          (4 * Math.PI))
    );
  }
}
```

5.Open the app.module.ts file and import the MapComponent:
```typescript
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';

import { AppComponent } from './app.component';
import { MapComponent } from './map/map.component';

@NgModule({
  declarations: [
    AppComponent,
    MapComponent
  ],
  imports: [
    BrowserModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

6.Open the app.component.html file and add the <app-map></app-map> tag:
```html
<h1>My Angular App</h1>
<app-map></app-map>
```

7.Open the tsconfig.app.json file in the root directory of your Angular project.
Add "node_modules/@types" to the "types" array. Your tsconfig.app.json file should look like this:
```json
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "outDir": "./out-tsc/app",
    "types": [
      "node",
      "googlemaps"
    ]
  },
  "files": [
    "src/main.ts",
    "src/polyfills.ts"
  ],
  "include": [
    "src/**/*.d.ts"
  ]
}
```

8.In the root directory of your Angular project, open the angular.json file.
Inside the "build" target, ensure that the "options" property has a "polyfills" property with the value set to "src/polyfills.ts". It should look like this:

```
"options": {
  "polyfills": "src/polyfills.ts",
  ...
},
```

9.Now, create a new file called polyfills.ts in the src folder of your project. Add the following content to the polyfills.ts file:
```typescript
import 'zone.js/dist/zone';  // Included by default in Angular CLI projects
```


10.Confirm that you have added the Google Maps API script correctly in the index.html file. It should be placed just before the closing </body> tag, like this:
```html
<script src="https://maps.googleapis.com/maps/api/js?key=YOUR_API_KEY&libraries=places"></script>
```

11.Start the application with the command:
```
ng serve -o
```
