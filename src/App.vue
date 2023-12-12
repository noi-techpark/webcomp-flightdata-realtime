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
    localAirport: localAirport,
    restEndpoint: restEndpoint,
  }" />
</template>

<script>
import Vue from "vue"


import NoiFlightControl from "./components/NoiFlightControl.vue"

// TODO: 4MB (!) unused payload => https://github.com/ghettovoice/vuelayers/issues/319
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
    localAirport: {
      type: String,
      default: "BZO"
    },
    theme: {
      type: String,
      // "odh" || "skyalps"
      default: "odh"
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
  }
}
</script>
