<template>
  <div>
    <slot v-bind="this"></slot>
    <el-pagination
      @current-change="onCurrentChange"
      @size-change="onPageSize"
      :current-page="currentPage"
      :page-size="pageSize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="totalRecords">
    </el-pagination>
  </div>
</template>
<script>
export default {
  props: ['data', 'total', 'orderBy', 'filters'],
  data() {
    return {
      pageSize: 10,
      currentPage: 1,
      orderby: this.orderBy,
      pageData: [],
      filtered: [],
    };
  },
  computed: {
    filter() {
      const ret = [];
      if (this.filters) {
        Object.entries(this.filters).forEach((item) => {
          if (item[1] !== '' && item[1] !== undefined) {
            ret.push(`${item[0]}:${item[1]}`);
          }
        });
      }
      return (ret.length ? ret.join(',') : undefined);
    },
    offset() {
      return (this.currentPage - 1) * this.pageSize;
    },
    params() {
      return {
        pagesize: this.pageSize,
        orderby: this.orderby,
        offset: this.offset,
        filter: this.filter,
      };
    },
    totalRecords() {
      if (this.data) {
        return this.filtered.length;
      }
      return this.total;
    },
  },
  watch: {
    $route: 'getDataByParams',
    data: 'buildCurrentPage',
  },
  created() {
    this.getDataByParams();
  },
  methods: {
    search() {
      this.getDataByParams();
    },
    getDataByParams() {
      if (this.data) {
        this.buildCurrentPage();
      }
    },
    buildCurrentPage() {
      if (this.data) {
        let filtered = this.data.slice();
        if (this.filters) {
          Object.entries(this.filters).forEach((item) => {
            const key = item[0];
            const value = item[1];
            if (value) {
              filtered = filtered.filter((e) => {
                if (typeof e[key] === 'string') {
                  return e[key].indexOf(value) > -1;
                }
                return e[key] === value;
              });
            }
          });
        }
        if (this.orderby) {
          const key = this.orderby.replace(/^-/, '');
          if (this.orderby[0] === '-') {
            filtered.sort((a, b) => (b[key] - a[key]));
          } else {
            filtered.sort((a, b) => (a[key] - b[key]));
          }
        }
        this.filtered = filtered;
        if (filtered.length > this.pageSize) {
          filtered = filtered.filter((e, index) => (
            (index < (this.offset + this.pageSize)) && (index >= this.offset)
          ));
        }
        this.pageData = filtered;
      }
    },
    onSortChange(sort) {
      if (sort.order === null) {
        this.orderby = this.orderBy;
      } else if (sort.order === 'descending') {
        this.orderby = `-${sort.prop}`;
      } else {
        this.orderby = `${sort.prop}`;
      }
      this.getDataByParams();
    },
    onCurrentChange(page) {
      this.currentPage = page;
      this.getDataByParams();
    },
    onPageSize(pageSize) {
      this.pageSize = pageSize;
      this.getDataByParams();
    },
  },
};
</script>

