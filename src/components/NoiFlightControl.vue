<!--
SPDX-FileCopyrightText: NOI Techpark <digital@noi.bz.it>

SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <div :class="'noi-flightdata-realitime global ' + device + ' ' + options.theme" v-show="options" ref="viewport" style="width: 100%;  position: relative;"> <!-- height: 100%; min-height:200px; -->

    <div class="container-fluid">
        <div class="row noi-top-bar">
            <div class="col">
                <div @click="closeDetails()"  v-if="(device != 'desktop' && details.show)">
                    <Icon class="noi-top-back-icon" icon="ep:arrow-left"/>
                </div>
                <img v-if="options.theme == 'skyalps'" :src="require('@/assets/icons/skyalpsl-full-alt-trasp.png')" />
                <img v-else :src="require('@/assets/icons/odh.svg')" />
            </div>
        </div>
        <div class="row noi-description-bar" v-if="options.showTopDescription && !showResults && !details.show" :class="{'pt-2': showResults}">
            <div class="col">
                <h2>{{ $t("header.title") }}</h2>
                <p>{{ $t("header.description") }} <strong>{{ localAirport.name }} ({{ localAirport.key }})</strong></p>
            </div>
        </div>
    </div>

    <div class="noi-content-container">
        <div class="noi-details-back" v-if="details.show && device == 'desktop'">
            <span @click="closeDetails()"><Icon icon="ep:arrow-left" /> {{ $t("backToList") }}</span>
        </div>

        <!-- search container -->
        <div class="noi-change-search " v-if="device =='desktop'" :class="{'noi-hide': (!showResults || details.show)}">
            <span @click="toggleSearchPanel()">
                {{ $t("changeSearch") }}
                <Icon icon="ep:arrow-up" :class="{close: !openSearchPanel}"/>
            </span>
        </div>
        <div class="noi-search-fields-container" :class="{close: (showResults && !openSearchPanel), 'noi-hide':details.show}">
            <form @submit.prevent="search"  autocomplete="off">
                
                <div class="container">
                    <div class="row justify-content-md-center text-center">
                        <div class="col-md-4 col-sm-12">
                            <airports-dropdown 
                                @clicked="onAirportSelect" 
                                @searchTextChanged="handleAirportSearchTextChange"
                                :clusters="sortedAirportsClusters" 
                                name="airportPicker" 
                                :placeholder="$t('search.airport')">
                            </airports-dropdown>
                        </div>
                            
                        <div class="col-md-2 col-sm-12 form-floating">
                            <period-dropdown 
                                @clicked="onPeriodSelect" 
                                :elements="periods" 
                                name="periodPicker" 
                                :placeholder="$t('search.period')">
                            </period-dropdown>
                        </div>

                        <div class="col-md-2 col-sm-12">
                            <input type="submit" class="button" value="View details" />
                        </div>
                    </div>
                </div>

            </form>
        </div>
        
        <!-- results container -->
        <div class="noi-results-container" v-if="showResults || details.show">
            
            <!-- details -->
            <details-overlay v-if="details.show" :flight="details.flight" :options="options" :device="device"></details-overlay>

            <!-- results list -->
            <div v-if="showResults" :class="{'noi-hide': details.show}">
                <div class="noi-loader-results-container" v-if="loadingResults">
                    <div class="text-center">
                        <div class="spinner-grow" role="status" style="width: 3rem; height: 3rem;">
                            <span class="visually-hidden">
                                {{ $t("loading") }}...
                            </span>
                        </div>
                    </div>
                </div>
                <div v-else>
        
                    <!-- top calendar bar -->
                    <div class="noi-top-navigator-container">
                        <div class="container-flex">
                            <div class="noi-horizontal-calendar-container">
                                <hooper 
                                    ref="hooper"
                                    :itemsToShow="4" 
                                    :centerMode="false"
                                    :wheelControl="false"
                                    :initialSlide="this.getTimeInterval">
                                        <slide v-for="(el, index) in calendarElements" :key="index" class="noi-horizontal-calendar-element" :class="{selected: selectedTimeIntervalIndex == index, day:period == 'day'}">
                                            <div @click="handleCalendarElementClick(index,el)" >{{ el.label }} <span>{{ el.subLabel }}</span></div>
                                        </slide>
                                    <hooper-navigation slot="hooper-addons" class="lg-only"></hooper-navigation>
                                </hooper>
                            </div>
                        </div>
                    </div>

                    <!-- mobile change search link -->
                    <div class="noi-change-search sm-only" :class="{'noi-hide': (!showResults || details.show)}">
                        <span @click="mobileChangeSearch()">
                            {{ $t("changeSearch") }} 
                            <Icon icon="ep:arrow-up" :class="{close: !openSearchPanel}"/>
                        </span>
                    </div>
        
                    <!-- results -->
                    <div class="container-flex noi-flights-results-list" v-if="lastFlights.length > 0">
                        <div class="row noi-flights-results-row" 
                            v-for="flight in lastFlights" :key="flight.key" :class="flight.flightInfo['colorClass']"
                            @click="evaluateMobileClick(flight)"  >
                            
                            <!-- desktop -->
                            <div class="col-md-2 lg-only">
                                <div class="row">
                                    <div class="col-md-3 noi-flight-row-status-badge">
                                        <Icon :icon="flight.flightInfo['iconClass']" :class="flight.flightInfo['colorClass']" />
                                    </div>
                                    <div class="col-md-9 noi-flight-row-logo">
                                        <a :href="airlineLink(flight)" target="_blank" title="Skyalps Home">
                                            <img src="https://third-party.opendatahub.com/webcomp-flightdata-realtime/src/assets/icons/skyalpsl-full-alt.png" />
                                        </a><br/>
                                        <!-- <span>{{ $t("operatedBy") }} XXXXXX</span> -->
                                    </div>
                                </div>
                            </div>
                            <div class="col-md-6 lg-only">
                                <div class="row">
                                    <div class="col-md-3 noi-flight-row-airport departure">
                                        <div><div class="noi-flight-row-badge">{{ flight.departure.airport.iataCode }}</div> <span>{{ flight.departure.airport.name }}</span></div><br/>
                                        <div><span class="date">{{ asZoneDate(flight.departure.date, "UTC", flight.departure.time) }}, </span><span class="hour">{{ asZoneTime(flight.departure.time) }}</span></div>
                                    </div>
                                    <div class="col-md-6 container noi-flight-row-path">
                                        <div class="noi-flight-row-plane-line" :class="flight.flightInfo['planeLineClass']">
                                            <div class="before"></div>
                                            <div class="plane">
                                                <Icon icon="material-symbols:flight" rotate="90deg" />
                                            </div>
                                            <div class="after"></div>
                                        </div>
                                        <span>{{ $t("timeLeft") }}: {{ getShownTimeLeft(flight) }} {{ $t("min") }}</span>
                                    </div>
                                    <div class="col-md-3 noi-flight-row-airport arrival">
                                        <div><div class="noi-flight-row-badge">{{ flight.arrival.airport.iataCode }}</div> <span>{{ flight.arrival.airport.name }}</span></div><br/>
                                        <div><span class="date">{{ asZoneDate(flight.arrival.date, "UTC", flight.arrival.time) }}, </span><span class="hour">{{ asZoneTime(flight.arrival.time) }}</span></div>
                                    </div>
                                </div>
                            </div>
                            <div class="col-md-4 lg-only">
                                <div class="row">
                                    <div class="col-md-3 noi-flight-row-flight-number">
                                        {{ flight.flightNumber }}
                                    </div>
                                    <div class="col-md-4 noi-flight-row-status">
                                        <span class="status" :class="flight.flightInfo['colorClass']">{{ $t("status."+flight.flightInfo["text"]) }}</span><br/>
                                        <!-- <span class="details">{{ $t("arrived") }} XX:YY</span> -->
                                    </div>
                                    <div class="col-md-5 noi-flight-row-button">
                                        <div @click="openDetails(flight)" class="button light">{{ $t("moreDetails") }}</div>
                                    </div>
                                </div>
                            </div>


                            <!-- mobile -->
                            <div class="col sm-only">
                                <div class="noi-flight-row-airport"> {{ $t("from") }} {{ flight.departure.airport.name }} {{ $t("to") }} {{ flight.arrival.airport.name }}</div>
                                <div><span class="hour">{{ asZoneTime(flight.departure.time) }} - {{ asZoneTime(flight.arrival.time) }}</span></div>
                                <div class="noi-flight-row-status">
                                    <span class="status" :class="flight.flightInfo['colorClass']">{{ $t("status."+flight.flightInfo["text"]) }}</span>
                                    <span class="message">{{ $t("estimetedArrival") }} XX:YY</span>
                                </div>
                            </div>

                            <div class="col-3 noi-flight-row-flight-number sm-only">
                                <Icon v-if="flight.flightInfo.text == 'IN_FLIGHT'" icon="material-symbols:flight" rotate="90deg" /> {{ flight.flightNumber }}
                            </div>
                        </div>
                    </div>
                    <div v-else>
                        Nessun volo trovato.
                    </div>
                    
        
                </div>
            </div>
            
        </div>
    </div>


    <!-- footer -->
    <div class="footer">
        Powered by Open Data Hub <a href="https://opendatahub.com" target="_blank"><img :src="require('@/assets/icons/odh.png')" height="25px" style="display: inline-block; margin-left: 10px" /></a>
    </div>
    


  </div>
