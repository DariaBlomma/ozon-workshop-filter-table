<script lang="jsx">
import OzTable from './oz-table';
import OzTableColumn from './oz-table-column';
import FiltersWrapper from './filters-wrapper';


export default {
  name: 'Common',
  components: {
    // ? не работает без объявления здесь
    OzTableColumn,
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
      // * имеет разное значение в зависимости от типа пагинации. При статической - текущая страница. При бесконечной - отрендеренные страницы
      rows: [],
      // *при бесконечной пагинации все полученные с сервера на данный момент ряды
      fetchedRows: [],
      // * при бесконечной пагинации ряды следующей страницы
      newRows: [],
      // * при статической пагинации все полученные с сервера ряды
      allPages: [],
      // * при статической пагинации используется как промежуточный массив при разбиении на страницы полученных рядов
      list: [],
      // * для определения дублирующихся страниц при сбросе фильтра
      neighbourPages: [],
      currentPage: 1,
      pageSize: 5,
      // * для бесконечной пагинации, получения большего кол-ва рядов на первой странице
      nextPageFetchedCount: 0,
      // * при бесконечной пагинации текст сообщения, что больше нет рядов
      emptyMessage: '',
      staticPaging: true,
      newRowsFetched: false,
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
    },
  },
  methods: {
    // * при статической пагинации получение всех рядов с сервера
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
    // * используется при бесконечной пагинации
    async fetchFirstPage() {
      try {
        const res = await fetch(`https://jsonplaceholder.typicode.com/comments?postId=1`);
        this.fetchedRows = await res.json();
        this.rows = this.fetchedRows;
      } catch (e) {
        console.warn('Could not fetch first page', e);
      }
    },
    // * используется при статической пагинации
    sortList(list) {
      console.log('in sort list')
      if (this.staticPaging) {
        this.preparePages(list);
        this.getPage(this.currentPage);
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
    removeFilter(list) {
      this.hasFilter = false; 
      this.rows = this.fetchedRows; 
      // console.log('in removeFilter');

      if (this.staticPaging) {
        this.preparePages(list);
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
    // * используется при статической пагинации
    async getPage(number) {
      this.rows = this.list[number - 1];
      this.currentPage = number;
    },
    async fetchNextPage() {
      try { 
        // console.log('in fetch next page')
        this.newRowsFetched = false;
        this.prevFetchedPage = this.currentPage;
        // console.log(' this.prevFetchedPage: ',  this.prevFetchedPage);
        // console.log('this.currentPage + 1: ', this.currentPage + 1);
        if (this.prevFetchedPage === this.currentPage + 1) {
          console.log('already fetched page')
        }
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
  render() {
      // onGetPage={this.infGetPage} для бесконечной пагиинации
      // onGetPage={this.getPage} для тстатической пагинации

      // ? используются дефолтные слоты для 4 колонок как this.$slots.default?
      // ? в колонке email  title, body получается именованным слотом?
      const scopedSlots = {
        title: () => <b>Email</b>,
        body: ( { row }) => <a href={ `mailto:${row.email}` }>{ row.email }</a>,
      };
    return (
      <div>
          <OzTable
            rows={this.rows}
            all-pages={this.allPages}
            total-pages={this.getTotalPages}
            current-page={this.currentPage}
            static-paging={this.staticPaging}
            empty-message={this.emptyMessage}

            onGetPage={this.getPage}
            onSort-list={this.sortList}
          >
          
            <oz-table-column prop="id" title="ID" />
            <oz-table-column prop="postId" title="Post ID" />

            <oz-table-column prop="email" scopedSlots={scopedSlots} />

            <oz-table-column prop="name" title="Name" />
          </OzTable>
          <FiltersWrapper 
            static-paging={this.staticPaging}
            all-rows={this.allPages}
            fetched-rows={this.fetchedRows}
            new-rows-length={this.newRowsLength}
            current-page={this.currentPage}
            page-size={this.pageSize}

            onFilter={this.filterList}
            onRemove-filter={this.removeFilter}
            onFetch-for-filter={this.infGetPage}
          />
        </div>
    );
  },
};
</script>