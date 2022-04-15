<template>
  <div class="container col">
    <div ref="exampleMap" class="flex-grow" style="background: purple"></div>

    <div class="row gap-4">
      <CButton v-if="!exampleMapId" @click="startMap"> CreateMap </CButton>
      <CButton @click="addMarker">AddMarker</CButton>
      <CButton @click="removeMarker">RemoveMarker</CButton>
      <CButton @click="addCluster">AddCluster</CButton>
      <CButton @click="addPolygon">AddPolygon</CButton>
      <CButton @click="updatePolygon">UpdatePolygon</CButton>
      <CButton @click="removePolygon">RemovePolygon</CButton>
      <CButton @click="getPolygon">GetPolygon</CButton>
    </div>
  </div>
</template>

<script setup lang="ts">
import { CapacitorGoogleMaps } from '@capacitor-community/google-maps';
import { ref, onMounted } from 'vue';

import CButton from './CButton.vue';

const exampleMapId = ref<string | null>(null);
const exampleMap = ref<InstanceType<typeof HTMLElement> | null>(null);

var markerId = "";
var polygonId = "";

const getRandomArbitrary = (min: number, max: number) => {
  return Math.random() * (max - min) + min;
};

const startMap = async () => {
  // if map already (being) initialized, avoid a new map being initialized
  if (exampleMapId.value != null) {
    return;
  }

  // find element where element should be attached to
  const element = exampleMap.value;
  if (!element) {
    return;
  }

  // prevent race condition, where firing `startMap` quickly after one another, starts up multiple maps
  exampleMapId.value = '';

  // remove background, so map can be seen
  element.style.background = '';

  // get boundaries of element
  const boundingRect = element.getBoundingClientRect();

  // finally create the map
  try {
    const result = await CapacitorGoogleMaps.createMap({
      boundingRect: {
        width: Math.round(boundingRect.width),
        height: Math.round(boundingRect.height),
        x: Math.round(boundingRect.x),
        y: Math.round(boundingRect.y),
      },
      cameraPosition: {
        target: {
          latitude: -33.86,
          longitude: 151.2,
        },
        zoom: 3,
      },
      preferences: {
        appearance: {
          isTrafficShown: true,
        },
      },
    });

    // remember mapId for curent element for later use
    exampleMapId.value = result.googleMap.mapId;

    // set `data-maps-id` attribute for delegating touch events
    element.setAttribute('data-maps-id', exampleMapId.value);

    // remove background, so map can be seen
    element.style.background = '';

    alert('map created: ' + JSON.stringify(result, null, 1));

    // since we now know the mapId, we can start listening to events

    CapacitorGoogleMaps.didTapMarker(
      {
        mapId: exampleMapId.value,
      },
      () => {
        // this will never fire, because it is overwritten by the next `didTapMarker` listener
        alert('this will never be shown');
      }
    );

    CapacitorGoogleMaps.didTapMarker(
      {
        mapId: exampleMapId.value,
        preventDefault: true, // set to true if you do not want the default behaviour
      },
      (result) => {
        // but this will fire
        alert('didTapMarker: ' + JSON.stringify(result, null, 1));
      }
    );

      CapacitorGoogleMaps.didTapPolygon(
      {
        mapId: exampleMapId.value,
      },
      (result) => {
        alert('didTapPolygon: ' + JSON.stringify(result, null, 1));
      }
    );

    CapacitorGoogleMaps.didEndMovingCamera(
      {
        mapId: exampleMapId.value,
      },
      (result) => {
        const latitude = result.cameraPosition.target?.latitude;
        const longitude = result.cameraPosition.target?.longitude;
      }
    );
  } catch (e) {
    console.error(e);
    exampleMapId.value = null;
  }
};

const removeMarker = async () => {
  if (!exampleMapId.value) {
    return;
  }

  const result = await CapacitorGoogleMaps.removeMarker({
    mapId: exampleMapId.value,
    markerId: markerId
  });
};

const addMarker = async () => {
  if (!exampleMapId.value) {
    return;
  }

  const result = await CapacitorGoogleMaps.addMarker({
    mapId: exampleMapId.value,
    icon: {
      url: 'https://www.google.com/favicon.ico',
      targetSizePx: {
        width: 64,
        height: 64
      }
    },
    position: {
      latitude: -33.86,
      longitude: 151.2,
    },
    preferences: {
      title: 'Some title',
      snippet: 'Some snippet',
      metadata: {
        some: 'data',
      },
    },
  });
  markerId = result.marker.markerId;
  alert('marker created: ' + JSON.stringify(result, null, 1));
};

