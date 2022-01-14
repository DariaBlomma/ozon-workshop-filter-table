<script>
import OzTable from './oz-table';
import OzTableColumn from './oz-table-column';
import FiltersWrapper from './filters-wrapper';


export default {
  name: 'Common',
  render(createElement) {
    return createElement(
      'div', {}, 
      [
        createElement(OzTable, 
          {
            props: {
              rows: this.rows,
              allPages: this.allPages,
              totalPages: this.getTotalPages,
              currentPage: this.currentPage,
              staticPaging: this.staticPaging,
              emptyMessage: this.emptyMessage,
            },
            on: {
              getPage: this.getPage,
              // * для бесконечной пагинации
              // getPage: this.infGetPage,
              sortList: this.sortList,
            },
          },
          [
            createElement(OzTableColumn, 
              {
                props: {
                  prop: 'id', 
                  title: 'ID',
                },
              },
            ),
            createElement(OzTableColumn, 
              {
                props: {
                  prop: 'postId', 
                  title: 'Post ID',
                },
              },
            ),
            createElement(OzTableColumn, 
              {
                props: {
                  prop: 'email', 
                },
                scopedSlots: {
                  title: () => createElement('b', 
                    {
                      domProps: {
                        innerHTML: 'Email',
                      },
                    },
                  ),
                  // body: ({ row: { email } }) => {
                  // * props : row: { name, email, ....}, 
                  // * props.row.email
                  body: ( { email }) => {
                  return createElement('a', 
                    {
                      attrs: {
                        href: `mailto:${email}`
                      },
                      domProps: {
                        innerHTML: `${email}`,
                      },
                    },
                  )},
                },
              },
            ),
            createElement(OzTableColumn, 
              {
                props: {
                  prop: 'name', 
                  title: 'Name',
                },
              },
            ),
          ]
        ),
        createElement(FiltersWrapper, 
          {
            props: {
              staticPaging: this.staticPaging,
              allRows: this.allPages,
              fetchedRows: this.fetchedRows,
              newRowsLength: this.newRowsLength,
              currentPage: this.currentPage,
              pageSize: this.pageSize,
            },
            on: {
              removeFilter: this.removeFilter,
              fetchForFilter: this.infGetPage,
              filter: this.filterList,
            },
          }
        )
      ],
    )
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
      if (this.staticPaging) {
        this.preparePages(list);
        this.getPage(this.currentPage);
      }
    },
    filterList(list) {
      this.hasFilter = true;
      
      if (this.staticPaging) {
        this.preparePages(list);
        this.getPage(this.currentPage);
      } else {
        this.rows = list;
        }
    },
    removeFilter(list) {
      this.hasFilter = false; 
      this.rows = this.fetchedRows; 

      if (this.staticPaging) {
        this.preparePages(list);
        this.getPage(this.currentPage);
      }
    },
    // * создает разбитый на страницы массив
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
        this.newRowsFetched = false;
        this.prevFetchedPage = this.currentPage;

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