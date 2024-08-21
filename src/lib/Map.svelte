<script>
  import { Map, NavigationControl } from 'mapbox-gl';
  import Spinner from './Spinner.svelte';
  import { token } from '../js';

  const mapWidth = 100;
  const mapHeight = (100 / 16) * 9;

  let promise;

  // const accessToken = import.meta.env.VITE_MAPBOX_TOKEN;
  const accessToken = token;
  let map;
  let zoom = $state(9);
  let coords = $state({ lng: 0, lat: 0 });
  let foundAddress = $state('');
  let loading = $state(false);
  let container = $state();
  $inspect(coords);

  // $effect(renderMap);

  function handleMyPositionClick() {
    console.log('in the function');
    promise = locateUserPosition();
  }

  function renderMap() {
    const initialState = {
      lng: coords.lng,
      lat: coords.lat,
      zoom,
    };

    map = new Map({
      accessToken,
      container,
      style: 'mapbox://styles/mapbox/streets-v12', // style URL
      center: [initialState.lng, initialState.lat], // starting position [lng, lat]
      zoom: initialState.zoom, // starting zoom
    });
    map.addControl(new NavigationControl());
  }

  function locateUserPosition() {
    if (!navigator.geolocation) {
      return alert(
        'Location feature is not available in your browser - please use a more modern browser or manually enter an address'
      );
    }
    loading = true;
    navigator.geolocation.getCurrentPosition(
      async successResult => {
        const coordinates = [
          successResult.coords.longitude,
          successResult.coords.latitude,
        ];
        coords.lng = coordinates.at(0);
        coords.lat = coordinates.at(1);
        console.log(coordinates);
        const address = await getAddressFromCoordinates(coordinates);

        console.log(address);
        // this.selectPlace(coordinates, address);
        renderMap();
        loading = false;
      },
      error => {
        loading = false;
        alert(
          'Could not locate you unfortunately. Please enter an address manually❗'
        );
      }
    );
  }

  async function getAddressFromCoordinates(coordinates) {
    try {
      const [lng, lat] = coordinates;
      if (!lng || !lat)
        throw new Error('You have to provide both the coordinates');
      if (lng > 180 || lng < -180)
        throw new Error('Longitude should be between -180 and 180');
      if (lat > 90 || lat < -90)
        throw new Error('Latitude should be between -90 and 90');
      // console.log(lng, lat, 'coords');
      const url = `https://api.mapbox.com/search/geocode/v6/reverse?longitude=${lng}&latitude=${lat}&access_token=${accessToken}`;
      loading = true;
      const res = await fetch(url);
      if (!res.ok) {
        throw new Error(`Something went wrong with request❗`);
      }
      const data = await res.json();
      if (!data.features.length)
        throw new Error(
          '❌ No address found for the given latitude and longitude.'
        );
      const address = data.features[0].properties.full_address;
      foundAddress = address;
      renderMap();
      return address;
    } catch (err) {
      alert(err.message);
    } finally {
      loading = false;
    }
  }
  async function getCoordsFromAddress(address) {
    if (!address) return;
    try {
      const urlAddress = encodeURI(address);
      console.log(urlAddress);
      const url = `https://api.mapbox.com/search/geocode/v6/forward?q=${urlAddress}&access_token=${accessToken}&proximity=11,46`;
      loading = true;
      const res = await fetch(url);
      if (!res.ok) {
        throw new Error(`Something went wrong with request❗`);
      }
      const data = await res.json();
      console.log(data.features[0].geometry.coordinates);
      const coordinates = data.features[0].geometry.coordinates;
      coords.lng = coordinates[0];
      coords.lat = coordinates[1];
      console.log('coords: ', coords.lng, coords.lat);

      foundAddress = await getAddressFromCoordinates([coords.lng, coords.lat]);
      console.log(address);
      console.log(foundAddress);
      renderMap();
      // console.log(data.features[0].geometry.coordinates);
      return data.features[0].geometry.coordinates;
    } catch (err) {
      alert(err.message);
    } finally {
      loading = false;
    }
  }
</script>

