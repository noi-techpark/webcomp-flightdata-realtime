<!--
SPDX-FileCopyrightText: NOI Techpark <digital@noi.bz.it>

SPDX-License-Identifier: AGPL-3.0-or-later
-->
<template>
    <div :class="'container-flex noi-details-container '+device+ ' ' + options.theme"> <!-- :class="{hide: !show}" -->
        <div class="row" v-if="isMobile">
            <div class="col noi-details-mobile-title">
                {{ flight.flightNumber }}: {{ flight.departure.airport.name }} - {{ flight.arrival.airport.name }}
            </div>
        </div>
        <div class="row">
            <div class="col-md-4 order-2 order-md-1 noi-details-description-container">
                <div class="noi-details-description-container-gray" v-if="isMobile">
                    <div class="row" style="padding:0 2rem;">
                        <div class="col noi-details-status" :class="flight.flightInfo['colorClass']">
                            {{ $parent.$t("status."+flight.flightInfo["text"]) }}
                        </div>
                        <div class="col update-button" >
                            <div @click="$parent.refreshData" class="button light">
                                <Icon icon="mdi:refresh"  :horizontalFlip="true"/>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="noi-details-description-container-top">
                    <div class="row" v-if="!isMobile">
                        <div class="col noi-details-date">
                            {{ $parent.asZoneDate(flight.departure.date, "UTC", flight.departure.time,"EEEE, MMMM dd, yyyy") }}
                        </div>
                    </div>
                    <div class="row ">
                        <div class="col noi-details-flight-number">
                            {{ flight.flightNumber }}
                            <img src="https://third-party.opendatahub.com/webcomp-flightdata-realtime/src/assets/icons/skyalpsl-full-alt-trasp.png" />
                        </div>
                    </div>
                    <div class="row">
                        <div class="col noi-details-from-to-title" v-if="!isMobile">
                            {{ $parent.$t("from") }} {{ flight.departure.airport.name }} {{ $parent.$t("to") }} {{ flight.arrival.airport.name }}
                        </div>
                        <div class="col noi-details-from-to-title" v-if="isMobile">
                            {{ $parent.$t("from") }} {{ flight.departure.airport.name }} ({{ flight.departure.airport.iataCode }}) {{ $parent.$t("to") }} {{ flight.arrival.airport.name }} ({{ flight.arrival.airport.iataCode }})
                        </div>
                    </div>
                    <div class="row" v-if="isMobile">
                        <div class="col noi-details-date">
                            {{ $parent.asZoneDate(flight.departure.date, "UTC", flight.departure.time,"EEEE, MMMM dd, yyyy") }}
                        </div>
                    </div>
                    <div class="row">
                        <div class="col container noi-details-flight-row-path">
                            <div class="noi-details-flight-row-plane-line" :class="flight.flightInfo['planeLineClass']">
                                <div class="before">
                                    <Icon icon="tabler:point-filled"/>
                                </div>
                                <div class="plane">
                                    <img v-if="options.theme == 'skyalps'" :src="require('@/assets/icons/plane-big-blue.png')" height="24px"/>
                                    <img v-else :src="require('@/assets/icons/plane-big-black.png')" height="24px"/>
                                    <!-- <Icon icon="material-symbols:flight" rotate="90deg" /> -->
                                </div>
                                <div class="after">
                                    <Icon icon="tabler:point"/>
                                </div>
                            </div>
                            <div class="row noi-details-flight-times">
                                <div class="col departure">
                                    {{ $parent.asZoneTime(flight.departure.time) }}h
                                </div>
                                <div class="col arrival">
                                    {{ $parent.asZoneTime(flight.arrival.time) }}h
                                </div>
                            </div>
                            <span>{{ $parent.$t("timeLeft") }}: {{ $parent.getShownTimeLeft(flight) }} {{ $parent.$t("min") }}</span>
                        </div>
                    </div>
                </div>

                <div class="noi-details-description-container-gray">
                    <div class="row" style="padding:0 3rem;" v-if="!isMobile">
                        <div class="col noi-details-status" :class="flight.flightInfo['colorClass']">
                            {{ $parent.$t("status."+flight.flightInfo["text"]) }}
                            <span>{{ $parent.$t("estimatedArrival") }}: {{ $parent.asZoneTime(flight.arrival.time) }}</span>
                        </div>
                    </div>

                    <div class="row noi-details-from-to">
                        <div class="col-1">
                            <Icon icon="fa-solid:plane-departure"/>
                        </div>
                        <div class="col-7">
                            {{ flight.departure.airport.name }} ({{ flight.departure.airport.iataCode }})
                        </div>
                        <div class="col-4">
                            <span v-if="flight.departure.airport.gate">{{ $parent.$t("gate") }} {{ flight.departure.airport.gate }}</span>
                        </div>
                    </div>
                    <div class="row noi-details-from-to">
                        <div class="col-1">
                            <Icon icon="fa-solid:plane-arrival"/>
                        </div>
                        <div class="col-7">
                            {{ flight.arrival.airport.name }} ({{ flight.arrival.airport.iataCode }})
                        </div>
                        <div class="col-4">
                            <span v-if="flight.arrival.airport.gate">{{ $parent.$t("gate") }} {{ flight.arrival.airport.gate }}</span>
                        </div>
                    </div>
                </div>
  
            </div>
            <div class="col-md-8 order-1 order-md-2 noi-details-map-container">
                <div ref="viewport" style="width: 100%; height: 100%; min-height: 250px; position: relative" >            
                    <vl-map ref="map"
                            :load-tiles-while-animating="true"
                            :load-tiles-while-interacting="true"
                            data-projection="EPSG:4326"
                            style="width: 100%; height: 100%; min-height: 250px"
                            @moveend="setExtend($event)">
        
                        <vl-feature>
                            <vl-geom-point :coordinates="flight.departure.airport.coordinates"></vl-geom-point>
                            <vl-style-box>
                                <vl-style-circle :radius="5">
                                    <vl-style-fill :color="fillColor"></vl-style-fill>
                                    <vl-style-stroke :color="strokeColor"></vl-style-stroke>
                                </vl-style-circle>
                                <vl-style-text :text="flight.departure.airport.iataCode" font="12px monospace" :offsetY="-20">
                                    <vl-style-stroke :color="strokeColor" :width="15"></vl-style-stroke>
                                    <vl-style-fill :color="fillColor" :width="30"></vl-style-fill>
                                </vl-style-text>
                            </vl-style-box>
                        </vl-feature>
    
                        <vl-feature>
                            <vl-geom-point :coordinates="getLineCoordinates(flight).plane"></vl-geom-point>
                            <vl-style-box>
                                <vl-style-icon v-if="options.theme == 'skyalps'" :src="require('@/assets/icons/plane-big-blue.png')"
                                            :scale="0.4"
                                            :rotation="((flight.angle) * Math.PI * 2)/360"
                                            :anchor="[0.5, 0.5]">
                                        <vl-style-fill :color="strokeColor"></vl-style-fill>
                                </vl-style-icon>
                                <vl-style-icon v-else :src="require('@/assets/icons/plane-big-black.png')"
                                            :scale="0.4"
                                            :rotation="((flight.angle) * Math.PI * 2)/360"
                                            :anchor="[0.5, 0.5]">
                                        <vl-style-fill :color="strokeColor"></vl-style-fill>
                                </vl-style-icon>
                            </vl-style-box>
                        </vl-feature>

                        <vl-feature>
                            <vl-geom-point :coordinates="flight.arrival.airport.coordinates"></vl-geom-point>
                            <vl-style-box>
                                <vl-style-circle :radius="5">
                                    <vl-style-fill :color="fillColor"></vl-style-fill>
                                    <vl-style-stroke :color="strokeColor"></vl-style-stroke>
                                </vl-style-circle>

                                <vl-style-text :text="flight.arrival.airport.iataCode" font="12px monospace" :offsetY="-20">
                                    <vl-style-stroke :color="fillColor" :width="15"></vl-style-stroke>
                                    <vl-style-fill :color="strokeColor" :width="30"></vl-style-fill>
                                </vl-style-text>
                            </vl-style-box>
                        </vl-feature>
    
                        <vl-feature v-if="getLineCoordinates(flight).start">
                            <vl-geom-line-string :coordinates="getLineCoordinates(flight).start"></vl-geom-line-string>
                            <vl-style-box>
                                <vl-style>
                                    <vl-style-stroke :color="strokeColor" :width="2"></vl-style-stroke>
                                </vl-style>
                            </vl-style-box>
                        </vl-feature>
                        <vl-feature v-if="getLineCoordinates(flight).end">
                            <vl-geom-line-string :coordinates="getLineCoordinates(flight).end"></vl-geom-line-string>
                            <vl-style-box>
                                <vl-style>
                                    <vl-style-stroke :color="strokeColor"  :width="2" :line-dash="[5, 5]"></vl-style-stroke>
                                </vl-style>
                            </vl-style-box>
                        </vl-feature>

                        <vl-view :zoom.sync="map.zoom"
                                :center.sync="map.center"
                                :rotation.sync="map.rotation"
                                :min-zoom="((isMobile) ? 4.5 : 5)"></vl-view>
        
                        <vl-layer-tile id="osm">
                            <vl-source-osm></vl-source-osm>
                        </vl-layer-tile>
                    </vl-map>
                </div>
            </div>
        </div>

        
    </div>