</template>

<script>
import axios from "axios"
import { DateTime, Settings } from "luxon"

import Vue, { ref } from "vue"
import { Icon } from '@iconify/vue2';
    
import AirportsDropdown from './AirportsDropdown.vue';
import PeriodDropdown from './PeriodDropdown.vue';
import DetailsOverlay from './DetailsOverlay.vue';
    
import {Hooper, Slide,Navigation as HooperNavigation} from 'hooper';

export default {
   components:{AirportsDropdown,PeriodDropdown,Hooper,HooperNavigation, Slide,Icon,DetailsOverlay},
  name: "NoiFlightControl",
  //component props
  props: {
    options: Object,
    default: () => {

    }
  },

  //components methods
  methods: {
    handleViewportResize: function () {
        setTimeout(()=>{
            const wi = this.$refs.viewport.clientWidth
            this.device = "desktop"
            if (wi < 768) this.device = "tablet"
            if (wi < 480) this.device = "smartphone"
      
            console.log("wi",wi)
            console.log("wi",wi)
            console.log("wi",wi)
            console.log("this.device",this.device)
        })
    },
    closeDetails(){
        this.details.flight = {};
        this.details.show = false;
    },
    evaluateMobileClick(flight){
        if(this.device != 'desktop'){
            this.openDetails(flight);
        }
    },
    openDetails(flight){
        this.details.flight = flight;
        this.details.show = true;
    },
    mobileChangeSearch(){
        this.showResults = false;
    },
    toggleSearchPanel(){
        this.openSearchPanel = !this.openSearchPanel
    },
    handleCalendarElementClick(index,el){
        this.selectedTimeIntervalIndex = index;
        this.search();
    },
    handleAirportSearchTextChange(val){
        this.airportSelectFieldValue = val;
    },
    onAirportSelect(el){
        console.log(el)
        this.airport = el;
        console.log("EXTERNAL departure HANDLER",'value: ' + el.value, ' label: ' + el.label);
    },
    onPeriodSelect(el){
        this.period = el;

        this.selectedTimeIntervalIndex = this.getDefaultTimeInterval;
        const hop = this.$refs.hooper
        hop.slideTo(this.selectedTimeIntervalIndex);

        console.log("EXTERNAL period HANDLER",'value: ' + el.value, ' label: ' + el.label);
    },
    airlineLink(flight) {
        let dep = DateTime.fromFormat(flight.departure.date, "yyyy-LL-dd", "UTC")
        let loc = flight.departure.airport.iataCode + "-" + flight.arrival.airport.iataCode
        let link = "https://booking.skyalps.com/flight-results/" + loc + "/" + dep.toFormat("yyyy-LL-dd") + "/NA/1/0/0"
        return link
    },
    asZoneTime(time = "00:00", source_zone = "UTC") {
        if (time == "") return ""
        let datetime = DateTime.fromFormat(time, "T", {
            zone: source_zone
        })
        datetime = datetime.setZone(this.current_timezone)
        return datetime.toFormat("HH:mm")
    },
    asZoneDate(date = "2022-01-01", source_zone = "UTC", time = "00:00",outputFormat = "EEE, dd MMM") {
        if (date == "") return ""
        let datetime = DateTime.fromFormat(date + " " + time, "yyyy-LL-dd T", {
            zone: source_zone
        })
        datetime = datetime.setZone(this.current_timezone)
        return datetime.toFormat(outputFormat)
    },
    
    updateTime() {
        let time = DateTime.utc()
        time = time.setZone(this.current_timezone)
        this.time = time
    },
    getShownTimeLeft(flight){
        let d = new Date();
        let now = d.getTime()/1000;
        let totalMinutesTime = Math.round((flight.arrival.timestamp-flight.departure.timestamp)/60);
        let minutesLeft =  totalMinutesTime;
        if(now > flight.departure.timestamp){
            if(now < flight.arrival.timestamp){
                let elapsedMinutes = Math.round((now-flight.departure.timestamp)/60);
                if(elapsedMinutes > 0){
                    minutesLeft -= elapsedMinutes;
                }
            }else{
                minutesLeft = '-';
            }
        }
        return minutesLeft;
    },

    // resolve a message request
    $t: function (messageKey, fallbackMessage) {
        let els = messageKey.split(".");
        let langPack = this.currentLangPack
        for (var i = 0; i < els.length; i++) {
            var key = els[i];
            langPack = langPack[key];

            if (langPack === undefined) {
                if (fallbackMessage) {
                    return fallbackMessage;
                } else {
                    return "[" + messageKey + "]";
                }
            }
        }
        return langPack;
    },

    validateForm(){
        let airport = this.airport;
        let airportSelectFieldValue = this.airportSelectFieldValue;
        let period = this.period;

        let errors = []

        if(errors.length > 0){
            return {
                result:false,
                errors:errors
            }
        }else{
            return {
                result:true,
                data:{
                    airportSelectFieldValue:airportSelectFieldValue,
                    airport:airport,
                    period:period
                }
            }
        }
    },

    async search(){
        console.log("airportSelectFieldValue",this.airportSelectFieldValue)

        //form validation
        let formValidationResponse = this.validateForm();
        if(formValidationResponse.result == false){
            let validatedFormErrors = formValidationResponse.errors;
            console.log("validatedFormErrors",validatedFormErrors)
            return false;
        }

        //set time and update status
        this.updateTime()
        this.showResults = true;
        this.openSearchPanel = false;
        this.loadingResults = true;
        
        //get data
        await this.fetchSchedules();

        //set staus
        this.loadingResults = false;
    },
    addWhereCondition: function(where,condition){
        if(where != ""){
            where += ","
        }
        return where += condition;
    },  
    parseData: function(data){
        let parsedData = data.data.data.map((o) => {
            let datetimeDeparture = DateTime.fromFormat( o.smetadata.fltsfromperiod + " " + o.smetadata.std, "yyyy-LL-dd T",{ zone: "UTC" })
            let datetimeArrival = DateTime.fromFormat( o.smetadata.fltstoperiod + " " + o.smetadata.sta, "yyyy-LL-dd T",{ zone: "UTC" })

            let rate = o.smetadata.fares ? o.smetadata.fares["SKY_LIGHT"] : null

            if (rate) {
            rate =
                rate.fare.adultFareOW +
                rate.fare.tax1OW +
                rate.fare.tax2OW +
                rate.fare.tax3OW +
                rate.fare.tax4OW
            }

            if (isNaN(rate) || rate == 0) rate = false

            //check for airports map
            if(!this.airports[o.smetadata.fromdestination]){
                console.log("Error: any departure airport named '"+o.smetadata.fromdestination+"' has been found in mapping.")
            }
            if(!this.airports[o.smetadata.todestination]){
                console.log("Error: any arrival airport named '"+o.smetadata.todestination+"' has been found in mapping.")
            }

            let obj = {
                departure:{
                    date:o.smetadata.fltsfromperiod.replace(/\//g, "-"),
                    time:o.smetadata.std,
                    timestamp:o.smetadata.departure_timestamp,
                    airport:{
                        iataCode:o.smetadata.fromdestination,
                        name:this.airports[o.smetadata.fromdestination].name,
                        coordinates:this.airports[o.smetadata.fromdestination].pos
                    }
                },
                arrival:{
                    date:o.smetadata.fltstoperiod.replace(/\//g, "-"),
                    time:o.smetadata.sta,
                    timestamp:o.smetadata.arrival_timestamp,
                    airport:{
                        iataCode:o.smetadata.todestination,
                        name:this.airports[o.smetadata.todestination].name,
                        coordinates:this.airports[o.smetadata.todestination].pos
                    }
                },
                status: o.smetadata.remark ? o.smetadata.remark : "SCHEDULED",
                company: o.sorigin,
                flightNumber: o.smetadata.fltnumber,
                rates: rate ? { basic_adult_oneway_withtaxes: rate } : null
            }

            let flightInfo = this.getFlightInfo(obj);
            obj.flightInfo = flightInfo;

            return obj;
        })

        // sort by timestamps
        parsedData.sort(this.sortByTimestamp)

        // group by key, keep newest entry/key
        let sflights = {}

        //client side parsing
        for (let i = 0; i < parsedData.length; i++) {
            let arr = parsedData[i]
            const key = arr.departure.date + "-" + arr.flightNumber + "-" + arr.arrival + "-" + arr.departure
            if (typeof sflights[key] == "undefined") sflights[key] = []
            sflights[key] = arr
        }

        //copy obj due to decouple
        parsedData = Object.values(sflights).sort(this.sortByDateTime)
        return parsedData
    },
    async getData(whereCondition){
        //compile get parameters
        let params = {
            limit: "-1",
            offset: "0",
            shownull: "false",
            distinct: "true",
            where: whereCondition,
            origin: "webcomp-flightdata"
        }
        params = new URLSearchParams(params).toString()
        
        //make request
        let data = await axios.get(this.options.restEndpoint + params)
        console.log("getData",params,data)
        return data;
    },
    async fetchSchedules() {
        try {
            // getting outward flights
            //adding where conditions
            let where = ""
            if(this.airport && this.airport.value){
                let els = this.airport.value.split("|");
                let airportOrCondition = "";
                if(els.length > 1){
                    airportOrCondition += "or("
                    for(let i = 0; i<els.length; i++){
                        if(i > 0){
                            airportOrCondition += ","
                        }
                        airportOrCondition += "smetadata.fromdestination.eq."+els[i]+",smetadata.todestination.eq."+els[i]
                    }
                    airportOrCondition += ")"
                }else{
                    airportOrCondition = "or(smetadata.fromdestination.eq."+this.airport.value+",smetadata.todestination.eq."+this.airport.value+")";
                }
                where = this.addWhereCondition(where,airportOrCondition);
            }else{
                if(!this.airport || (this.airport.label != this.airportSelectFieldValue)){
                    // check if are there a flight number
                    let airportSelectFieldValue = this.airportSelectFieldValue;
                    console.log("airportSelectFieldValue",airportSelectFieldValue)
                    if(airportSelectFieldValue){
                        where = this.addWhereCondition(where,"smetadata.fltnumber.eq."+airportSelectFieldValue);
                    }
                }
            }
            where = this.addWhereCondition(where,"or(smetadata.fromdestination.eq."+this.localAirport.key+",smetadata.todestination.eq."+this.localAirport.key+")");

            // handle dates
            let calendarElement = this.calendarElements[this.selectedTimeIntervalIndex];
            if(calendarElement){
                let dateLowerLimit = new Date(calendarElement.minDate)
                let dateUpperLimit = new Date(calendarElement.maxDate);
                dateUpperLimit.setDate(dateUpperLimit.getDate() + 1);
                where = this.addWhereCondition(where,"smetadata.departure_timestamp.gt."+dateLowerLimit.getTime()/1000);
                where = this.addWhereCondition(where,"smetadata.departure_timestamp.lt."+dateUpperLimit.getTime()/1000);
            }else{
                let now = new Date();
                where = this.addWhereCondition(where,"smetadata.departure_timestamp.gt."+now.getTime()/1000); //TDO: decide how to handle it.
            }

            where = "and("+where+")"

            let data = await this.getData(where);

            //parse received data
            this.flights = this.parseData(data)

        } catch (error) {
            console.error(error)
        }
    },
    sortByTimestamp: function (a, b) {
        if (a.timestamp < b.timestamp) {
            return -1
        }
        if (a.timestamp > b.timestamp) {
            return 1
        }
        return 0
    },
    sortByDateTime: function (a, b) {
        const adate = a.departure.date + "/" + a.departure.time
        const bdate = b.departure.date + "/" + b.departure.time
        let adatetime = DateTime.fromFormat(adate, "yyyy-LL-dd/H:mm", {zone: "UTC"})
        let bdatetime = DateTime.fromFormat(bdate, "yyyy-LL-dd/H:mm", {zone: "UTC"})

        if (adatetime < bdatetime) {
            return -1
        }
        if (adatetime > bdatetime) {
            return 1
        }
        return 0
    },


    getFlightInfo(data) {
        let arrivalDate = DateTime.fromMillis(data.arrival.timestamp * 1000, {zone: this.current_timezone})
        let departureDate = DateTime.fromMillis(data.departure.timestamp * 1000, {zone: this.current_timezone})
        const arrival_diff = arrivalDate.diff(this.time, ["minutes"]).toObject()["minutes"]
        const departure_diff = departureDate.diff(this.time, ["minutes"]).toObject()["minutes"]

        // landed
        if (arrival_diff < 0) {
            return {
                text: "LANDED",
                planeLineClass:"end",
                iconClass:"mdi:check-circle-outline",
                colorClass: "gray"
            }
        }

        // in flight
        if (arrival_diff > 0 && departure_diff < 0) {
            return {
                text: "IN_FLIGHT",
                planeLineClass:"middle",
                iconClass:"mdi:check-circle-outline",
                colorClass: "green"
            }
        }
        
        // boarding => departure - 30m
        if (departure_diff < 30) {
            return {
                text: "BOARDING",
                planeLineClass:"start",
                iconClass:"mdi:check-circle-outline",
                colorClass: "green"
            }
        }

        // scheduled
        if (departure_diff > 0) {
            return {
                text: "SCHEDULED",
                planeLineClass:"start",
                iconClass:"mdi:check-circle-outline",
                colorClass: "gray"
            }
        }

        // TODO: realtime data endpoints
        // ontime
        // delayed
        // cancelled


        // return {
        //     text: "ON TIME",
        //     planeLineClass:"start",
        //     iconClass:"mdi:check-circle-outline",
        //     colorClass: "green"
        // }
        
        // return {
        //     text: "DELAYED",
        //     planeLineClass:"start",
        //     iconClass:"mdi:alert-circle-outline",
        //     colorClass: "yellow"
        // }

        // return {
        //     text: "CANCELLED",
        //     planeLineClass:"start",
        //     iconClass:"mdi:close-circle-outline",
        //     colorClass: "red"
        // }

        return {
            text: "",
            planeLineClass:"start",
            iconClass:"mdi:check-circle-outline",
            colorClass: "yellow"
        }
    }
  },

  //computed calculations
  computed: {
    getTimeInterval(){
        if(this.selectedTimeIntervalIndex){
            return this.selectedTimeIntervalIndex;
        }
        
        return this.getDefaultTimeInterval;
    },
    getDefaultTimeInterval(){        
        if(!this.clusterNumber[this.period.value]){
            return 0;
        }

        return Math.round(this.clusterNumber[this.period.value]/2)+1;
    },
    calendarElements(){
        let today = DateTime.now().startOf('day');
        let els = []
        switch(this.period.value){
            case 'month':
                for(let i = -Math.round(this.clusterNumber.month/2)-1; i<Math.round(this.clusterNumber.month/2); i++){
                    let minDate = today.plus({ days: 30*i });
                    let maxDate = minDate.plus({ days: 29 });
                    let el = {
                        label:minDate.toFormat('dd')+" - "+maxDate.toFormat('dd'),
                        subLabel:maxDate.toFormat('MMM'),
                        minDate:minDate,
                        maxDate:maxDate
                    }
                    els.push(el)
                }
                break;
            case 'week':
                for(let i = -Math.round(this.clusterNumber.week/2)-1; i<Math.round(this.clusterNumber.week/2); i++){
                    let minDate = today.plus({ days: 7*i });
                    let maxDate = minDate.plus({ days: 6 });
                    let el = {
                        label:minDate.toFormat('dd')+" - "+maxDate.toFormat('dd'),
                        subLabel:maxDate.toFormat('MMM'),
                        minDate:minDate,
                        maxDate:maxDate
                    }
                    els.push(el)
                }
                break;
            case 'day':
            default:
                for(let i = -Math.round(this.clusterNumber.day/2)-1; i<Math.round(this.clusterNumber.day/2); i++){
                    let minDate = today.plus({ days: 1*i });
                    let maxDate = minDate.plus({ days: 0 });
                    let el = {
                        label:minDate.toFormat('dd'),
                        subLabel:minDate.toFormat('MMM'),
                        minDate:minDate,
                        maxDate:maxDate
                    }
                    els.push(el)
                }
                break;
        }
        
        return els;
    },
    periods(){
        let periodsMap = [
            {
                value:'day'
            },
            {
                value:'week'
            },
            {
                value:'month'
            }
        ]
        
        return periodsMap.map((period)=>{
            return {
                label:this.$t("periods."+period.value),
                value:period.value
            }
        })
    },
    lastFlights() {
        return this.flights
    },
    sortedAirports() {
        let sortedAirports = []
        Object.keys(this.airports).map((key)=>{
            let o = this.airports[key];
            o.key = key
            sortedAirports.push(o)
        });
        
        sortedAirports = sortedAirports.sort((a,b)=>{
            if(a.name<b.name) return -1;
            if(b.name<a.name) return 1;
            else return 0;
        })
        return sortedAirports;
    },
    sortedAirportsClusters() {
        let sortedAirportsClusters = []
        this.airportsClusters.sort((a,b)=>{
            if(a.name<b.name) return -1;
            if(b.name<a.name) return 1;
            else return 0;
        }).map((el)=>{
            let o = {
                label:el.name+' (All)',
                value:el.airports.join('|'),
                airports:[]
            }

            o.airports = el.airports.sort((a,b)=>{
                if(a.name<b.name) return -1;
                if(b.name<a.name) return 1;
                else return 0;
            }).map((airportKey)=>{
                return {
                    value:airportKey,
                    label:this.airports[airportKey].name+' ('+airportKey+')'
                };
            });

            sortedAirportsClusters.push(o)
        });
        
        return sortedAirportsClusters;
    }
  },

  //on mount component
  mounted: function () {
    //init local airport
    let key = this.options.localAirport;
    if(!this.airports[key]){
        key = 'BZO';
    }
    this.localAirport = this.airports[key];
    this.localAirport.key = key


    //init periods
    this.selectedTimeIntervalIndex = this.getDefaultTimeInterval;

    //init current language
    this.currentLanguage = "en";
    if(this.options.language){
        this.currentLanguage = this.options.language;
    }
    
    //init current langPack
    let fullLangPack = require("../mappings/languages.json");
    if (this.options.langPack){
        fullLangPack = { ...fullLangPack, ...this.options.langPack }
    }
    if(fullLangPack[this.currentLanguage]){
        this.currentLangPack = fullLangPack[this.currentLanguage]
    }else{
        this.currentLangPack = fullLangPack['en']
    }

    Settings.defaultLocale = this.currentLanguage

    //init timezone
    this.current_timezone = this.options.timezone
        
    this.roundTripCheck = 1;
    this.handleViewportResize();
  },

  //component whatcers
  watch: {
  },

  //on component has been created
  created: async function () {
    this.updateTime()
    this.time = this.time.setZone(this.current_timezone)

    window.addEventListener("resize", this.handleViewportResize)
  },
  destroyed() {
    
  },
  data() {
      return {
        clusterNumber:{
            'day':20,
            'week':12,
            'month':6
        },
        period:{label:'Day',value:'day'},
        selectedTimeIntervalIndex:0,

        localAirport:{name:"Bozen",key:'BZO'},
        airport:ref(),
        airportSelectFieldValue:'',
        details:{
            flight:{},
            show:false
        },
        flights: [],
        
        showResults: false,
        openSearchPanel: true,
        loadingResults: false,

        device: "desktop",
        current_timezone: "",
        time: "00:00:00 UTC",
        currentLangPack:{},
        airports: require("../mappings/airports.js"),
        airportsClusters: require("../mappings/airportsClusters.js")
    }
  }
}
</script>

<style lang="scss">
@import "~bootstrap/scss/bootstrap.scss";
@import "~vuelayers/dist/vuelayers.min.css";
@import "../css/hooper.css";


.noi-flightdata-realitime {

    // skyapls theme
    &.skyalps {
        --noi-primary: #a1bad4;
        --noi-mid: #a1bad4;
        --noi-secondary: #004988;
        --noi-text-primary: black;
        --noi-text-secondary: white;
        .noi-top-navigator-container {
            color:white;
            .noi-horizontal-calendar-container{
                .hooper-navigation{
                    fill:white;
                }
            } 
        }
        .noi-details-container {
            .noi-details-description-container{
                .noi-details-description-container-top{
                    .noi-details-from-to-title{
                        color:#004988;
                    }
                }
            }
        }

        &.smartphone, &.tablet{
            .noi-top-navigator-container div{
                color:white;
            }
            .noi-results-container{                
                .noi-flights-results-list{
                    .noi-flights-results-row{
                        .noi-flight-row-airport{
                            color:#004988;
                        }
                    }
                }
            }
        }
    }


    font-family: var(--noi-font-family, Verdana);
    font-size: var(--basic-font-size, 16px);
    color:var(--noi-text-primary,black);
    background-color: white;

    a {
        text-decoration: none;
        color: var(--noi-primary, #0068b4);
    }

    h1,h2,h3,h4,h5,h6 {
        color: var(--noi-text-primary, #000000);
    }

    h1 {
        font-size: var(--h1-font-size, 3rem);
    }

    h2 {
        font-size: var(--h2-font-size, 2rem);

        font-weight: bold;
        line-height: 2.5rem;
        margin-bottom: 0.125rem;
        padding-top: 0.25rem;

        &.dark {
            background: var(--noi-jam-strong, #e4c200);
        }

        .iconbg {
            display: inline-block;
            background: black;
            border-radius: 0.3125rem;
            padding: 0.25rem;
        }
    }

    h3 {
        font-size: var(--h3-font-size, 2rem);
        background: var(--noi-jam-light, #6e6e6d);
    }



    .dark {
        img {
            -webkit-filter: invert(1);
            filter: invert(1);
        }
    }


    .noi-hide{
        display: none;
    }

    .load-more {
        cursor: pointer;

        td {
            color: black !important;
            background-color: var(--noi-jam-strong, #e4c200) !important;
        }
    }

    .sm-only {
        display: none;
    }
    .lg-only {
        display: block;
    }

    .noi-top-back-icon{
        position: absolute;
        left: 1rem;
        font-size: 1.4rem;
        top: 1.15rem;
    }


    /* MOBILE */
    &.smartphone, &.tablet {
        .sm-only {
            display: block;
        }
        .lg-only {
            display: none;
        }

        .noi-description-bar{
            padding:1rem 0 0 0;
        }
        .noi-search-fields-container{
            padding:0;
            margin-bottom: 1rem;

            .col-sm-12{
                padding-bottom:1rem;
            }
        }

        .noi-content-container{
            width: 100%;
            margin: 0;

            .noi-top-navigator-container{
                height: 4rem;
                margin: 0;
                color: var(--noi-text-primary, black);
                background-color: var(--noi-primary, #ffffff);

                .noi-horizontal-calendar-container{
                    width:100%;

                    .noi-horizontal-calendar-element{
                        height: 4rem;
                        width: min-content;
                        padding: 0.53rem 0;
                        
                        &.day{
                            padding: 0.53rem 1rem;
                        }

                        span{
                            font-size: 1.1rem;
                            display: block;
                        }
                    }
                }
            }

            .noi-results-container{
                background-color: #f0f0f0;
                
                .noi-flights-results-list{
                    .noi-flights-results-row{
                        text-align: left;
                        border-left: 0.625rem solid;
                        background-color: #ffffff;
                        margin: 1rem;
                        position: relative;
                        
                        &.green{
                            border-left-color: green;
                        }
                        &.gray{
                            border-left-color: gray;
                        }
                        &.yellow{
                            border-left-color:gold;
                        }
                        &.red{
                            border-left-color: red;
                        }

                        .noi-flight-row-airport{
                            font-weight: bold;
                        }
                        span.hour{
                            font-size: 0.8rem;
                        }
                        .noi-flight-row-status{
                            padding:0.5rem 0;

                            span{
                                display: inline;
                            }
                            .status{
                                font-size: 0.9rem;
                            }
                            .message{
                                font-weight: normal;
                                font-size: 0.8rem;
                                padding-left: 0.6rem;
                            }
                        }

                        .noi-flight-row-flight-number{
                            text-align: right;
                            font-weight: normal;
                            padding-top: 0;
                            font-size: 0.75rem;
                        }
                    }
                }

            }
            
            .hooper {
                height: 4rem;
            }
        }
    }


    
    /* DESKTOP - DEFAULT */
    .noi-top-bar{
        background-color: #F5F4F6;
        text-align: center;

        img{
            margin: 0.25rem;
            width: auto;
            height: 3rem;
        }
    }
    .noi-description-bar{
        text-align: center;
        padding: 1rem 1rem 0 1rem;
    }
    .noi-details-back{
        margin:3rem 0 1rem 0;
        text-align: left;
        font-size: 1rem;
        font-weight: 500;

        span{
            cursor: pointer;
        }
    }

    .noi-content-container{
        width: 90%;
        margin: 0 auto;
    }

    .noi-change-search{
        text-align: center;
        width: 100%;
        padding:2rem 0 1rem 0;

        span {
            cursor: pointer;
            font-weight: bold;

            .close{
                transform: rotate(180deg);
            }
        }
    }
    .noi-search-fields-container {    
        color: var(--noi-text-primary, black);
        width: 100%;
        background-color: var(--noi-primary-contrast, #ffffff);
        padding:1rem 1.5rem 0 1.5rem;
        margin-bottom: 2rem;

        
        &.close{
            // transition:opacity 0.3s ease-out, height 0.8s ease-out, display 0.8s ease-out;
            // opacity: 0;
            display: none;
        }

        .col{
            padding:0 0.625rem;
        }

        input{
            border: 1px solid #ced4da;
            border-radius: 0.25rem;
        }

        option.cluster{
            color:  blue;
        }
    }

    .noi-results-container {
        position: relative;
        color: var(--noi-text-primary, #000000);
        width: 100%;
        background-color: var(--noi-primary-contrast, #ffffff);
        height: 100%;
        overflow-x: hidden;
    }     
    
    .noi-top-navigator-container{
        height: 3rem;
        color:var(--noi-text-primary,black);
        background-color: var(--noi-mid, #f5f5f5);
        width: 100%;
        margin:1rem 0;

        .noi-horizontal-calendar-container{
            width:50%;
            margin:0 auto;

            .noi-horizontal-calendar-element{
                text-align: center;
                font-size: 1.5rem;
                font-weight: bold;
                padding: 0.53rem 0;
                cursor: pointer;
                height: 3rem;
    
                &:hover, &.selected{
                    color:var(--noi-text-secondary,white);
                    background-color: var(--noi-secondary, black);
                }

                span{
                    font-size: 1.2rem;
                    padding-left: 0.2rem;
                }
            }
    }
    }

    .noi-flights-results-list{

        max-height: 500px;
        overflow-x: hidden;
        overflow-y: auto;
        
        .noi-flights-results-row{
            text-align: center;
            color: #000000;
            padding: 1rem 0;
            margin:0 2rem;
            vertical-align: middle;

            .green{
                color: green;
            }
            .gray{
                color: gray;
            }
            .yellow{
                color:gold;
            }
            .red{
                color: red;
            }

            .noi-flight-row-status-badge{
                text-align: left;
                font-size: 2rem;
            }
            .noi-flight-row-logo{
                img{
                    max-width: 110px;
                }
                span{
                    font-weight: bold;
                    font-size: 0.9rem;
                }
            }
            .noi-flight-row-airport{

                span{
                    font-weight: bold;
                    &.date{
                        color:#6e6e6d;
                        font-weight:normal;
                    }
                    &.hour{
                        
                    }
                }

                div{
                    display: inline-block;
                }

                div.noi-flight-row-badge{
                    background-color: #f5f5f5;
                    border-radius: 0.25rem;
                    width: 2rem;
                    height: 1.3125rem;
                    font-size: 0.725rem;
                    padding-top: 0.2rem;
                    font-weight: bold;
                }

                &.departure{
                    
                }
                &.arrival{

                }

            }
            .noi-flight-row-path{

                .noi-flight-row-plane-line{
                    margin: 1rem 0;
                    position: relative;

                    div.before,div.after{
                        height: 0.125rem;
                        position: absolute;
                        padding:0;
                        margin: 0;
                    }

                    div.before{
                        border-top:0.125rem solid var(--noi-secondary, black);
                    }
                    div.plane{
                        font-size: 1.5rem;
                        background: white;
                        padding: 0 0.25rem;
                        margin: -0.9rem 0;
                        position: absolute;
                        width: 2rem;
                        color:var(--noi-secondary, black);
                        z-index: 2;
                        background-color: white;
                        line-height: 1.5rem;
                    }
                    div.after{
                        border-top:0.125rem solid var(--noi-mid, #f5f5f5);
                    }

                    &.start{
                        div.before{
                            width: 0;
                        }
                        div.plane{
                            left:0;
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
                            left:calc(100% - 2rem);
                        }
                        div.after{
                            width: 0;
                        }
                    }
                }
                span{
                    font-weight: bold;
                    font-size: 0.9rem;
                    position: relative;
                    top:0.5rem;
                }
            }
            .noi-flight-row-flight-number{
                font-weight: bold;
                padding-top: 0.75rem;
            }
            .noi-flight-row-status{
                font-weight: bold;

                .status{
                    font-size: 1.2rem;

                }
                .details{
                    font-size: 0.7rem;
                    position: relative;
                    top: -0.375rem;
                }
            }
            .noi-flight-row-button{
                text-align: right;
                font-weight: bold;
            }
        }
    }

    .noi-loader-results-container{
        color: rgba(0, 0, 0, 0.5);
        margin:50px 0;
    }

    //buttons
    .button {
        background-color: var(--noi-secondary, black);
        border: none;
        color: white;
        cursor: pointer;
        padding: 1rem 2rem;
        text-align: center;
        text-decoration: none;
        display: inline-block;
        font-size: 1rem;
        font-weight: bold;
        width:100%;
    
        &.light{
            background-color: white;
            border: 1px solid var(--noi-secondary, gray);
            border-radius: 0.25rem;
            color: var(--noi-secondary, gray);
        }
    }

    .footer {
        // position:sticky;
        // bottom:0;
        // left:0;
        background-color: #f5f5f5;
        width: 100%;
        text-align: right;
        font-size: 1rem;
        padding: 0.5rem 1.5rem;
        font-size: 0.8rem;
    }

}
</style>