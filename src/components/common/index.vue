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

    @addFilter="addFilter"
    @removeFilter="removeFilter"
    @addSort="addSort"
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
    :refilter="refilter"

    @filter="filterList"
    @remove-filter="removeFilter"
  />
</div>

</template>

<script>
import { orderBy } from 'lodash/collection';
// todo - фильтры должны учитывать тип данные, м.б range, cheeckbox
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
      staticPaging: false,
      rows: [],
      fetchedRows: [],
      newRows: [],
      afterNewRows: [],
      renderedRows: 0,
      renderedRowsLength: 0,
      allPages: [],
      list: [],
      currentPage: 1,
      constantCurrentPage: 1,
      pageSize: 5,
      requiredRowsLength: 5,
      isSorted: false,
      isFiltered: false,
      pageRendered: false,
      newRowsFetched: false,
      sortedList: [],
      filteredList: [],
      sortFilterInfo: {},
      rememberLengthCount: 0,
      rememberedCurrentPage: 0,
      emptyMessage: '',
      canBeSorted: true,
      canBeFiltered: true,
      uniqueFiltered: false,
      nextPageFetchedCount: 0,
      hasFilter: false,
      hasSort: false,
    };
  },
  computed: {
    getTotalPages() {
      return this.list.length;
    },
    // * отправить на фильтрацию, если есть новые ряды, установлен фильтр и кол-во рядов меньше требуемого
    refilter() {
      return !!this.newRows.length && this.hasFilter &&  (this.rows.length < this.requiredRowsLength);
    },
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
    sortList() {
      this.canBeFiltered = false;
      let array = [];
      if (this.isFiltered) {
        if (this.staticPaging) {
          array = this.filteredList;
        } else {
          if (this.canBeSorted) {
            array = this.filteredList;
          }
        }
        
      } else {
          if (!this.staticPaging) {
            array = this.fetchedRows;
          } else {
            array = this.allPages;
          }
      }
      
      this.sortedList =  orderBy(array, [this.sortFilterInfo.sortProp], [this.sortFilterInfo.sortDirection]);

      if (!this.staticPaging) {
        this.rows = this.sortedList;
        this.canBeFiltered = true;
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
        console.log('in filter this.rows: ', this.rows);
        }

        // if (this.renderedRows < this.requiredRowsLength) {
        if (this.refilter) {
          console.log('less')
          this.fetchForFilter();
        } else {
          this.rememberLengthCount = 0;
          this.rememberCurrentPage(true);
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
      
    //   this.filteredList =  array.filter(row => row[this.sortFilterInfo.filterProp].search(this.sortFilterInfo.filterText) > -1);
    //   if (!this.staticPaging) {
    //     this.rows = this.filteredList;
    //   }
    // },
    addFilter(value) {  
      this.isFiltered = true;
      this.sortFilterInfo = value;
      this.filterList();
      
      if (this.staticPaging) {
        this.preparePages(this.filteredList);
        this.getPage(this.currentPage);
      }
    },
    // removeFilter(value) {
    removeFilter() {
      this.hasFilter = false;
      this.rows = this.fetchedRows; 
      // this.sortFilterInfo = value;
      // this.sortList();

      // if (this.staticPaging) {
      //   this.preparePages(this.sortedList);
      //   this.getPage(this.currentPage);
      // }
    },
    addSort(value) {
      this.isSorted = true;
      this.sortFilterInfo = value;
      this.sortList();

      if (this.staticPaging) {
        this.preparePages(this.sortedList);
        this.getPage(this.currentPage);
      }
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
    filterUniqueRows(array) {
       // * при быстрой прокрутке элементы могут дублироваться. Поэтому фильтруем на уникальность
      this.uniqueFiltered = false;
      this.rows = array.filter((value, index, array) => {
        if (array[index - 1]) {
          return array[index - 1].id !== value.id;
        }
      });
      this.uniqueFiltered = true;
    },
    // * вызывается, когда после фильтрации недостаточно рядов
    async fetchForFilter() {
      await this.fetchNextPage() && this.newRowsFetched;
      this.fetchedRows = [...this.fetchedRows, ...this.newRows];
      this.currentPage++;
    },
    async infGetPage() {
      console.log('in inf pager');
      this.blockingPromise && await this.blockingPromise;

// * при текущей реализации не нужны, пока оставлю про запас
      // this.renderedRows = this.$children[0].$refs.tbody.children.length;
      // this.renderedRowsLength = this.$children[0].$refs.tbody.children.length;

      await this.fetchNextPage() && this.newRowsFetched;
      

      if (this.newRows.length) {
        this.fetchedRows = [...this.fetchedRows, ...this.newRows];

        // * проверка на отсутствие фильтров и сортировки нужна, чтобы отфильтрованные ряды были в приоритете, а не сбрасывались здесь на fetchedRows
        if (!this.hasFilter && !this.hasSort) {
          this.rows = this.fetchedRows;
        }
        this.currentPage++;

        if (this.hasFilter) {
          this.rememberCurrentPage();
          this.getRequiredRowsLength();
          // todo - иногда попадаются дублирующиеся ряды. Cкорее, страницы, после сброса фильтра
          // todo - filter uniquerows убирает первый ряд. М.б дело в прокрутке?
        }

        //     // if (this.canBeFiltered) {
        //     //   this.canBeSorted = false;
        //     // }
        //     // // * при быстрой прокрутке элементы могут дублироваться. Поэтому фильтруем на уникальность
        //     // this.filterUniqueRows();

        //     // if (this.uniqueFiltered) {
            //   await this.infGetPage();
        //     // }
  
        // console.log(this.rows);
        // if (this.sortFilterInfo.filterProp) {     
        //   this.rememberCurrentPage();
        //   this.getRequiredRowsLength();
        //   // повторный вызов нужен для фильтрации по новополученным полям
        //   this.filterList();

        //   if (this.renderedRows < this.requiredRowsLength) {
        //     if (this.canBeFiltered) {
        //       this.canBeSorted = false;
        //     }
        //     // * при быстрой прокрутке элементы могут дублироваться. Поэтому фильтруем на уникальность
        //     this.filterUniqueRows();

        //     if (this.uniqueFiltered) {
        //       await this.infGetPage();
        //     }      

        //   } else {
        //     this.filterUniqueRows();
        //     this.rememberLengthCount = 0;
        //     this.rememberCurrentPage(true);
        //     this.canBeSorted = true;
            
        //     if (this.sortFilterInfo.sortProp) {
        //       if (this.canBeSorted) {
        //         this.sortList();
        //       }
        //     }
        //   }
        // }

        // if (this.sortFilterInfo.sortProp) {
        //   if (!this.sortFilterInfo.filterProp) {
        //     // убираем дубликаты после удаления фильтра
        //     this.filterUniqueRows();
        //     if (this.uniqueFiltered) {
        //       this.sortList();
        //     }        
        //   }
        // }

      } else {
        this.emptyMessage = 'There are no more pages left';

        return;
      }

    }
  },
};
</script>