const addCluster = async () => {
  if (!exampleMapId.value) {
    return;
  }

  const result = await CapacitorGoogleMaps.addCluster({
    mapId: exampleMapId.value,
    markers: [
      {
        position: {
          latitude: -33.86,
          longitude: 151.2,
        },
        preferences: {
          title: 'Google',
          snippet: 'Some snippet',
          metadata: {
            some: 'data',
          },
        },
        icon: {
          url: 'https://www.google.com/favicon.ico',
          targetSizePx: {
            width: 64,
            height: 64
          }
        },
      },
      {
        position: {
          latitude: -33.86,
          longitude: 151.201,
        },
        preferences: {
          title: 'Instagram',
          snippet: 'Some snippet',
          metadata: {
            some: 'data',
          },
        },
        icon: {
          url: 'https://www.instagram.com/favicon.ico',
          targetSizePx: {
            width: 64,
            height: 64
          }
        },
      },
      {
        position: {
          latitude: -33.861,
          longitude: 151.201,
        },
        preferences: {
          title: 'Facebook',
          snippet: 'Some snippet',
          metadata: {
            some: 'data',
          },
        },
        icon: {
          url: 'https://www.facebook.com/favicon.ico',
          targetSizePx: {
            width: 64,
            height: 64
          }
        },
      },
      {
        position: {
          latitude: 53.958332,
          longitude: -1.080278,
        },
        preferences: {
          title: 'BBC',
          snippet: 'TV company',
          metadata: {
            some: 'data',
          },
        },
        icon: {
          url: 'https://www.bbc.com/favicon.ico',
          targetSizePx: {
            width: 64,
            height: 64
          }
        },
      },
      {
        position: {
          latitude: 52.192001,
          longitude: -2.220000,
        },
        preferences: {
          title: 'The Guardian',
          snippet: 'News',
          anchor: {x: 0.5, y: 0.5},
          metadata: {
            some: 'data',
          },
        },
        icon: {
          url: 'https://www.theguardian.com/favicon.ico',
          targetSizePx: {
            width: 128,
            height: 128
          }
        },
      },
    ],
    clusterIcon: {
      url: 'http://172.16.0.114/point-of-interest.svg',
      targetSizePx: {
        width: 128,
        height: 128
      }
    },
    clusterCaptionPrefs: {
      padding: {
        left: 2,
        right: 2,
        bottom: 8
      },
      textSize: 16,
      textColor: "#0000FF"
    }
  });

  alert('cluster created: ' + JSON.stringify(result, null, 1));
};

const addPolygon = async () => {
  if (!exampleMapId.value) {
    return;
  }

  const result = await CapacitorGoogleMaps.addPolygon({
    mapId: exampleMapId.value,
    points: [
      {latitude: 52.36545, longitude: 4.70995},
      {latitude: 52.36377, longitude: 4.71648},
      {latitude: 52.36262, longitude: 4.71510},
      {latitude: 52.33483, longitude: 4.71322},
      {latitude: 52.32801, longitude: 4.71407},
      {latitude: 52.32549, longitude: 4.71047},
      {latitude: 52.32570, longitude: 4.70583},
    ],
    preferences: {
      isClickable: true,
      holes: [
        [
          {latitude: 52.33752147380871, longitude: 4.709846874581462},
          {latitude: 52.334971943107575, longitude: 4.71180417415214},
          {latitude: 52.33342361091616, longitude: 4.710059294689908},
        ],
        [
          {latitude: 52.32817731200254, longitude: 4.708302710330382},
          {latitude: 52.32827088058939, longitude: 4.7093999677920895},
          {latitude: 52.32696090236376, longitude: 4.709221344484369},
          {latitude: 52.3271792347618, longitude: 4.70814960463805},
        ],
      ],
      strokePattern: [{pattern: 'Dash', length: 20}, {pattern: 'Gap', length: 20},],
      strokeWidth: 3,
      metadata: {someText: 'Hello'}
    }
  });
  polygonId = result.polygon.polygonId;
  alert('polygon created: ' + JSON.stringify(result, null, 1));
};

const updatePolygon = async () => {
  if (!exampleMapId.value) {
    return;
  }

  const result = await CapacitorGoogleMaps.updatePolygon({
    mapId: exampleMapId.value,
    polygonId: polygonId,
    preferences: {
      isClickable: true,
      strokeWidth: 3,
      metadata: {someText: 'Updated message'}
    }
  });
  alert('polygon updated: ' + JSON.stringify(result, null, 1));
};


const removePolygon = async () => {
  if (!exampleMapId.value) {
    return;
  }
  const result = await CapacitorGoogleMaps.removePolygon({
    mapId: exampleMapId.value,
    polygonId: polygonId
  });
  alert('polygon removed: ' + JSON.stringify(result, null, 1));
}

const getPolygon = async () => {
  if (!exampleMapId.value) {
    return;
  }
  const result = await CapacitorGoogleMaps.getPolygon({
    mapId: exampleMapId.value,
    polygonId: polygonId
  });
  alert('get polygon: ' + JSON.stringify(result, null, 1));
}

const moveCamera = async () => {
  if (!exampleMapId.value) {
    return;
  }

  await CapacitorGoogleMaps.moveCamera({
    mapId: exampleMapId.value,
    cameraPosition: {
      target: {
        latitude: -33.86,
        longitude: 151.2,
      },
      zoom: 12,
    },
    duration: 1000,
  });
};

onMounted(async () => {
  // initialize map
  await CapacitorGoogleMaps.initialize({
    key: 'your_api_key',
    devicePixelRatio: window.devicePixelRatio,
  });
});
</script>

<style scoped>
.row {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-evenly;
}

.col {
  display: flex;
  flex-direction: column;
}

.flex-grow {
  flex-grow: 1;
}

.gap-4 {
  gap: 1rem;
}
</style>
