<!--
SPDX-FileCopyrightText: NOI Techpark <digital@noi.bz.it>

SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
    <div class="form-floating">
        <input  class="form-control" 
                :name="name" 
                type="text"
                ref="passengers"
                readonly="true"
                autocomplete="off"
                v-model="searchText" 
                @focus="onFocus"
                @blur="onBlur"
                placeholder="Search something..." >

        <div class="period-dropdown-container text-left" v-bind:class="classObject"> <!--@blur="onBlur"  -->
            <div v-for="(element, index) in elements" class="period-dropdown-cluster">
                <div class="period-dropdown-element clusterParent text-start" @click="selectElement(element)">
                    {{ element.label }}
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
            elements: {
                type: [Object, Array],
                required: true
            }
        },
        data() {
            return {
                isOpen: false,
                searchText: '',
                selectedElement:{value:null,label:''}
            }
        },
        computed: {
            classObject() {
                return {
                    'open': this.isOpen
                }
            },
        },

        watch: {
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
            selectElement(el) {
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


    .period-dropdown-container{
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
        overflow: auto;
        z-index:10;

        .period-dropdown-cluster{
            
        }

        .period-dropdown-element{
            background: white;
            color: black;
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
        }
        &.open{
            display: block;
        }
    }
</style>