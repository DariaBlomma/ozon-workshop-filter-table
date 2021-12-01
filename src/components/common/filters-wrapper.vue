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
                    <input 
                        type="number" 
                        name="id_min" 
                        class="input number" 
                        min="1" 
                        max="500"
                        v-model="filter.id_min"
                        @input="filterByRange('id_min', false)"
                    >
                    <span>To</span>
                    <input 
                        type="number" 
                        name="id_max" 
                        class="input number" 
                        min="1" 
                        max="500"
                        v-model="filter.id_max"
                        @input="filterByNumber('id_max')"
                    >
                </div>
                <font-awesome-icon
                    icon="times"
                    class=closeIcon
                    @click="removeFilter('id_min', 'id_max')"
                />
            </label>
        </div> 
        <div class="filters-wrapper__line">
            <label>
                Post ID
                <input 
                    type="number" 
                    name="post_id" 
                    class="input number" 
                    min="1" 
                    max="100"
                    v-model="filter.postId"
                    @input="filterByNumber('postId')"
                >
                <font-awesome-icon
                    icon="times"
                    class=closeIcon
                    @click="removeFilter('postId')"
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
                <input 
                    type="text" 
                    name="name" 
                    class="input" 
                    placeholder="Введите значение"
                    v-model="filter.name"
                    @input="filterText('name')"
                >
                <font-awesome-icon
                    icon="times"
                    class=closeIcon
                    @click="removeFilter('name')"
                />
            </label>
        </div>
    </div>
</template>
<script>
import _ from 'lodash';

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
        refilter: {
            type: Boolean,
            required: true,
        },
    },
    data() {
        return {
            filter: {
                postId: undefined,
                email: "",
                name: "",
                id_min: "",
                id_max: "",
            },
            activeFilterProp: "",
            isSorted: false,
            textIsFiltered: false,
            numberIsFiltered: false,
            filterArray: [],
            filterCount: 0,
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
            console.log('refilter', this.refilter);
            if (this.refilter) {
                if (this.textIsFiltered) {
                    this.filterText(this.activeFilterProp);
                }
                if (this.numberIsFiltered) {
                    this.filterByNumber(this.activeFilterProp);
                }
            }
        }
    },
    methods: {
        filterByRange: _.debounce(function filterByNumber(property) {
            this.textIsFiltered = false;
            this.numberIsFiltered = true;
            console.log('in filter by number');
            this.activeFilterProp = property;
            // ! при множественной фильтрации изменится array
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
            this.filteredList =  array.filter(row => row[property] === parseInt(this.filter[property]));

            // console.log('this.filteredList: ', this.filteredList);
            this.$emit('filter', this.filteredList);
        }, 500),
        filterByNumber: _.debounce(function filterByNumber(property) {
            this.textIsFiltered = false;
            this.numberIsFiltered = true;
            console.log('in filter by number');
            this.activeFilterProp = property;
            // ! при множественной фильтрации изменится array
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
            this.filteredList =  array.filter(row => row[property] === parseInt(this.filter[property]));

            // console.log('this.filteredList: ', this.filteredList);
            this.$emit('filter', this.filteredList);
        }, 500),
        filterText: _.debounce(function filterText(property) {
            this.textIsFiltered = true;
            this.numberIsFiltered = false;
            console.log('in filter')
            this.activeFilterProp = property;
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
            this.$emit('filter', this.filteredList);
        }, 500),
        removeFilter(property) {
            // console.log('property: ', property);
            if (typeof this.filter[property] === 'string') {
                this.filter[property] = "";
            } else {
                // todo - возможно, потом типы изменятся
                 this.filter[property] = undefined;
            }
            
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