<div class="map-wrap" style="--mapWidth:{mapWidth};--mapHeight:{mapHeight}">
  {#if loading}
    <Spinner color="#25b09b" />
  {/if}
  <div class="sidebar">
    Longitude: {coords.lng.toFixed(4)} | Latitude: {coords.lat.toFixed(4)}
  </div>
  <div class="map" bind:this={container}></div>
</div>
<p class="foundAddress">{foundAddress}</p>
<button class="getPosBtn" onclick={handleMyPositionClick}
  >Get my position</button>
<form
  class="addressForm"
  onsubmit={e => {
    e.preventDefault();
    getCoordsFromAddress(e.target[0].value);
  }}>
  <h3>Find by address</h3>
  <div class="field">
    <label for="address"> Address</label>
    <input type="text" id="address" name="address" />
  </div>
  <button class="hide"></button>
</form>
<form
  onsubmit={e => {
    e.preventDefault();
    coords.lng = +e.target[0].value;
    coords.lat = +e.target[1].value;

    getAddressFromCoordinates([coords.lng, coords.lat]);
  }}>
  <h3>Find by coordinates</h3>
  <div class="field">
    <label for="lng">Longitude <span>(-180 / 180)</span></label>
    <input type="text" id="lng" />
  </div>
  <div class="field">
    <label for="lat">Latitude <span>(-90 / 90)</span></label>
    <input type="text" id="lat" />
  </div>

  <button class="hide"></button>
</form>

<style lang="scss">
  @use '../styles/vars';

  .map-wrap {
    width: calc(var(--mapWidth) * 1%);
    /* flex: 1; */
    /* height: calc(var(--mapHeight) * 1svw); */
    aspect-ratio: 16/9;

    border: 1px solid rgba(#666, 0.3);
    position: relative;
    background-color: #fff;
    border-radius: 10px;

    .map {
      width: 100%;
      height: 100%;
    }

    .sidebar {
      background-color: rgb(35 55 75 / 90%);
      color: #fff;
      padding: 6px 12px;
      font-family: monospace;
      z-index: 1;
      position: absolute;
      top: 0;
      left: 0;
      margin: 12px;
      border-radius: 4px;
      font-size: 0.6rem;
      @media screen and (min-width: vars.$lg) {
        font-size: 1.3rem;
      }
    }
  }

  .foundAddress {
    text-align: center;
    width: 100%;
    border-radius: 10px;
    box-shadow: 0px 3px 3px rgba(#333, 0.5);
    padding: 0.3rem 0.6rem;
  }

  button {
    background-color: #ff3e00;
    color: #fff;
    padding: 0.5rem 1rem;
    border-radius: 10px;
    font-size: 1.5rem;
  }

  .hide {
    display: none;
  }
  form {
    box-shadow: 0px 3px 3px rgba(#333, 0.5);
    padding: 1rem;
    font-size: 1.2rem;
    width: 100%;
    h3 {
      text-align: center;
      margin-bottom: 1rem;
    }
    .field {
      display: flex;
      align-items: center;
      @media screen and (min-width: vars.$lg) {
        justify-content: center;
        margin: 0 auto 1rem;
      }
      @media screen and (min-width: vars.$lg) {
        justify-content: center;
        margin: 0 auto 1rem;
      }
      @media screen and (min-width: vars.$xl) {
        width: 70%;
      }
    }
    label,
    input {
      display: inline-block;
      width: 47%;
      margin-bottom: 1rem;
      font-size: 1.3rem;
    }
    label {
      line-height: 1.2;
      display: flex;
      flex-direction: column;
      align-items: start;
      @media screen and (min-width: vars.$lg) {
        flex-direction: row;
        align-items: center;
        gap: 1rem;
      }
    }
    label span {
      white-space: pre-wrap;
      font-size: 0.8rem;
      color: #666;
    }
    input {
      padding: 0.3rem 0.6rem;
      border-radius: 10px;
      &:focus {
        outline: none;
        border: 1px solid rgba(#ff3e00, 1);
        background-color: rgba(#ff3e00, 1);
        color: #fff;
      }
    }
  }
  .addressForm {
    .field {
      flex-direction: column;
      justify-content: center;
      align-items: start;

      input {
        width: 100%;
      }
    }
    @media screen and (min-width: vars.$lg) {
    }
  }
</style>
