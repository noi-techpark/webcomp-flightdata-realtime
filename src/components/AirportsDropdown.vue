<template>
    <div class="form-floating">
        <input  class="form-control" 
                :name="name" 
                type="text"
                ref="passengers"
                autocomplete="off"
                v-model="searchText" 
                @focus="onFocus"
                @blur="onBlur"
                placeholder="Search something..." >

        <div class="airport-dropdown-container text-left" v-bind:class="classObject"> <!--@blur="onBlur"  -->
            
            <div class="airport-dropdown-element" @click="selectAirport(null)">
                <div class="row ">
                    <span class="col text-start">{{ $parent.$t("allAirports") }}</span> 
                    <span class="col text-end"></span>
                </div>
            </div>
            
            <div v-for="(cluster, index) in filteredClusters" class="airport-dropdown-cluster">
                <div class="airport-dropdown-element clusterParent text-start" @click="selectAirport(cluster)">
                    {{ cluster.label }}
                </div>

                <div class="airport-dropdown-element" v-for="(airport, index) in cluster.airports" @click="selectAirport(airport)">
                    <div class="row ">
                        <span class="col text-start">{{ airport.label }}</span> 
                        <span class="col text-end">{{ airport.value }}</span>
                    </div>
                </div>
            </div>
        </div>
        
        <label :for="name" class="form-label">
            {{ placeholder }}
        </label>
    </div>

</template>

<script>
    
    export default {
        props: {
            name: {
                type: String,
                default: "needToBeARamdumUniqueString"
            },
            placeholder: {
                type: String,
                default: "Type here"
            },
            clusters: {
                type: [Object, Array],
                required: true
            }
        },
        data() {
            return {
                isOpen: false,
                searchText:'',
                selectedElement:{value:null,label:''}
            }
        },
        computed: {
            filteredClusters() {
                let that = this;
                let filteredClusters = JSON.parse(JSON.stringify(this.clusters));
                if(that.searchText == null){
                    return filteredClusters;
                }

                return filteredClusters.filter(function(cluster) {
                    cluster.airports = cluster.airports.filter(function(airport){
                        return (
                            airport.value.toUpperCase().includes(that.searchText.toUpperCase()) ||
                            airport.label.toUpperCase().includes(that.searchText.toUpperCase())
                        )
                    });
                    
                    if(cluster.airports.length > 0){
                        return true;
                    }else{
                        return false;
                    }
                })
            },
            classObject() {
                return {
                    'open': this.isOpen && (this.filteredClusters.length > 0)
                }
            },
        },

        watch: {
            'searchText': function(val){
                if(val == ''){
                    this.selectedElement = { 'value': null, 'label': '' };
                }
                this.$emit('searchTextChanged', val);
            },
            'selectedElement': function(val){
                this.$emit('clicked', { 'value': this.selectedElement.value, 'label': this.selectedElement.label });
            }
        },

        methods: {
            onFocus() {
                this.isOpen = true
            },
            onBlur() {
                let that = this;
                setTimeout(()=>{
                    that.isOpen = false
                },200);
            },
            selectAirport(el) {
                if (el == null){
                    el = {label:this.$parent.$t("AllAirports"),value:null};
                }

                let selectedValue = el.value
                let selectedLabel = el.label
                this.selectedElement = {
                    value:selectedValue,
                    label:selectedLabel
                }
                console.log(this.selectedElement)
                
                this.isOpen = false;
                this.searchText = selectedLabel
            }
        }

    };
</script>

<style lang="scss">

    $lightgrey: #f5f4f6;
    $grey: #e0e0e0;


    .airport-dropdown-container{
        display: none;
        border: 1px solid $grey;
        border-radius: 5px;
        background: white;
        width: calc(100% - 8px);
        height: auto;
        max-height: 30vh;
        padding: 0;
        margin: 1rem 0;
        position: absolute;
        z-index:10;
        overflow: auto;
        // &:after {
        //     content: "";
        //     position: absolute;
        //     background: black;
        //     width: 0;
        //     height: 0;
        //     /* margin-left: 0em; */
        //     /* margin-top: 0em; */
        //     left: calc(50% - 0.5rem);
        //     top: -0.5rem;
        //     box-sizing: border-box;
        //     border: 0.5rem solid black;
        //     border-color: transparent transparent #fff #fff;
        //     /* transform-origin: 50% 50%; */
        //     z-index: 9999999;
        //     transform: rotate(-45deg);
        //     /* box-shadow: 0px 0px 10px 0 rgba(0, 0, 0, 0.1); */
        // }
        .airport-dropdown-cluster{
            
        }

        .airport-dropdown-element{
            background: white;
            cursor: pointer;
            display: block;
            transition: background 0.2s ease-out;
            top:0;
            left:0;
            width:100%;
            padding: 0.8rem 1.5rem;
            
            &:hover,
            &.hover{
                background:$lightgrey;
            }
            
            &.clusterParent{
                font-weight: bold;
            }
        }
        &.open{
            display: block;
        }
    }
</style>