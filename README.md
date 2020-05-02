# membuat-aplikasi-cuaca-1
code make aplication of weather
//manifest 



snapdragon-serious-hummingbird

Show
public/manifest.json
Nice! Your code is now formatted.
{
  "name": "Weather",
  "short_name": "Weather",
  "icons": [
    {
      "src": "/images/icons/icon-128x128.png",
      "sizes": "128x128",
      "type": "image/png"
    },
    {
      "src": "/images/icons/icon-144x144.png",
      "sizes": "144x144",
      "type": "image/png"
    },
    {
      "src": "/images/icons/icon-152x152.png",
      "sizes": "152x152",
      "type": "image/png"
    },
    {
      "src": "/images/icons/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "/images/icons/icon-256x256.png",
      "sizes": "256x256",
      "type": "image/png"
    },
    {
      "src": "/images/icons/icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ],
  "start_url": "/index.html",
  "display": "standalone",
  "background_color": "#3E4EB8",
  "theme_color": "#2F3BA2"
}
​
//apps.js
/*
 * @license
 * Your First PWA Codelab (https://g.co/codelabs/pwa)
 * Copyright 2019 Google Inc. All rights reserved.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     https://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License
 */
'use strict';

const weatherApp = {
  selectedLocations: {},
  addDialogContainer: document.getElementById('addDialogContainer'),
};

/**
 * Toggles the visibility of the add location dialog box.
 */
function toggleAddDialog() {
  weatherApp.addDialogContainer.classList.toggle('visible');
}

/**
 * Event handler for butDialogAdd, adds the selected location to the list.
 */
function addLocation() {
  // Hide the dialog
  toggleAddDialog();
  // Get the selected city
  const select = document.getElementById('selectCityToAdd');
  const selected = select.options[select.selectedIndex];
  const geo = selected.value;
  const label = selected.textContent;
  const location = {label: label, geo: geo};
  // Create a new card & get the weather data from the server
  const card = getForecastCard(location);
  getForecastFromNetwork(geo).then((forecast) => {
    renderForecast(card, forecast);
  });
  // Save the updated list of selected cities.
  weatherApp.selectedLocations[geo] = location;
  saveLocationList(weatherApp.selectedLocations);
}

/**
 * Event handler for .remove-city, removes a location from the list.
 *
 * @param {Event} evt
 */
function removeLocation(evt) {
  const parent = evt.srcElement.parentElement;
  parent.remove();
  if (weatherApp.selectedLocations[parent.id]) {
    delete weatherApp.selectedLocations[parent.id];
    saveLocationList(weatherApp.selectedLocations);
  }
}

/**
 * Renders the forecast data into the card element.
 *
 * @param {Element} card The card element to update.
 * @param {Object} data Weather forecast data to update the element with.
 */
function renderForecast(card, data) {
  if (!data) {
    // There's no data, skip the update.
    return;
  }

  // Find out when the element was last updated.
  const cardLastUpdatedElem = card.querySelector('.card-last-updated');
  const cardLastUpdated = cardLastUpdatedElem.textContent;
  const lastUpdated = parseInt(cardLastUpdated);

  // If the data on the element is newer, skip the update.
  if (lastUpdated >= data.currently.time) {
    return;
  }
  cardLastUpdatedElem.textContent = data.currently.time;

  // Render the forecast data into the card.
  card.querySelector('.description').textContent = data.currently.summary;
  const forecastFrom = luxon.DateTime
      .fromSeconds(data.currently.time)
      .setZone(data.timezone)
      .toFormat('DDDD t');
  card.querySelector('.date').textContent = forecastFrom;
  card.querySelector('.current .icon')
      .className = `icon ${data.currently.icon}`;
  card.querySelector('.current .temperature .value')
      .textContent = Math.round(data.currently.temperature);
  card.querySelector('.current .humidity .value')
      .textContent = Math.round(data.currently.humidity * 100);
  card.querySelector('.current .wind .value')
      .textContent = Math.round(data.currently.windSpeed);
  card.querySelector('.current .wind .direction')
      .textContent = Math.round(data.currently.windBearing);
  const sunrise = luxon.DateTime
      .fromSeconds(data.daily.data[0].sunriseTime)
      .setZone(data.timezone)
      .toFormat('t');
  card.querySelector('.current .sunrise .value').textContent = sunrise;
  const sunset = luxon.DateTime
      .fromSeconds(data.daily.data[0].sunsetTime)
      .setZone(data.timezone)
      .toFormat('t');
  card.querySelector('.current .sunset .value').textContent = sunset;

  // Render the next 7 days.
  const futureTiles = card.querySelectorAll('.future .oneday');
  futureTiles.forEach((tile, index) => {
    const forecast = data.daily.data[index + 1];
    const forecastFor = luxon.DateTime
        .fromSeconds(forecast.time)
        .setZone(data.timezone)
        .toFormat('ccc');
    tile.querySelector('.date').textContent = forecastFor;
    tile.querySelector('.icon').className = `icon ${forecast.icon}`;
    tile.querySelector('.temp-high .value')
        .textContent = Math.round(forecast.temperatureHigh);
    tile.querySelector('.temp-low .value')
        .textContent = Math.round(forecast.temperatureLow);
  });

  // If the loading spinner is still visible, remove it.
  const spinner = card.querySelector('.card-spinner');
  if (spinner) {
    card.removeChild(spinner);
  }
}

/**
 * Get's the latest forecast data from the network.
 *
 * @param {string} coords Location object to.
 * @return {Object} The weather forecast, if the request fails, return null.
 */
function getForecastFromNetwork(coords) {
  return fetch(`/forecast/${coords}`)
      .then((response) => {
        return response.json();
      })
      .catch(() => {
        return null;
      });
}

/**
 * Get's the cached forecast data from the caches object.
 
 *
 * @param {string} coords Location object to.
 * @return {Object} The weather forecast, if the request fails, return null.
 */
function getForecastFromCache(coords) {
  // CODELAB: Add code to get weather forecast from the caches object.
  if (!('caches' in window)) {
  return null;
}
const url = `${window.location.origin}/forecast/${coords}`;
return caches.match(url)
    .then((response) => {
      if (response) {
        return response.json();
      }
      return null;
    })
    .catch((err) => {
      console.error('Error getting data from cache', err);
      return null;
    });


}

/**
 * Get's the HTML element for the weather forecast, or clones the template
 * and adds it to the DOM if we're adding a new item.
 *
 * @param {Object} location Location object
 * @return {Element} The element for the weather forecast.
 */
function getForecastCard(location) {
  const id = location.geo;
  const card = document.getElementById(id);
  if (card) {
    return card;
  }
  const newCard = document.getElementById('weather-template').cloneNode(true);
  newCard.querySelector('.location').textContent = location.label;
  newCard.setAttribute('id', id);
  newCard.querySelector('.remove-city')
      .addEventListener('click', removeLocation);
  document.querySelector('main').appendChild(newCard);
  newCard.removeAttribute('hidden');
  return newCard;
}

/**
 * Gets the latest weather forecast data and updates each card with the
 * new data.
 */
function updateData() {
  Object.keys(weatherApp.selectedLocations).forEach((key) => {
    const location = weatherApp.selectedLocations[key];
    const card = getForecastCard(location);
    // CODELAB: Add code to call getForecastFromCache
    getForecastFromCache(location.geo)
    .then((forecast) => {
      renderForecast(card, forecast);
    });

    // Get the forecast data from the network.
    getForecastFromNetwork(location.geo)
        .then((forecast) => {
          renderForecast(card, forecast);
        });
  });
}

/**
 * Saves the list of locations.
 *
 * @param {Object} locations The list of locations to save.
 */
function saveLocationList(locations) {
  const data = JSON.stringify(locations);
  localStorage.setItem('locationList', data);
}

/**
 * Loads the list of saved location.
 *
 * @return {Array}
 */
function loadLocationList() {
  let locations = localStorage.getItem('locationList');
  if (locations) {
    try {
      locations = JSON.parse(locations);
    } catch (ex) {
      locations = {};
    }
  }
  if (!locations || Object.keys(locations).length === 0) {
    const key = '40.7720232,-73.9732319';
    locations = {};
    locations[key] = {label: 'New York City', geo: '40.7720232,-73.9732319'};
  }
  return locations;
}

/**
 * Initialize the app, gets the list of locations from local storage, then
 * renders the initial data.
 */
function init() {
  // Get the location list, and update the UI.
  weatherApp.selectedLocations = loadLocationList();
  updateData();

  // Set up the event handlers for all of the buttons.
  document.getElementById('butRefresh').addEventListener('click', updateData);
  document.getElementById('butAdd').addEventListener('click', toggleAddDialog);
  document.getElementById('butDialogCancel')
      .addEventListener('click', toggleAddDialog);
  document.getElementById('butDialogAdd')
      .addEventListener('click', addLocation);
}

init();
//install.js
/*
 * @license
 * Your First PWA Codelab (https://g.co/codelabs/pwa)
 * Copyright 2019 Google Inc. All rights reserved.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     https://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License
 */
'use strict';

let deferredInstallPrompt = null;
const installButton = document.getElementById('butInstall');
installButton.addEventListener('click', installPWA);

// CODELAB: Add event listener for beforeinstallprompt event
window.addEventListener('beforeinstallprompt', saveBeforeInstallPromptEvent);


/**
 * Event handler for beforeinstallprompt event.
 *   Saves the event & shows install button.
 *
 * @param {Event} evt
 */
function saveBeforeInstallPromptEvent(evt) {
  // CODELAB: Add code to save event & show the install button.
  deferredInstallPrompt = evt;
installButton.removeAttribute('hidden');

}


/**
 * Event handler for butInstall - Does the PWA installation.
 *
 * @param {Event} evt
 */
function installPWA(evt) {
  // CODELAB: Add code show install prompt & hide the install button.
  deferredInstallPrompt.prompt();
  // Hide the install button, it can't be called twice.
evt.srcElement.setAttribute('hidden', true);

  // CODELAB: Log user response to prompt.
  deferredInstallPrompt.userChoice
    .then((choice) => {
      if (choice.outcome === 'accepted') {
        console.log('User accepted the A2HS prompt', choice);
      } else {
        console.log('User dismissed the A2HS prompt', choice);
      }
      deferredInstallPrompt = null;
    });


}

// CODELAB: Add event listener for appinstalled event
window.addEventListener('appinstalled', logAppInstalled);

/**
 * Event handler for appinstalled event.
 *   Log the installation to analytics or save the event somehow.
 *
 * @param {Event} evt
 */
function logAppInstalled(evt) {
  // CODELAB: Add code to log the event
  console.log('Weather App was installed.', evt);

}
//index.html
<!--
 Your First PWA Codelab (https://g.co/codelabs/pwa)

 Copyright 2019 Google Inc.

 Licensed under the Apache License, Version 2.0 (the "License");
 you may not use this file except in compliance with the License.
 You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

 Unless required by applicable law or agreed to in writing, software
 distributed under the License is distributed on an "AS IS" BASIS,
 WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 See the License for the specific language governing permissions and
 limitations under the License.
-->

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Weather PWA</title>
  <meta name="codelab" content="your-first-pwa-v3">
  <link rel="stylesheet" type="text/css" href="/styles/inline.css">
  <link rel="icon" href="/images/favicon.ico" type="image/x-icon" />

  <!-- CODELAB: Add link rel manifest -->
  <link rel="manifest" href="/manifest.json">
  <!-- CODELAB: Add iOS meta tags and icons -->
  <meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black">
<meta name="apple-mobile-web-app-title" content="Weather PWA">
<link rel="apple-touch-icon" href="/images/icons/icon-152x152.png"> 
  <!-- CODELAB: Add description here -->
  <meta name="description" content="A sample weather app">
  <!-- CODELAB: Add meta theme-color -->
  <meta name="theme-color" content="#2F3BA2" />

</head>
<body>

  <header class="header">
    <h1>
      Weather PWA
      <a href="https://darksky.net/poweredby/" class="powered-by">
        Powered by Dark Sky
      </a>
    </h1>
    <button id="butInstall" aria-label="Install" hidden></button>
    <button id="butRefresh" aria-label="Refresh"></button>
  </header>

  <main class="main">

    <button id="butAdd" class="fab" aria-label="Add">
      <span class="icon add"></span>
    </button>

    <div id="about" class="weather-card">
      <b>Your First Progressive Web App Codelab</b><br>
      Get started at <a href="https://g.co/codelabs/pwa">https://g.co/codelabs/pwa</a>.
    </div>

    <div id="weather-template" class="weather-card" hidden>
      <div class="card-spinner">
        <svg viewBox="0 0 32 32" width="32" height="32">
          <circle cx="16" cy="16" r="14" fill="none"></circle>
        </svg>
      </div>
      <button class="remove-city">&times;</button>
      <div class="city-key" hidden></div>
      <div class="card-last-updated" hidden></div>
      <div class="location">&nbsp;</div>
      <div class="date">&nbsp;</div>
      <div class="description">&nbsp;</div>
      <div class="current">
        <div class="visual">
          <div class="icon"></div>
          <div class="temperature">
            <span class="value"></span><span class="scale">°F</span>
          </div>
        </div>
        <div class="description">
          <div class="humidity">
            <span class="label">Humidity:</span>
            <span class="value"></span><span class="scale">%</span>
          </div>
          <div class="wind">
            <span class="label">Wind:</span>
            <span class="value"></span>
            <span class="scale">mph</span>
            <span class="direction"></span>°
          </div>
          <div class="sunrise">
            <span class="label">Sunrise:</span>
            <span class="value"></span>
          </div>
          <div class="sunset">
              <span class="label">Sunset:</span>
              <span class="value"></span>
            </div>
        </div>
      </div>
      <div class="future">
        <div class="oneday">
          <div class="date"></div>
          <div class="icon"></div>
          <div class="temp-high">
            <span class="value"></span>°
          </div>
          <div class="temp-low">
            <span class="value"></span>°
          </div>
        </div>
        <div class="oneday">
          <div class="date"></div>
          <div class="icon"></div>
          <div class="temp-high">
            <span class="value"></span>°
          </div>
          <div class="temp-low">
            <span class="value"></span>°
          </div>
        </div>
        <div class="oneday">
          <div class="date"></div>
          <div class="icon"></div>
          <div class="temp-high">
            <span class="value"></span>°
          </div>
          <div class="temp-low">
            <span class="value"></span>°
          </div>
        </div>
        <div class="oneday">
          <div class="date"></div>
          <div class="icon"></div>
          <div class="temp-high">
            <span class="value"></span>°
          </div>
          <div class="temp-low">
            <span class="value"></span>°
          </div>
        </div>
        <div class="oneday">
          <div class="date"></div>
          <div class="icon"></div>
          <div class="temp-high">
            <span class="value"></span>°
          </div>
          <div class="temp-low">
            <span class="value"></span>°
          </div>
        </div>
        <div class="oneday">
          <div class="date"></div>
          <div class="icon"></div>
          <div class="temp-high">
            <span class="value"></span>°
          </div>
          <div class="temp-low">
            <span class="value"></span>°
          </div>
        </div>
        <div class="oneday">
          <div class="date"></div>
          <div class="icon"></div>
          <div class="temp-high">
            <span class="value"></span>°
          </div>
          <div class="temp-low">
            <span class="value"></span>°
          </div>
        </div>
      </div>
    </div>
  </main>

  <div id="addDialogContainer">
    <div class="dialog">
      <div class="dialog-title">Add new city</div>
      <div class="dialog-body">
        <select id="selectCityToAdd" aria-label="City to add">
          <!--
            Values are lat/lon values, use Google Maps to find and add
            additional cities.
          -->
          <option value="28.6472799,76.8130727">Dehli, India</option>
          <option value="-5.7759362,106.1174957">Jakarta, Indonesia</option>
          <option value="51.5287718,-0.2416815">London, UK</option>
          <option value="40.6976701,-74.2598666">New York, USA</option>
          <option value="48.8589507,2.2770202">Paris, France</option>
          <option value="-64.8251018,-63.496847">Port Lockroy, Antarctica</option>
          <option value="37.757815,-122.5076401">San Francisco, USA</option>
          <option value="31.2243085,120.9162955">Shanghai, China</option>
          <option value="35.6735408,139.5703032">Tokyo, Japan</option>
        </select>
      </div>
      <div class="dialog-buttons">
        <button id="butDialogCancel" class="button">Cancel</button>
        <button id="butDialogAdd" class="button">Add</button>
      </div>
    </div>
  </div>

  <script src="/scripts/luxon-1.11.4.js"></script>
  <script src="/scripts/app.js"></script>
  <!-- CODELAB: Add the install script here -->
  <script src="/scripts/install.js"></script>
  <script>
    // CODELAB: Register service worker.
    if ('serviceWorker' in navigator) {
  window.addEventListener('load', () => {
    navigator.serviceWorker.register('/service-worker.js')
        .then((reg) => {
          console.log('Service worker registered.', reg);
        });
  });
}
  </script>

</body>
</html>
//offline.js
<!--
 Your First PWA Codelab (https://g.co/codelabs/pwa)

 Copyright 2019 Google Inc.

 Licensed under the Apache License, Version 2.0 (the "License");
 you may not use this file except in compliance with the License.
 You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

 Unless required by applicable law or agreed to in writing, software
 distributed under the License is distributed on an "AS IS" BASIS,
 WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 See the License for the specific language governing permissions and
 limitations under the License.
-->

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Weather PWA</title>
  <meta name="codelab" content="your-first-pwa-v3">
  <link rel="manifest" href="/manifest.json">
  <meta name="theme-color" content="#2F3BA2" />
  <style>
    * {
      box-sizing: border-box;
    }

    html,
    body {
      color: #444;
      font-family: 'Helvetica', 'Verdana', sans-serif;
      -moz-osx-font-smoothing: grayscale;
      -webkit-font-smoothing: antialiased;
      height: 100%;
      margin: 0;
      padding: 0;
      width: 100%;
    }

    html {
      overflow: hidden;
    }

    body {
      align-content: stretch;
      align-items: stretch;
      background: #ececec;
      display: flex;
      flex-direction: column;
      flex-wrap: nowrap;
      justify-content: flex-start;
    }

    /**
    * Header
    */

    .header {
      align-content: center;
      align-items: stretch;
      background: #3f51b5;
      box-shadow:
        0 4px 5px 0 rgba(0, 0, 0, 0.14),
        0 2px 9px 1px rgba(0, 0, 0, 0.12),
        0 4px 2px -2px rgba(0, 0, 0, 0.2);
      color: #fff;
      display: flex;
      flex-direction: row;
      flex-wrap: nowrap;
      font-size: 20px;
      height: 56px;
      justify-content: flex-start;
      padding: 16px 16px 0 16px;
      position: fixed;
      transition: transform 0.233s cubic-bezier(0, 0, 0.21, 1) 0.1s;
      width: 100%;
      will-change: transform;
      z-index: 1000;
    }

    .header h1 {
      flex: 1;
      font-size: 20px;
      font-weight: 400;
      margin: 0;
    }

    .header .powered-by {
      color: white;
      font-size: 0.6em;
      text-decoration: none;
    }

    .main {
      flex: 1;
      overflow-x: hidden;
      overflow-y: auto;
      padding-top: 60px;
    }

    .weather-card {
      background: #fff;
      border-radius: 2px;
      box-shadow:
        0 2px 2px 0 rgba(0, 0, 0, 0.14),
        0 3px 1px -2px rgba(0, 0, 0, 0.2),
        0 1px 5px 0 rgba(0, 0, 0, 0.12);
      box-sizing: border-box;
      margin: 16px;
      padding: 16px;
      position: relative;
    }

    .offline-panda {
      display: block;
      margin-bottom: 1.5em;
      margin-left: auto;
      margin-right: auto;
    }
  </style>
</head>
<body>
  <header class="header">
    <h1>
      Weather PWA
      <a href="https://darksky.net/poweredby/" class="powered-by">
        Powered by Dark Sky
      </a>
    </h1>
  </header>
  <main class="main">
    <div class="weather-card">
      <img src="data:image/png;base64,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" alt="" class="offline-panda">
      <div>
        Oops, you appear to be offline, this app requires an internet
        connection.
      </div>
    </div>
  </main>
</body>
</html>
//service-worker.js
/*
 * @license
 * Your First PWA Codelab (https://g.co/codelabs/pwa)
 * Copyright 2019 Google Inc. All rights reserved.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     https://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License
 */
'use strict';

// CODELAB: Update cache names any time any of the cached files change.
const CACHE_NAME = 'static-cache-v2';
const DATA_CACHE_NAME = 'data-cache-v1';

// CODELAB: Add list of files to cache here.
const FILES_TO_CACHE = [
  '/',
  '/index.html',
  '/scripts/app.js',
  '/scripts/install.js',
  '/scripts/luxon-1.11.4.js',
  '/styles/inline.css',
  '/images/add.svg',
  '/images/clear-day.svg',
  '/images/clear-night.svg',
  '/images/cloudy.svg',
  '/images/fog.svg',
  '/images/hail.svg',
  '/images/install.svg',
  '/images/partly-cloudy-day.svg',
  '/images/partly-cloudy-night.svg',
  '/images/rain.svg',
  '/images/refresh.svg',
  '/images/sleet.svg',
  '/images/snow.svg',
  '/images/thunderstorm.svg',
  '/images/tornado.svg',
  '/images/wind.svg',
  '/offline.html',
];

self.addEventListener('install', (evt) => {
  console.log('[ServiceWorker] Install');
  // CODELAB: Precache static resources here.
  evt.waitUntil(
    caches.open(CACHE_NAME).then((cache) => {
      console.log('[ServiceWorker] Pre-caching offline page');
      return cache.addAll(FILES_TO_CACHE);
    })
);

  self.skipWaiting();
});

self.addEventListener('activate', (evt) => {
  console.log('[ServiceWorker] Activate');
  // CODELAB: Remove previous cached data from disk.
  evt.waitUntil(
    caches.keys().then((keyList) => {
      return Promise.all(keyList.map((key) => {
        if (key !== CACHE_NAME && key !== DATA_CACHE_NAME) {
          console.log('[ServiceWorker] Removing old cache', key);
          return caches.delete(key);
        }
      }));
    })
);

  self.clients.claim();
});

self.addEventListener('fetch', (evt) => {
  console.log('[ServiceWorker] Fetch', evt.request.url);
  // CODELAB: Add fetch event handler here.
  if (evt.request.mode !== 'navigate') {
    if (evt.request.url.includes('/forecast/')) {
  console.log('[Service Worker] Fetch (data)', evt.request.url);
  evt.respondWith(
      caches.open(DATA_CACHE_NAME).then((cache) => {
        return fetch(evt.request)
            .then((response) => {
              // If the response was good, clone it and store it in the cache.
              if (response.status === 200) {
                cache.put(evt.request.url, response.clone());
              }
              return response;
            }).catch((err) => {
              // Network request failed, try to get it from the cache.
              return cache.match(evt.request);
            });
      }));
  return;
}
evt.respondWith(
    caches.open(CACHE_NAME).then((cache) => {
      return cache.match(evt.request)
          .then((response) => {
            return response || fetch(evt.request);
          });
    })
);

    
  // Not a page navigation, bail.
  return;
}
evt.respondWith(
    fetch(evt.request)
        .catch(() => {
          return caches.open(CACHE_NAME)
              .then((cache) => {
                return cache.match('offline.html');
              });
        })
);

});
