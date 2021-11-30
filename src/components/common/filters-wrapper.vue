<template>
    <div class="filters-wrapper">
        <font-awesome-icon
            icon="times"
            class=closeIcon
        />
        <h1>Filters</h1>  
        <div class="filters-wrapper__line">
            <label class="range-label">
                ID
                <div class="range">
                    <span>From</span>
                    <input type="number" name="id_min" class="input number" min="1" max="500">
                    <span>To</span>
                    <input type="number" name="id_max" class="input number" min="1" max="500">
                </div>
                <font-awesome-icon
                    icon="times"
                    class=closeIcon
                />
            </label>
        </div> 
        <div class="filters-wrapper__line">
            <label>
                Post ID
                <input type="number" name="post_id" class="input number" min="1" max="100">
                <font-awesome-icon
                    icon="times"
                    class=closeIcon
                />
            </label>
        </div>
        <div class="filters-wrapper__line">
            <label>
                Email
                <input 
                    type="text" 
                    name="email" 
                    class="input" 
                    placeholder="Введите значение"
                    v-model="filter.email"
                    @input="filterText('email')"
                >
                <font-awesome-icon
                    icon="times"
                    class=closeIcon
                    @click="removeFilter('email')"
                />
            </label>
        </div>
        <div class="filters-wrapper__line">
            <label>
                Name
                <input type="text" name="name" class="input" placeholder="Введите значение">
                <font-awesome-icon
                    icon="times"
                    class=closeIcon
                />
            </label>
        </div>
    </div>
</template>
<script>
import _ from 'lodash';
// import Vue from 'vue';

export default {
    name: "FiltersWrapper",
    props: {
        staticPaging: {
            type: Boolean,
            required: true,
        },
        fetchedRows: {
            type: Array,
            required: true,
        },
        // refilter: {
        //     type: Boolean,
        //     required: true,
        // },
        requiredLength: {
            type: Number,
            default: 0,
        },
        renderedLength: {
            type: Number,
            default: 0,
        }
    },
    data() {
        return {
            filter: {
                email: "",
            },
            isSorted: false,
            filterArray: [],
            filterCount: 0,
            // filterStarted: false,
        }
    },
    created() {
        // Vue.set(this, 'filterArray', this.arrToFilter);
    },
    mounted() {
        // console.log('fetchedRows', this.fetchedRows)
    },
    computed: {
        filterArrLength() {
            return this.fetchedRows.length;
        }
    },
    watch: {
        async fetchedRows(newValue) {
            console.log('newValue: ', newValue);
            this.filterArray = await newValue;
            // console.log('filterArray: ', this.filterArray.length);
            // if (this.filterStarted) {
            //     // this.filterText();
            // }
        }
    },
    methods: {
        filterText: _.debounce(function filterText(property) {
            // this.filterStarted = true;
            // console.log('property: ', property);
            // console.log('email: ', this.filter[property]);
            this.filterCount++;
            let array = [];
            if (this.isSorted) {
                if (!this.staticPaging) {
                array = this.fetchedRows;          
                } else {
                array = this.sortedList;
                }
            } else {
                array = this.filterArray;          
                // console.log('array.length: ', array.length);
            }
            
            this.filteredList =  array.filter(row => row[property].search(this.filter[property]) > -1);
            // console.log('this.filteredList: ', this.filteredList);
            // this.$emit('filter', this.filteredList);
            this.$emit('filter', this.filteredList);
        }, 500),
        removeFilter(property) {
            // console.log('property: ', property);
            // this.filterStarted = false;
            this.filter[property] = "";
            this.$emit('remove-filter');
        },
    },
};
</script>
<style scoped>
    .filters-wrapper {
        position: fixed;
        top: 0;
        right: 0;
        z-index: 1;
        padding: 15px;
        background-color: #fff;
        min-width: 500px;
        border: 2px solid #c8cacc;
    }

    .filters-wrapper__line {
        display: flex;
        position: relative;
        margin: 10px;
    }

    .range-label {
        width: 100%;
        display: flex;
        align-items: center;
    }


    .range {
        margin-left: 20px;
    }

    .input {
        padding: 10px;
        margin: 10px;
    }

    .filterIcon {
    margin-left: auto;
    margin-right: 8px;
  }

  .filterIcon:hover {
    cursor: pointer;
  }

  .closeIcon {
    position: absolute;
    top: 16px;
    right: 6px;
  }

  .closeIcon:hover {
    cursor: pointer;
  }
</style>