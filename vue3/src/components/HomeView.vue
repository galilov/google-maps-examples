<template>
  <div class="container col">
    <div ref="exampleMap" class="flex-grow" style="background: purple"></div>

    <div class="row gap-4">
      <CButton v-if="!exampleMapId" @click="startMap"> CreateMap </CButton>
      <!-- <CButton @click="addMarker"> AddMarker </CButton>  -->
      <CButton @click="addMarkers"> AddMarkers (3000) </CButton>
      <CButton @click="moveCamera"> MoveCamera </CButton>
      <CButton @click="addCircle">AddCircle</CButton>
      <CButton @click="getCircle">GetCircle</CButton>
      <CButton @click="addCluster">AddCluster</CButton>
      <CButton @click="addPolygon">AddPolygon</CButton>
      <CButton @click="updatePolygon">UpdatePolygon</CButton>
      <CButton @click="removePolygon">RemovePolygon</CButton>
      <CButton @click="getPolygon">GetPolygon</CButton>
      <!--<CButton @click="addPolyline">AddPolyline</CButton>
      <CButton @click="getPolyline">GetPolyline</CButton>
      <CButton @click="updatePolyline">UpdatePolyline</CButton>
      <CButton @click="removePolyline">removePolyline</CButton> -->
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
var polylineId = "";
var circleId = "";

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
        zoom: 12,
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

    CapacitorGoogleMaps.didTapPolyline(
      {
        mapId: exampleMapId.value,
      },
      (result) => {
        alert('didTapPolyline: ' + JSON.stringify(result, null, 1));
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
    position: {
      latitude: -33.86,
      longitude: 151.2,
    },
    preferences: {
      title: 'Some title',
      snippet: 'Some snippet',
      icon: {
        url: 'https://www.google.com/favicon.ico',
        size: {
          width: 64,
          height: 64
        }
      },
      metadata: {
        some: 'this is metadata',
      },
    },
  });
  markerId = result.marker.markerId;
  alert('marker created: ' + JSON.stringify(result, null, 1));
};

const addMarkers = async () => {
  if (!exampleMapId.value) {
    return;
  }

  const markers = new Array(3000).fill(true);

  var markerEntries = new Array();

  markers.map(() => {
    markerEntries.push({
      position: {
        latitude: -getRandomArbitrary(30, 35),
        longitude: getRandomArbitrary(150, 155),
      },
      preferences: {
        title: 'Some title',
        snippet: 'Some snippet',
        metadata: {
          some: 'data',
        },
        icon: {
          url: 'https://raw.githubusercontent.com/galilov/google-maps-examples/main/airplane.svg',
        },
      }
    });
  });

  let addMarkersOptions = {
    mapId: exampleMapId.value,
    markers: markerEntries
  }

  const result = await CapacitorGoogleMaps.addMarkers(addMarkersOptions);
  alert('markers created');
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
          icon: {
            url: 'https://www.google.com/favicon.ico',
            size: {
              width: 64,
              height: 64
            }
          },
          metadata: {
            some: 'data',
          },
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
          icon: {
            url: 'https://www.instagram.com/favicon.ico',
            size: {
              width: 64,
              height: 64
            }
          },
          metadata: {
            some: 'data',
          },
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
          icon: {
            url: 'https://www.facebook.com/favicon.ico',
            size: {
              width: 64,
              height: 64
            }
          },
          metadata: {
            some: 'data',
          },
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
          icon: {
            url: 'https://www.bbc.com/favicon.ico',
            size: {
              width: 64,
              height: 64
            }
          },
          metadata: {
            some: 'data',
          },
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
          icon: {
            url: 'https://www.theguardian.com/favicon.ico',
            size: {
              width: 128,
              height: 128
            }
          },
          metadata: {
            some: 'data',
          },
        },
      },
    ],
    /*clusterIcon: {
      url: 'http://172.16.0.114/point-of-interest.svg',
      size: {
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
    }*/
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
      {latitude: -23.344373454330547, longitude: 114.30080112467083},
      {latitude: -18.835698392780067, longitude: 121.68361358134968},
      {latitude: -13.861653304809693, longitude: 126.86916042592173},
      {latitude: -13.605520021054103, longitude: 127.04494167489028},
      {latitude: -11.029736250663484, longitude: 130.20900415632406},
      {latitude: -11.805092096522086, longitude: 136.44923849470737},
      {latitude: -14.798357688586348, longitude: 135.74611349883318},
      {latitude: -17.163989735408332, longitude: 139.96486347407827},
      {latitude: -10.857151647259334, longitude: 142.3379103351536},
      {latitude: -13.861653304809693, longitude: 144.4472853227761},
      {latitude: -18.75249389014763, longitude: 146.4687696859144},
      {latitude: -19.99616591493106, longitude: 148.5781446735369},
      {latitude: -22.128491318525338, longitude: 150.68751966115943},
      {latitude: -25.107585835817005, longitude: 152.88478527326626},
      {latitude: -28.634771032354635, longitude: 153.67580089362468},
      {latitude: -31.599216872081893, longitude: 152.53322277532916},
      {latitude: -37.31958582890507, longitude: 149.9843946652853},
      {latitude: -38.4983983230759, longitude: 147.1718946817886},
      {latitude: -38.153650818473956, longitude: 140.66798846995243},
      {latitude: -35.766214712702045, longitude: 138.11916035990853},
      {latitude: -34.90584424370075, longitude: 135.30666037641183},
      {latitude: -31.599216872081893, longitude: 131.4394728991039},
      {latitude: -33.89057188468875, longitude: 123.1777541975823},
      {latitude: -34.90584424370075, longitude: 118.34376985094735},
      {latitude: -33.96349903353246, longitude: 115.00392612054499},
      {latitude: -31.148980062555705, longitude: 115.53126986745065},
      {latitude: -26.531628708497426, longitude: 113.1582230063753},
    ],
    preferences: {
      isClickable: true,
      strokePattern: [{pattern: 'Dash', length: 20}, {pattern: 'Gap', length: 20},],
      strokeWidth: 3,
      patternIcon: {
        url: 'https://raw.githubusercontent.com/galilov/google-maps-examples/main/1.png',
        size: {width: 20, height: 20}
      },
      metadata: {someText: 'Hello'},
      aboveMarkers: false,
    }
  });
  polygonId = result.polygon.id;
  alert('polygon created: ' + JSON.stringify(result, null, 1));
};

