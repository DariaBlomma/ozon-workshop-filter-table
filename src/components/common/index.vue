<template>
<div>
  <!--  @getPage="infGetPage" для бесконечной пагиинации -->
  <!-- @getPage="getPage" для тстатической пагинации -->
  <oz-table
    :rows="rows"
    :all-pages="allPages"
    :total-pages="getTotalPages"
    :current-page="currentPage"
    :static-paging="staticPaging"
    :empty-message="emptyMessage"

    @getPage="infGetPage"
  >
    <oz-table-column prop="id" title="ID" />
    <oz-table-column prop="postId" title="Post ID" />

    <oz-table-column prop="email">
      <template #title>
        <b>Email</b>
      </template>

      <template #body="{ row }">
        <a :href="`mailto:${row.email}`">{{ row.email }}</a>
      </template>
    </oz-table-column>

    <oz-table-column prop="name" title="Name" />
  </oz-table>
  <FiltersWrapper 
    :static-paging="staticPaging"
    :fetched-rows="fetchedRows"
    :new-rows-length="newRowsLength"
    :current-page="currentPage"
    :page-size="pageSize"

    @filter="filterList"
    @remove-filter="removeFilter"
    @fetch-for-filter="infGetPage"
  />
</div>

</template>

<script>
import OzTable from './oz-table';
import OzTableColumn from './oz-table-column';
import FiltersWrapper from './filters-wrapper';

export default {
  name: 'Common',
  components: {
    OzTableColumn,
    OzTable,
    FiltersWrapper,
  },
  async created() {
    if (this.staticPaging) {
      this.fetchAllPages();
    } else {
      // * нужно для получения первой страницы при загрузке
      this.blockingPromise = this.fetchFirstPage();
    }
  },   
  // * для сброса фильтров или сортировки к исходному состоянию должен быть неизменяемый массив - fetchedRows, allPages 
  data() {
    return {
      rows: [],
      fetchedRows: [],
      newRows: [],
      afterNewRows: [],
      allPages: [],
      list: [],
      filteredList: [],
      neighbourPages: [],
      sortInfo: {},
      currentPage: 1,
      constantCurrentPage: 1,
      pageSize: 5,
      requiredRowsLength: 5,
      rememberLengthCount: 0,
      rememberedCurrentPage: 0,
      nextPageFetchedCount: 0,
      renderedRows: 0,
      renderedRowsLength: 0,
      emptyMessage: '',
      staticPaging: false,
      isFiltered: false,
      pageRendered: false,
      newRowsFetched: false,
      canBeFiltered: true,
      uniqueFiltered: false,
      hasFilter: false,
    };
  },
  computed: {
    getTotalPages() {
      return this.list.length;
    },
    // * передается пропсом в фильтр для определения refilter
    newRowsLength() {
      return !!this.newRows.length;
    }
  },
  methods: {
    async fetchAllPages() {
      try {
        const res = await fetch(`https://jsonplaceholder.typicode.com/comments`);
        this.allPages = await res.json();
        this.preparePages(this.allPages);
        this.getPage(1);
      } catch (e) {
        console.warn('Could not fetch all pages', e);
      }
    },
    async fetchFirstPage() {
      try {
        const res = await fetch(`https://jsonplaceholder.typicode.com/comments?postId=1`);
        this.fetchedRows = await res.json();
        this.rows = this.fetchedRows;
      } catch (e) {
        console.warn('Could not fetch first page', e);
      }
    },
    filterList(list) {
      console.log('in filter list')
      this.hasFilter = true;
      if (this.staticPaging) {
        this.preparePages(list);
        this.getPage(this.currentPage);
      } else {
        this.rows = list;
        // console.log('in filter this.rows: ', this.rows);
        }
    },
    // async filterList() {
    //   let array = [];
    //   if (this.isSorted) {
    //     if (!this.staticPaging) {
    //       array = this.fetchedRows;          
    //     } else {
    //       array = this.sortedList;
    //     }
    //   } else {
    //     if (!this.staticPaging) {
    //       array = this.fetchedRows;          
    //     } else {
    //       array = this.allPages;
    //     }
    //   }
      
    //   this.filteredList =  array.filter(row => row[this.sortInfo.filterProp].search(this.sortInfo.filterText) > -1);
    //   if (!this.staticPaging) {
    //     this.rows = this.filteredList;
    //   }
    // },
    // addFilter(value) {  
    //   console.log('in add filter')
    //   this.isFiltered = true;
    //   this.sortInfo = value;
    //   this.filterList();
      
    //   if (this.staticPaging) {
    //     this.preparePages(this.filteredList);
    //     this.getPage(this.currentPage);
    //   }
    // },
    // removeFilter(value) {
    removeFilter() {
      this.hasFilter = false; 
      this.rows = this.fetchedRows; 
      // console.log('in removeFilter');

      // if (this.staticPaging) {
      //   this.preparePages(this.sortedList);
      //   this.getPage(this.currentPage);
      // }
    },
    // создает разбитый на страницы массив
    preparePages(array) {
      if (array.length) {
        const list = [];
        for (let i = 0; i < Math.ceil(array.length/this.pageSize); i++) {
            list[i] = array.slice((i*this.pageSize), (i*this.pageSize) + this.pageSize);          
        }
        this.list = list;
      }
    },
    async getPage(number) {
      this.rows = this.list[number - 1];
      this.currentPage = number;
    },
    async fetchNextPage() {
      try { 
        this.newRowsFetched = false;
        const res = await fetch(`https://jsonplaceholder.typicode.com/comments?postId=${this.currentPage + 1}`);
        this.newRows = await res.json();
        this.nextPageFetchedCount++;

        // * если на экране помещается чуть больше 10 рядов при первой загрузке, следующая партия не грузится. Поэтому получаем сразу 2 страницы
        if (this.nextPageFetchedCount === 1) {
          const page2 = await fetch(`https://jsonplaceholder.typicode.com/comments?postId=${this.currentPage + 2}`);
          this.newRows = [ ... this.newRows, ... await page2.json()];
          this.currentPage++;
        } 
        this.newRowsFetched = true;
      } catch (e) {
        console.warn('Could not fetch next page', e);
      }
    },
    // * также вызывается, когда после фильтрации недостаточно рядов
    async infGetPage() {
      // console.log('in inf pager');
      this.blockingPromise && await this.blockingPromise;
    
      // * проверка, что не получаем 2 раза ту же страницу (происходит при сбросе фильтра по несуществующим данным)
      if (this.neighbourPages.length === 2) {
        this.neighbourPages.shift();
      }

      if (this.neighbourPages.length < 2) {
        this.neighbourPages.push(this.currentPage + 1);
      }

      if (this.neighbourPages.length && this.neighbourPages[0] !== this.neighbourPages[1]) {
        await this.fetchNextPage() && this.newRowsFetched;
      } else {
        return;
      }    
      
      if (this.newRows.length) {
        this.fetchedRows = [...this.fetchedRows, ...this.newRows];

        // * проверка на отсутствие фильтров и сортировки нужна, чтобы отфильтрованные ряды были в приоритете, а не сбрасывались здесь на fetchedRows
        if (!this.hasFilter && !this.hasSort) {
          this.rows = this.fetchedRows;
        }
        this.currentPage++;
      } else {
        this.emptyMessage = 'There are no more pages left';
        return;
      }
    }
  },
};
</script>
