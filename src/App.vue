<!--
SPDX-FileCopyrightText: NOI Techpark <digital@noi.bz.it>

SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <NoiFlightControl :options="{
    language: language,
    showTopDescription: showTopDescription,
    langPack: JSON.parse(langPack),
    theme: theme,
    timezone: timezone,
    localAirport: localairport,
    restEndpoint: restEndpoint,
  }" />
</template>

<script>
import Vue from "vue"
import NoiFlightControl from "./components/NoiFlightControl.vue"
import { Map, TileLayer, OsmSource, Feature, Style } from 'vuelayers'

Vue.use(Map)
Vue.use(TileLayer)
Vue.use(OsmSource)
Vue.use(Feature)
Vue.use(Style)

export default {
  name: "App",
  props: {
    id: {
      type: String,
      default: ""
    },
    styleUrl: {
        type: String,
        default: ""
    },
    showTopDescription: {
      type: Boolean,
      default: true
    },
    language: {
      type: String,
      default: "en"
    },
    langPack: {
      type: String,
      default: "{}"
    },
    localairport: {
      type: String,
      default: "BZO"
    },
    theme: {
      type: String,
      // "OpenDataHub" || "skyalps"
      default: "OpenDataHub"
    },
    timezone: {
      type: String,
      default: "Europe/Rome"
      // https://www.npmjs.com/package/tzdata list
    },
    restEndpoint: {
      type: String,
      default: "https://mobility.api.opendatahub.com/v2/flat%2Cnode/Flight?"
    }
  },
  components: {
    NoiFlightControl
  },
  mounted: async function () {
    if (this.styleUrl != "") {
      const elem = document.getElementById(this.id)
      if (!elem) {
        console.log("you must give the element an id so that external css-files can be applied")
      }

      var externalStyles = document.createElement("style")

      fetch(this.styleUrl)
        .then((response) => response.text())
        .then((data) => {
          externalStyles.innerHTML = data
          elem.shadowRoot.appendChild(externalStyles)
        })
        .catch((error) => {
            console.error("wrong stylesheet url. styles can not be applied. cors disabled (?)")
        })
    }

  },
  created(){
    // add default fonts
    const fontOdh = document.createElement("link");
    fontOdh.type = "text/css";
    fontOdh.rel = "stylesheet";
    fontOdh.href = "https://fonts.testingmachine.eu/source-sans-pro/style.css";
    document.head.appendChild(fontOdh);
    const fontSkyalps = document.createElement("link");
    fontSkyalps.type = "text/css";
    fontSkyalps.rel = "stylesheet";
    fontSkyalps.href = "https://fonts.googleapis.com/css2?family=Barlow+Semi+Condensed:wght@300&display=swap";
    document.head.appendChild(fontSkyalps);
  }


}
</script>