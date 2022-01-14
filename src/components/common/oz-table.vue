<script>
import eventBus from './eventBus';
import { orderBy } from 'lodash/collection';
import OzTablePaginator from './oz-table-paginator';
import DotsLoaderIcon from './dost-loader.svg';

export default {
  name: 'oz-table',
  props: {
    rows: {
      type: Array,
      default: () => []
    },
    allPages: {
      type: Array,
      default: () => []
    },
    totalPages: {
      type: Number,
      default: 0
    },
    currentPage: {
      type: Number,
      default: 0
    },
    staticPaging: {
      type: Boolean,
      default: true
    },
    emptyMessage: {
      type: String,
      default: '',
    },
  },
  data() {
    return {
      sortInfo: {
        // property for sorting
        sortProp: '',
        // asc desc
        sortDirection: '',
      },
      // * используется при статической пагинации
      allSortedPages: [],
    };
  },
  components: { OzTablePaginator },
  computed: {
    // * массив данных для фильтрации. Зависит от типа пагинации
    array() {
        return this.staticPaging ? this.allPages : this.rows;
    },
    // * из какого массива рендерить ряды. Зависит от типа пагинации
    renderedRowsArray() {
        return this.staticPaging ? this.rows : this.sortedRows;
    },
    // * используется только при бесконечной пагинации
    sortedRows() {
      let res;

      if (!this.sortInfo.sortProp) {
        res =  this.rows;
      }

      res = orderBy(this.rows, [this.sortInfo.sortProp], [this.sortInfo.sortDirection]);

      return res;
    },
  },
  methods: {
    sortStaticRows() {
        this.allSortedPages = orderBy(this.allPages, [this.sortInfo.sortProp], [this.sortInfo.sortDirection]);
    },
    toggleSort(prop) {
      this.sortInfo.sortProp = prop;
      this.sortInfo.sortDirection = (this.sortInfo.sortDirection === 'desc' || !this.sortInfo.sortDirection) ? 'asc' : 'desc';
      if (this.staticPaging) {
        this.sortStaticRows();
        this.$emit('sort-list', this.allSortedPages);
        eventBus.$emit('sort-list', this.allSortedPages);
      }
    },
    renderHead(createElement, columnsOptions) {
      const { $style, sortInfo } = this;

      return columnsOptions.map((column) => {
        /* column : 
            {
              prop: 'id',
              title: 'ID",
              scopedSlots: {},
            }
        */

        // * renderedTitle - string, eg: Id
        const renderedTitle = column.scopedSlots.title ? column.scopedSlots.title() : column.title;
        let sortIcon = 'sort';

        if (sortInfo.sortProp === column.prop) {
          sortIcon = sortInfo.sortDirection === 'asc' ? 'sort-amount-down' : 'sort-amount-up';
        }

        return createElement(
          'th',
          {
            attrs: {
              key: column.prop,
              class: $style.headerCell,
            },
          },
          [
            createElement(
              'div',
              {
                attrs: {
                  class: $style.headerCellContent,
                },
              },
              [
                createElement(
                  'span', {}, renderedTitle,
                ),
                createElement('font-awesome-icon',  
                  {
                    class: {
                      [$style.sortIcon]: true,
                    },
                    props: {
                      icon: sortIcon,
                    },
                    on: {
                      click: () => this.toggleSort(column.prop),
                    },
                  },
                ),
              ],
            ),
          ],
        );
      });
    },
    renderRows(createElement, columnsOptions) { 
      return this.renderedRowsArray.map((row, index) => {
        const columns = this.renderColumns(createElement, row, columnsOptions);

        return createElement(
          'tr',
          {
            attrs: {
              key: row.id || index,
            },
          },
          [
            columns.map(column => {
              return createElement(column.tag, column.data, column.children);
            }),
          ],
        );
      });
    },
    renderColumns(createElement, row, columnsOptions) {
      /* 
        {
          body: '',
          id,
          email,
          postId,
        }
      */
      return columnsOptions.map((column) => {
        return createElement(
          'td',
          {
            attrs: {
              key: column.prop,
              class: this.$style.cell,
            },
          },
          [
            // * если есть column.scopedSlots.body, то это функция из index.js scopedSlots
            // * row[column.prop] - содержимое ячейки, напр, 1, myEmail"gmail.com
            column.scopedSlots.body ?
              createElement(column.scopedSlots.body(row).tag, column.scopedSlots.body(row).data, column.scopedSlots.body(row).children) :
              row[column.prop],
          ],
        );
      });
    },
    getColumnOptions() {
      // ? item.tag существует верный, а item.componentOptions.tag undefined
      // ? componentOptions.tag где задается ?
      return this.$slots.default.
        filter(item => item.componentOptions && (item.tag.includes('oz-table-column') || item.componentOptions.tag === 'oz-table-column')).
        map(column =>
          Object.assign({}, column.componentOptions.propsData, {
              scopedSlots: column.data.scopedSlots || {}
            }
          )
        );
    },
    renderInfPager(createElement) { 
      return createElement(
        'div',
        {
          attrs: {
            class: this.$style.infPager,
          },
          style: {
            background: `url("${DotsLoaderIcon}") no-repeat center`
          },
          directives: [
            {
              name: 'detect-viewport',
              value: {
                callback: this.$listeners.getPage,
              }
            },
          ],
        },
      );
    },
  },
  render(createElement) {
    const { $style, totalPages, currentPage, staticPaging, emptyMessage, $listeners } = this;
    const { getPage } = $listeners;
    /* columnsOptions : 
    [
      {
        prop: 'id',
        title: 'ID",
        scopedSlots: {},
      },
      ...
    ]
    */
    const columnsOptions = this.getColumnOptions();
    // * columnsHead [Vnode, ...]
    const columnsHead = this.renderHead(createElement, columnsOptions);
    // * rows [Vnode, ...]
    const rows = this.renderRows(createElement, columnsOptions);

    return createElement(
      'div',
        {},
        [
          createElement('table', 
            {
              attrs: {
                class: $style.table,
              },
            },
            [
              createElement('thead', 
                {},
                [
                  columnsHead.map(child => {
                    // * child - VNode
                    return createElement(child.tag, child.data, child.children);
                  }),
                ],
              ),
              createElement('tbody', 
                {
                  ref: 'tbody',
                },
                [
                  rows.map(child => {
                    // * child - VNode
                    return createElement(child.tag, child.data, child.children);
                  }),
                ],
              ),
              emptyMessage,
            ],
          ),
          staticPaging ? 
            createElement(
              'OzTablePaginator',
              {
                props: {
                  totalPages,
                  currentPage,
                },
                on: {
                  getPage,
                },
              },
            ) :
            this.renderInfPager(createElement),
        ],
    );
  }
};
</script>

<style module>
  .table {
    border-spacing: 0;
    margin: 8px;
    width: 100%;
  }

  .cell {
    text-align: left;
    border-bottom: 1px solid #c8cacc;
    padding: 1rem 1rem;
  }

  .headerCell {
    composes: cell;
    background: #c7cbcb;
  }

  .headerCellContent {
    display: flex;
    align-items: center;
  }

  .sortIcon {
    margin-left: 8px;
    margin-right: 24px;
  }

  .sortIcon:hover {
    cursor: pointer;
  }

  .infPager {
    width: 100%;
    height: 32px;
  }
</style>