const updatePolygon = async () => {
  if (!exampleMapId.value) {
    return;
  }

  const result = await CapacitorGoogleMaps.updatePolygon({
    mapId: exampleMapId.value,
    id: polygonId,
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
    id: polygonId
  });
  alert('polygon removed: ' + JSON.stringify(result, null, 1));
}

const getPolygon = async () => {
  if (!exampleMapId.value) {
    return;
  }
  const result = await CapacitorGoogleMaps.getPolygon({
    mapId: exampleMapId.value,
    id: polygonId
  });
  alert('get polygon: ' + JSON.stringify(result, null, 1));
}

const addCircle = async () => {
  if (!exampleMapId.value) {
    return;
  }

  const result = await CapacitorGoogleMaps.addCircle({
    mapId: exampleMapId.value,
    center: {latitude: 45, longitude: 45},
    radius: 100000,
    preferences: {
      isClickable: true,
      strokePattern: [{pattern: 'Dash', length: 20}, {pattern: 'Gap', length: 20},],
      strokeWidth: 3,
      metadata: {someText: 'Hello CIRCLE'}
    }
  });
  circleId = result.circle.id;
  alert('circle created: ' + JSON.stringify(result, null, 1));
};

const getCircle = async () => {
  if (!exampleMapId.value) {
    return;
  }
  const result = await CapacitorGoogleMaps.getCircle({
    mapId: exampleMapId.value,
    id: circleId
  });
  alert('get Circle: ' + JSON.stringify(result, null, 1));
}

const addPolyline = async () => {
  if (!exampleMapId.value) {
    return;
  }
  const result = await CapacitorGoogleMaps.addPolyline({
    mapId: exampleMapId.value,
    points: [
      {latitude:-25.17263, longitude: 62.68250},
      {latitude: -8.58917, longitude: 74.28406},
      {latitude: -14.61282, longitude: 78.15125},
      {latitude: -8.24139, longitude: 82.72156},
      {latitude: -18.32063, longitude: 93.26843},
    ],
    preferences: {
      isClickable: false,
      pattern: [{pattern: 'Dash', length: 20}, {pattern: 'Gap', length: 20},],
      width: 10,
      color: '#ff0000',
      metadata: {mydata: 'Hello POLYLINE!'}
    }
  });
  polylineId = result.polyline.id;
  alert('polyline created: ' + JSON.stringify(result, null, 1));
};

const getPolyline = async () => {
  if (!exampleMapId.value) {
    return;
  }
  const result = await CapacitorGoogleMaps.getPolyline({
    mapId: exampleMapId.value,
    id: polylineId
  });
  alert('get polyline: ' + JSON.stringify(result, null, 1));
}

const updatePolyline = async () => {
  if (!exampleMapId.value) {
    return;
  }

  const result = await CapacitorGoogleMaps.updatePolyline({
    mapId: exampleMapId.value,
    id: polylineId,
    preferences: {
      isClickable: true,
      width: 1,
      color: '#0000ff',
      metadata: {someText: 'Updated polyline message'}
    }
  });
  alert('polyline updated: ' + JSON.stringify(result, null, 1));
};

const removePolyline = async () => {
  if (!exampleMapId.value) {
    return;
  }
  const result = await CapacitorGoogleMaps.removePolyline({
    mapId: exampleMapId.value,
    id: polylineId
  });
  alert('polyline removed: ' + JSON.stringify(result, null, 1));
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
      zoom: 0,
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