</template>

<script>
    import { getBottomLeft, getTopRight } from "ol/extent"
    import { toLonLat } from "ol/proj"
    import { Icon } from '@iconify/vue2';
    
    export default {
        components:{Icon},
        props: {
            options: Object,
            show:true,
            device:{
                type: String,
                required: true
            },
            flight: {
                type: Object,
                required: true
            }
        },
        data() {
            return {
                map: {
                    zoom: 5.5,
                    current_tiles: "https://tile.openstreetmap.org/{z}/{x}/{y}.png",
                    center: [11.3361503, 46.4607403],
                    rotation: 0,
                    viewport: {
                        minLat: 45.3702,
                        minLng: 9.4166,
                        maxLat: 47.3768,
                        maxLng: 13.9527
                    },
                    features: {},
                    trails: {}
                }
            }
        },
        computed: {
            isMobile(){
                return (this.device == 'smartphone' || this.device == 'tablet');
            }
        },

        created: function () {
            this.fillColor = 'white';
            this.strokeColor = 'black';
            if(this.options.theme == 'skyalps'){
                this.strokeColor = '#004988';
            }
        },
        mounted: function () {
            this.flight.angle = this.getFlightAngle(this.flight)*1 + 90
            this.map.center = this.getCenterCoordinates(this.flight);
            this.fitZoom(this.flight);
        },

        methods: {
            getFlightAngle(flight){
                let dx = (flight.arrival.airport.coordinates[0]*1.0-flight.departure.airport.coordinates[0]*1.0);
                let dy = (flight.arrival.airport.coordinates[1]*1.0-flight.departure.airport.coordinates[1]*1.0);
                let angle = (Math.atan2(dx,dy)*180.0)/Math.PI;
                return angle;
            },
            reduceCoordinates(coordinates,which,offset){
                let offsetY = offset*((coordinates[0][1]-coordinates[1][1]));
                let offsetX = offset*((coordinates[0][0]-coordinates[1][0]));
                coordinates[which] = [coordinates[which][0]+offsetX,coordinates[which][1]+offsetY];
                return coordinates;
            },
            getLineCoordinates(flight){
                let offset = 0.1 * 2 / this.map.zoom;

                let startCoordinates = flight.departure.airport.coordinates
                let endCoordinates = flight.arrival.airport.coordinates;
                let centerCoordinates = this.getCenterCoordinates(flight);
                let lineCoordinates = {
                    start:[],
                    end:[]
                }

                let planeLineClass = flight.flightInfo['planeLineClass'];
                switch(planeLineClass){
                    case 'end':
                        lineCoordinates.start = this.reduceCoordinates([startCoordinates,endCoordinates],1,offset*2);
                        lineCoordinates.plane = this.reduceCoordinates([startCoordinates,endCoordinates],1,offset)[1]
                        lineCoordinates.end = null;
                        break;
                    case 'middle':
                        lineCoordinates.start = this.reduceCoordinates([startCoordinates,centerCoordinates],1,offset);
                        lineCoordinates.plane = centerCoordinates;
                        lineCoordinates.end = this.reduceCoordinates([centerCoordinates,endCoordinates],0,-offset);
                        break;
                    case 'start':
                    default:
                        lineCoordinates.start = null;
                        lineCoordinates.plane = this.reduceCoordinates([startCoordinates,endCoordinates],0,-offset)[0];
                        lineCoordinates.end = this.reduceCoordinates([startCoordinates,endCoordinates],0,-offset*2);
                        break;
                }
                return lineCoordinates;
            },
            getCenterCoordinates(flight){
                let x = (flight.departure.airport.coordinates[0]+flight.arrival.airport.coordinates[0])/2;
                let y = (flight.departure.airport.coordinates[1]+flight.arrival.airport.coordinates[1])/2;
                return [x,y];
            },

            getPoligonCoordinates(airportCorrdinates){
                let coordinates = [];
                coordinates.push([airportCorrdinates[0]-1,airportCorrdinates[1]]);
                coordinates.push([airportCorrdinates[0]-1,airportCorrdinates[1]-1]);
                coordinates.push([airportCorrdinates[0],airportCorrdinates[1]-1]);
                coordinates.push([airportCorrdinates[0],airportCorrdinates[1]]);
                return coordinates;
            },
            adjust_map_zoom(evt,x1, y1, x2, y2) {
                const map = evt.map
            },
            fitZoom(flight){
                let x2 = Math.pow(flight.departure.airport.coordinates[0]-flight.arrival.airport.coordinates[0],2);
                let y2 = Math.pow(flight.departure.airport.coordinates[1]-flight.arrival.airport.coordinates[1],2);
                let distance = Math.sqrt(x2+y2);
                console.log("distance",distance)
                if(distance == 0){
                    distance = 1;
                }
                let zoom = 38/distance;
                if(this.isMobile){
                    zoom = zoom*0.8;
                }

                if(zoom > 25){
                    zoom = 25;
                }
                if(zoom < 1){
                    zoom = 1;
                }
                console.log("zoom",zoom)
                this.map.zoom = zoom;
            },

            setExtend(evt) {
                const map = evt.map
                const extent = map.getView().calculateExtent(map.getSize())

                const bottomLeft = toLonLat(getBottomLeft(extent))
                const topRight = toLonLat(getTopRight(extent))

                this.map.viewport = {
                    maxLat: topRight[1],
                    minLat: bottomLeft[1],
                    maxLng: topRight[0],
                    minLng: bottomLeft[0]
                }
            }
        }

    };
