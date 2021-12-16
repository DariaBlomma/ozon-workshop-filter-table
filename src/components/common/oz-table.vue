<script lang="jsx">
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
    filteredPage: {
      type: Array,
      default: () => [],
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
      required: true,
    },
  },
  data() {
    return {
      sortInfo: {
        // property for sorting
        sortProp: '',
        // asc desc
        sortDirection: '',
      }
    };
  },
  methods: {
    toggleSort(prop) {
      this.sortInfo.sortProp = prop;
      this.sortInfo.sortDirection = (this.sortInfo.sortDirection === 'desc' || !this.sortInfo.sortDirection) ? 'asc' : 'desc';
      this.$emit('addSort', this.sortInfo);
    },
    renderHead(h, columnsOptions) {
      const { $style, sortInfo } = this;

      return columnsOptions.map((column) => {
        const renderedTitle = column.scopedSlots.title ? column.scopedSlots.title() : column.title;
        let sortIcon = 'sort';

        if (sortInfo.sortProp === column.prop) {
          sortIcon = sortInfo.sortDirection === 'asc' ? 'sort-amount-down' : 'sort-amount-up';
        }

        return (
          <th key={column.prop} class={$style.headerCell}>
            <div class={$style.headerCellContent}>
              <span>{renderedTitle}</span>
              <font-awesome-icon
                class={$style.sortIcon}
                icon={sortIcon}
                on={{ click: () => this.toggleSort(column.prop) }}
              />
            </div>
          </th>
        );
      });
    },
    renderRows(h, columnsOptions) {
      return this.rows.map((row, index) => {
        return <tr key={row.id || index}>{...this.renderColumns(h, row, columnsOptions)}</tr>;
      });
    },
    renderColumns(h, row, columnsOptions) {
      return columnsOptions.map((column) => {
        return (
          <td key={column.prop} class={this.$style.cell}>
            {column.scopedSlots.body ? column.scopedSlots.body({ row }) : row[column.prop]}
          </td>
        );
      });
    },
    getColumnOptions() {
      return this.$slots.default.
        filter(item => item.componentOptions && item.componentOptions.tag === 'oz-table-column').
        map(column =>
          Object.assign({}, column.componentOptions.propsData, {
              scopedSlots: column.data.scopedSlots || {}
            }
          )
        );
    },
    renderInfPager() { 
      const directives = [
        {
          name: 'detect-viewport',
          value: {
            callback: this.$listeners.getPage
          }
        }
      ];

      const style = {
        background: `url("${DotsLoaderIcon}") no-repeat center`
      };

      return (
        <div {...{ class: this.$style.infPager, style, directives }} />
      );
    },
  },
  render(h) {
    const { $style, totalPages, currentPage, staticPaging, emptyMessage, $listeners } = this;
    const { getPage } = $listeners;
    const columnsOptions = this.getColumnOptions();
    const columnsHead = this.renderHead(h, columnsOptions);
    const rows = this.renderRows(h, columnsOptions);

    return (
      <div>
        <table class={$style.table}>
          <thead>{...columnsHead}</thead>
          <tbody ref="tbody">{...rows}</tbody>
          { emptyMessage || ""}
        </table>
        {staticPaging
        
          ? <OzTablePaginator totalPages={totalPages} currentPage={currentPage} on={{ getPage: getPage }} />
          : this.renderInfPager()
        }
      </div>
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
