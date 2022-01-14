<script>
import _ from 'lodash';
import eventBus from './eventBus';
// * font-awesome-icon зарегистрирован глобально в main js

export default {
    name: "FiltersWrapper",
    render(createElement) {
        return !this.showFiltersWrapper 
            ?  createElement( 'button', 
                    {
                        attrs: {
                            class: "btn open-filter",
                        },
                        on: {
                            click: this.openFilter,
                        },
                        domProps: {
                            innerHTML: "Filter",
                        }
                    },
                )
            :   createElement('div', 
                    {
                        attrs: {
                            class: "filters-wrapper",
                        },
                    },
                    [
                        createElement('font-awesome-icon', 
                            {
                                class: {
                                    closeIcon: true,
                                },
                                // ? так не применяется
                                attrs: {
                                    class: 'closeIcon',
                                },
                                props: {
                                    icon: "times",
                                },
                                on: {
                                    click: this.closeFilter,
                                },
                            },
                        ),
                        createElement('h1', 'Filters'),
                        // id
                        createElement('div', 
                            {
                                attrs: {
                                    class: "filters-wrapper__line",
                                },
                            },
                            [
                                createElement('label',
                                    {
                                        attrs: {
                                            class: "range-label",
                                        },
                                    },
                                    [
                                        'ID',
                                        createElement('div', 
                                            {
                                                attrs: {
                                                    class: "range",
                                                },
                                            },
                                            [
                                                createElement('span', 'From'),
                                                createElement('input',
                                                    {
                                                        attrs: {
                                                            type: 'number',
                                                            name: 'id_min',
                                                            class: 'input number',
                                                            min: '1',
                                                            max: '500',
                                                        },
                                                        on: {
                                                            input:  (event) => {
                                                                this.filter.id.min = event.target.value;
                                                                this.filterByRange('id');
                                                            },
                                                        },
                                                    },
                                                ),
                                                createElement('span', 'To'),
                                                                                                createElement('input',
                                                    {
                                                        attrs: {
                                                            type: 'number',
                                                            name: 'id_max',
                                                            class: 'input number',
                                                            min: '1',
                                                            max: '500',
                                                        },
                                                        on: {
                                                            input:  (event) => {
                                                                this.filter.id.max = event.target.value;
                                                                this.filterByRange('id');
                                                            },
                                                        },
                                                    },
                                                ),
                                            ]
                                        ),
                                        createElement('font-awesome-icon',
                                            {
                                                class: {
                                                    closeIcon: true,
                                                },
                                                attrs: {
                                                    class: 'closeIcon',
                                                },
                                                props: {
                                                    icon: 'times',
                                                },
                                                on: {
                                                    click: () => {
                                                        this.removeFilter('id');
                                                    },
                                                },
                                            },
                                        )
                                    ],
                                )
                            ],
                        ),
                        // post id
                        createElement('div', 
                            {
                                attrs: {
                                    class: "filters-wrapper__line",
                                },
                            },
                            [
                                createElement('label',
                                    {},
                                    [
                                        'Post ID',
                                        createElement('input',
                                            {
                                                attrs: {
                                                    type: 'number',
                                                    name: 'post_id',
                                                    class: 'input number',
                                                    min: '1',
                                                    max: '100',
                                                },
                                                on: {
                                                    input:  (event) => {
                                                        this.filter.postId = event.target.value;
                                                        this.filterByNumber('postId');
                                                    },
                                                },
                                            },
                                        ),
                                        createElement('font-awesome-icon',
                                            {
                                                class: {
                                                    closeIcon: true,
                                                },
                                                attrs: {
                                                    class: 'closeIcon',
                                                },
                                                props: {
                                                    icon: 'times',
                                                },
                                                on: {
                                                    click: () => {
                                                        this.removeFilter('postId');
                                                    },
                                                },
                                            },
                                        ),
                                    ],
                                ),
                            ],
                        ),
                        // email
                        createElement('div', 
                            {
                                attrs: {
                                    class: "filters-wrapper__line",
                                },
                            },
                            [
                                createElement('label',
                                    {},
                                    [
                                        'Email',
                                        createElement('input',
                                            {
                                                attrs: {
                                                    type: 'text',
                                                    name: 'email',
                                                    class: 'input',
                                                    placeholder: "Введите значение"
                                                },
                                                on: {
                                                    input:  (event) => {
                                                        this.filter.email = event.target.value;
                                                        this.filterText('email');
                                                    },
                                                },
                                            },
                                        ),
                                        createElement('font-awesome-icon',
                                            {
                                                class: {
                                                    closeIcon: true,
                                                },
                                                attrs: {
                                                    class: 'closeIcon',
                                                },
                                                props: {
                                                    icon: 'times',
                                                },
                                                on: {
                                                    click: () => {
                                                        this.removeFilter('email');
                                                    },
                                                },
                                            },
                                        ),
                                    ],
                                ),
                            ],
                        ),
                        // name
                        createElement('div', 
                            {
                                attrs: {
                                    class: "filters-wrapper__line",
                                },
                            },
                            [
                                createElement('label',
                                    {},
                                    [
                                        'Name',
                                        createElement('input',
                                            {
                                                attrs: {
                                                    type: 'text',
                                                    name: 'name',
                                                    class: 'input',
                                                    placeholder: "Введите значение"
                                                },
                                                on: {
                                                    input:  (event) => {
                                                        this.filter.name = event.target.value;
                                                        this.filterText('name');
                                                    },
                                                },
                                            },
                                        ),
                                        createElement('font-awesome-icon',
                                            {
                                                class: {
                                                    closeIcon: true,
                                                },
                                                attrs: {
                                                    class: 'closeIcon',
                                                },
                                                props: {
                                                    icon: 'times',
                                                },
                                                on: {
                                                    click: () => {
                                                        this.removeFilter('name');
                                                    },
                                                },
                                            },
                                        ),
                                    ],
                                ),
                            ],
                        ),
                    // children of filter wrapper
                    ],
                // filter wrapper div end
                );
    // render function end
    },
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
            // * отсортированный массив при статической пагинации
            sortedList: [],
            requiredRowsLength: 5,
            rememberLengthCount: 0,
            rememberedCurrentPage: 0,
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
        eventBus.$on('sort-list', (data) => {
            this.sortedList = data;
        });
    },
    beforeDestroy() {
        // removing eventBus listener
        eventBus.$off('sort-list');
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
                return this.filteredList;
            }

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
            this.activeFilterProp = property;
            // ! при множественной фильтрации изменится array

            this.filteredList =  this.array.filter(row => row[property] === parseInt(this.filter[property]));

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

            this.activeFilterProp = property;

            this.filteredList =  this.array.filter(row => row[property].search(this.filter[property]) > -1);
            //  * в другом месте вызов fetch-for-filter блокирует перефильтрацию
            if (this.refilter) {
                this.$emit('fetch-for-filter')
            }
            this.$emit('filter', this.filteredList);

        }, 500),
        removeFilter(property) {
            this.hasFilter = false;

            if (typeof this.filter[property] === 'object') {
                this.filter[property].min = "";
                this.filter[property].max = "";
            }
            if (typeof this.filter[property] === 'string') {
                this.filter[property] = "";
            }
            
            this.$emit('removeFilter',  this.staticRows);
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