</script>

<style lang="scss">

    .noi-details-container{
        display: block;
        position: relative;
        top:0;
        left:0;
        width: 100%;
        height: 100%;
        background-color: white;
        z-index: 10;
        color:var(--noi-text-primary, black);
        margin-bottom:2rem;

        &.hide{
            display: none;
        }
        
        &.skyalps{
            &.smartphone, &.tablet{
                .noi-details-mobile-title{
                    background-color: #004988;
                    color:#ffffff;
                }
            }
        }

        // mobile
        &.smartphone, &.tablet{ 
            margin-bottom:0;
            padding-bottom:1rem;

            .noi-details-mobile-title{
                background-color:  var(--noi-primary, #ffffff);
                color: var(--noi-text-primary, black);
                text-align: center;
                font-size: 1rem;
                text-transform: uppercase;
                font-weight: bold;
                padding:1rem;
            }    


            .noi-details-description-container{

                .update-button{
                    display: inline;
                    text-align: right;
                    padding-top: 0.5rem;
                    .button{
                        font-size: 1.5rem;
                        background-color: transparent;
                        max-width: fit-content;
                        padding:0.2rem 0.4rem 0.4rem 0.5rem;
                    }
                }
                
                .noi-details-description-container-top{
                    .noi-details-flight-number{
                        img{
                            height: 1.5rem;
                            padding-left: 0.5rem;
                        }
                    }
                    .noi-details-from-to-title{
                        font-size: 1.15rem;
                    }
                    .noi-details-date{
                        font-size: 1rem;
                        font-weight: normal; 
                        padding-top: 1rem;
                        padding-bottom: 0;;
                    }
                    .noi-details-flight-row-path{
                        padding: 1rem 0 0.5rem 0;

                        .noi-details-flight-row-plane-line{
                            &.end{
                                div.before{
                                    width: calc(100% - 1.75rem);
                                    left:0;
                                }
                                div.plane{
                                    left:calc(100% - 3.2rem);
                                }
                                div.after{
                                    width: 0;
                                    right: 1rem;
                                }
                            }
                        }
                        .noi-details-flight-times{
                            font-weight: bold;    
                            margin: 1.8rem -0.5rem 0 -0.9rem;
                        }
                    }
                }
                .noi-details-description-container-gray{
                    .noi-details-status{
                        font-size: 1.3rem;
                        padding: 1rem 0;
                    }

                    .noi-details-from-to{
                        background-color: #ffffff;
                        font-size: 1rem;
                        font-weight: bold;
                        padding: 1rem;
                        border-top: 1px solid #e5e5e5;

                        .col-7{
                            padding-left:1.5rem;
                        }
                    }
                }
            }
        }



        // desktop
        .noi-details-description-container{
            background-color: white;
            padding-right: 0;
            
            // white area
            .noi-details-description-container-top{
                border-top: 1px solid #f5f5f5;
                border-left: 1px solid #f5f5f5;
                background-color: white;
                padding: 2rem 3rem 1rem 3rem;
                
                .noi-details-date{
                    text-align: center;                
                    font-size: 1.125rem;
                    font-weight: bold;
                    padding-bottom: 2.1rem;
                }
                .noi-details-flight-number{
                    text-align:center;         
                    font-size: 1rem;

                    img{
                        height: 1.8rem;
                        padding-left: 0.5rem;
                    }

                }
                .noi-details-from-to-title{
                    color:var(--noi-text-primary,#000000);
                    text-align: center;
                    font-size: 1.375rem;
                    font-weight: bold;
                    padding-top: 0.6rem;
                }
                .noi-details-flight-row-path{
                    text-align: center;
                    padding: 2rem 0 1rem 0;

                    .noi-details-flight-row-plane-line{
                        margin: 1rem 0;
                        position: relative;

                        div.before,div.after{
                            height: 2px;
                            position: absolute;
                            padding:0;
                            margin: 0;
                            svg{
                                width: 1.5rem;
                                height: 1.5rem;
                                position: absolute;
                                top: -0.9rem;
                                z-index: 100;
                            }
                        }

                        div.before{
                            border-top:2px solid var(--noi-secondary, black);
                            svg{
                                color:var(--noi-secondary, black);
                                left: -0.5rem;
                            }
                        }
                        div.plane{
                            font-size: 1.5rem;
                            background: white;
                            padding: 0 4px;
                            margin: -0.9rem 0;
                            position: absolute;
                            width: 2rem;
                            color:var(--noi-secondary, black);
                            z-index: 2;
                            background-color: white;
                            line-height: 1.5rem;
                        }
                        div.after{
                            border-top:2px dashed var(--noi-light-bg, #F5F4F6);
                            svg{
                                color:var(--noi-secondary, black);
                                right: -1rem;
                            }
                        }

                        &.start{
                            div.before{
                                width: 0;
                            }
                            div.plane{
                                left:0.5rem;
                            }
                            div.after{
                                width: calc(100% - 0.75rem);
                                left:0.75;
                            }
                        }
                        &.middle{
                            div.before{
                                width: calc(50% - 0.75rem);
                                left:0;
                            }
                            div.plane{
                                left:calc(50% - 2rem);
                            }
                            div.after{
                                width: calc(50% - 0.75rem);
                                left:50%;
                            }
                        }
                        &.end{
                            div.before{
                                width: calc(100% - 0.75rem);
                                left:0;
                            }
                            div.plane{
                                left:calc(100% - 2.2rem);
                            }
                            div.after{
                                width: 0;
                                right: 0;
                            }
                        }
                        
                    }
                    .noi-details-flight-times{
                        font-size: 1rem;
                        margin: 2.5rem -0.8rem 0 -0.8rem;

                        .departure{
                            text-align:left;
                        }
                        .arrival{
                            text-align:right;
                        }
                    }
                    span{
                        font-size: 0.875rem;
                    }
                    
                }

            }


            // gray area
            .noi-details-description-container-gray{
                background-color: #f5f5f5;

                .noi-details-status{
                    font-size: 1.5rem;
                    font-weight: bold;
                    padding: 2rem 0;

                    &.green{
                        color: green;
                    }
                    &.gray{
                        color: gray;
                    }
                    &.yellow{
                        color:gold;
                    }
                    &.red{
                        color: red;
                    }

                    span{
                        color:var(--noi-text-primary, black);
                        font-size: 1rem;
                        font-weight: bold;
                        padding:2.5rem 0 2rem 1rem;
                    }
                }
                .noi-details-from-to{
                    font-size: 1rem;
                    font-weight: bold;
                    padding: 2rem;
                    border-top: 1px solid #e5e5e5;
                }
            }
        }

        .noi-details-map-container{
            padding-left: 0;
        }
    }
</style>