<template>
    <div>
        <button 
            class="btn open-filter" 
            @click="openFilter"
        >
            Filter
        </button>
        <div class="filters-wrapper" v-if="showFiltersWrapper">
            <font-awesome-icon
                icon="times"
                class=closeIcon
                @click="closeFilter"
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
                            v-model="filter.id.min"
                            @input="filterByRange('id')"
                        >
                        <span>To</span>
                        <input 
                            type="number" 
                            name="id_max" 
                            class="input number" 
                            min="1" 
                            max="500"
                            v-model="filter.id.max"
                            @input="filterByRange('id')"
                        >
                    </div>
                    <font-awesome-icon
                        icon="times"
                        class=closeIcon
                        @click="removeFilter('id')"
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
    </div>

</template>
<script>
// todo при статической пагинации после фильтра вызывать getPage и фильтровать уже отсортированные ряды
import _ from 'lodash';
import eventBus from './eventBus';

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
        allRows: {
            type: Array,
            required: false,
            default: () => [],
        },
        newRowsLength: {
            type: Boolean,
            required: true,
        },
        currentPage: {
            type: Number,
            required: true,
        },
        pageSize: {
            type: Number,
            required: true,
        },
    },
    data() {
        return {
            filter: {
                postId: "",
                email: "",
                name: "",
                id: {
                    min: "",
                    max: "",
                },
            },
            activeFilterProp: "",
            filterArray: [],
            filteredList: [],
            sortedList: [],
            requiredRowsLength: 5,
            rememberLengthCount: 0,
            rememberedCurrentPage: 0,
            nextPageFetchedCount: 0,
            isSorted: false,
            textIsFiltered: false,
            numberIsFiltered: false,
            rangeIsFiltered: false,
            hasFilter: false,
            showFiltersWrapper: false,
        }
    },
    computed: {
        // * отправить на фильтрацию, если есть новые ряды, установлен фильтр и кол-во рядов меньше требуемого
        refilter() {
            return this.newRowsLength && this.hasFilter &&  (this.filteredList?.length < this.requiredRowsLength);
        },
        // * массив данных для фильтрации. Зависит от типа пагинации
        array() {
            return this.staticPaging ? this.staticRows : this.filterArray;
        },
        staticRows() {
            return this.sortedList.length ? this.sortedList : this.allRows;
        },
    },
    watch: {
        async fetchedRows(newValue) { 
            this.filterArray = await newValue;
            if (this.hasFilter) {
                this.rememberCurrentPage();
                this.getRequiredRowsLength();

                if (this.refilter) {
                    if (this.textIsFiltered) {
                        this.filterText(this.activeFilterProp);
                    }
                    if (this.numberIsFiltered) {
                        this.filterByNumber(this.activeFilterProp);
                    }

                    if (this.rangeIsFiltered) {
                        this.filterByRange(this.activeFilterProp);
                    }
                } else {
                    this.rememberLengthCount = 0;
                    this.rememberCurrentPage(true)
                }
            } else {
                this.removeFilter(this.activeFilterProp)
            }
        }
    },
    mounted() {
        // adding eventBus listener
        eventBus.$on('sortList', (data) => {
            this.sortedList = data
        console.log('Custom event triggered!', data)
        })
    },
    beforeDestroy() {
        // removing eventBus listener
        eventBus.$off('sortList')
    },
    methods: {
        openFilter() {
            this.showFiltersWrapper = true;
        },
        closeFilter() {
            this.showFiltersWrapper = false;
        },
        // * при самом первом вызове запоминаем номер страницы. После is equal увеличивает номер страницы на 1.
        // * это нужно для правильного подсчета требуемых рядов на странице. 
        // * Они зависят не от currentPage, то есть нужного id поста, а от разбиения по 5штук на страницу уже отфильтрованных данных
        // * Получается, после набора требуемого размера рядов, в следующий вызов прибавится 5
        // ! после уравнения отфильтрованного списка нужно передать true в параметр, чтобы обновить номер запомненной страницы
        rememberCurrentPage(updatePageNumber = false) {
            this.rememberedCurrentPage++;
            if (this.rememberedCurrentPage === 1) {
                this.rememberedPageNumber = this.currentPage;
            }
            if (updatePageNumber) {
                this.rememberedPageNumber++;
            }
            return this.rememberedPageNumber;
        },
        // ! после уравнения отфильтрованного списка нужно обнулить rememberLengthCount, чтобы пересчитать requiredLength
        getRequiredRowsLength() {
            this.rememberLengthCount++;
            
            if (this.rememberLengthCount === 1) {
                this.requiredRowsLength = this.pageSize * this.rememberedPageNumber;
            }
            return this.requiredRowsLength;
        },
        filterByRange: _.debounce(function filterByRange(property) {
            this.textIsFiltered = false;
            this.numberIsFiltered = false;
            this.rangeIsFiltered = true;
            this.hasFilter = true;
            console.log('in filter by range');
            this.activeFilterProp = property;
            // ! при множественной фильтрации изменится array

            if (this.filter[property].min && this.filter[property].max) {
                this.filteredList =  this.array.filter(row => {
                    return row[property] >= parseInt(this.filter[property].min) 
                    && row[property] <= parseInt(this.filter[property].max);
                });
            }
            
            if (this.filter[property].min && !this.filter[property].max) {
                this.filteredList =  this.array.filter(row => {
                    return row[property] >= parseInt(this.filter[property].min);
                });
            }

            if (!this.filter[property].min && this.filter[property].max) {
                this.filteredList =  this.array.filter(row => {
                    return row[property] <= parseInt(this.filter[property].max);
                });
            }

            // * так работает сброс
            if (!this.filter[property].min && !this.filter[property].max) {
                console.log('will return')
                return this.filteredList;
            }
            console.log('this.filteredList: ', this.filteredList);
            if (this.refilter) {
                this.$emit('fetch-for-filter')
            }
            this.$emit('filter', this.filteredList);
        }, 500),
        filterByNumber: _.debounce(function filterByNumber(property) {
            this.textIsFiltered = false;
            this.rangeIsFiltered = false;
            this.numberIsFiltered = true;
            this.hasFilter = true;
            console.log('in filter by number');
            this.activeFilterProp = property;
            // ! при множественной фильтрации изменится array

            this.filteredList =  this.array.filter(row => row[property] === parseInt(this.filter[property]));

            // console.log('this.filteredList: ', this.filteredList);
            if (this.refilter) {
                this.$emit('fetch-for-filter')
            }
            this.$emit('filter', this.filteredList);
        }, 500),
        filterText: _.debounce(function filterText(property) {
            this.textIsFiltered = true;
            this.rangeIsFiltered = false;
            this.numberIsFiltered = false;
            this.hasFilter = true;

            console.log('in filter text')
            this.activeFilterProp = property;

            this.filteredList =  this.array.filter(row => row[property].search(this.filter[property]) > -1);
            // console.log('this.filteredList: ', this.filteredList);
            console.log('refilter in filter text', this.refilter);
            //  * в другом месте вызов fetch-for-filter блокирует перефильтрацию
            if (this.refilter) {
                this.$emit('fetch-for-filter')
            }
            this.$emit('filter', this.filteredList);

        }, 500),
        removeFilter(property) {
            this.hasFilter = false;

            if (typeof this.filter[property] === 'object') {
                // console.log('this.filter[property]: ', this.filter[property]);
                this.filter[property].min = "";
                this.filter[property].max = "";
            }
            if (typeof this.filter[property] === 'string') {
                this.filter[property] = "";
            }
            
            this.$emit('remove-filter');
        },
    },
};
</script>
<style scoped>
    .filters-wrapper, .open-filter {
        position: fixed;
        top: 0;
        right: 0;
        z-index: 1;
    }

    .open-filter {
        padding: 10px 20px;
        background-color: blue;
        color: white;
        border-radius: 20px;
    }

    .filters-wrapper {
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