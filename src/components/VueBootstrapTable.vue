<template>
  <div class="container-fluid">
    <div class="row">
      <div class="col-sm-6">
        <div v-if="showFilter" style="padding-top: 10px;padding-bottom: 10px;">
          <div class="input-group">
            <div class="input-group-addon">
              <i class="glyphicon glyphicon-search"></i>
            </div>
            <div style="display: table-cell; width: 32%;">
              <select class="form-control" v-model="filterField">
                <option value="all">全部</option>
                <option v-for="option in columns" :value="option.key">{{ option.title }}</option>
              </select>
            </div>
            <input type="text" class="form-control" placeholder="Filter" v-model="filterKey">
          </div>
        </div>
      </div>
      <div class="col-sm-6">
        <div v-if="showColumnPicker" style="padding-top: 10px;padding-bottom: 10px;">
          <div class="btn-group" :class="{'open' : columnMenuOpen}"
                @mouseenter.stop.prevent = "columnMenuOpen = true"
                @mouseleave.stop.prevent = "columnMenuOpen = false">
            <button type="button" class="btn btn-default dropdown-toggle" data-toggle="dropdown" aria-haspopup="true">
              显示列
              <span class="caret"></span>
            </button>
            <ul class="dropdown-menu">
              <li v-for="column in displayCols">
                <a href="#" @click.stop.prevent="toggleColumn(column)">
                  <i class="glyphicon glyphicon-ok" :class="{ showOK: !column.visible }"></i>
                  {{column.title}}
                </a>
              </li>
            </ul>
          </div>
        </div>
      </div>
    </div>
    <div class="row">
      <div class="col-sm-12">
        <table class="table table-bordered table-hover table-condensed table-striped vue-table">
          <thead>
            <tr>
              <th v-for="column in displayCols | filterBy true in 'visible'" track-by="$index"
                  @click="column.key==='operation' ? null : sortBy(column.key)" 
                  :class="column.key==='operation' ? null : getClasses(column.key)">
                {{ column.title }}
              </th>
              <th v-if="actions">操作</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="entry in filteredValues | orderBy sortKey sortOrders[sortKey] | limitBy pageCount firstRow" track-by="$index">
              <td v-for="column in displayCols | filterBy true in 'visible'" track-by="$index" v-show="column.visible">
                {{ entry[column.key] }}
              </td>
              <td v-if="actions">
                <a href="javascript:;" v-for="item in actions" @click="item.fn(entry, $event)">
                  {{ item.text }}
                </a>
              </td>
            </tr>
          </tbody>
        </table>
      </div>      
    </div>
    <div class="table-footer">
      <ul class="pagination" v-if="lastPage > 1">
        <li :class="{disabled: currentPage == 1}">
          <a href="javascript:;" @click="togglePage('prev')">&laquo;</a>
        </li>
        <li :class="{active: currentPage == 1}">
          <a href="javascript:;" @click="togglePage(1)">1</a>
        </li>
        <li v-if="currentPage > 3 && lastPage > 5">
          <span><strong>...</strong></span>
        </li>
        <li v-for="page in centerPartPage" :class="{ active: currentPage == page }">
          <a href="javascript:;" @click="togglePage(page)"> {{ page }} </a>
        </li>
        <li v-if="lastPage > 5 && currentPage < lastPage - 2">
          <span><strong>...</strong></span>
        </li>
        <li :class="{active: currentPage == lastPage}">
          <a href="javascript:;" @click="togglePage(lastPage)">{{ lastPage }}</a>
        </li>
        <li :class="{disabled: currentPage == lastPage}">
          <a href="javascript:;" @click="togglePage('next')"> &raquo; </a>
        </li>
      </ul>
      <div class="pageCounts">
        <div class="input-group">
          <select class="form-control" v-model="pageCount">
            <option :value="5">5</option>
            <option :value="10">10</option>
            <option :value="15">15</option>
            <option :value="20">20</option>
          </select>
          <span class="input-group-addon">条/页</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    // 表头字段
    columns: {
      type: Array,
      required: true
    },
    // 配置操作字段
    actions: {
      type: Array,
      required: false
    },
    // 行数据
    values: {
      type: Array,
      required: true
    },
    // 可否排序
    sortable: {
      type: Boolean,
      required: false,
      default: true
    },
    // 有无搜索功能
    showFilter: {
      type: Boolean,
      required: false,
      default: false
    },
    // 可否动态选择列
    showColumnPicker: {
      type: Boolean,
      required: false,
      default: false
    }
  },
  data () {
    return {
      filteredSize: 0,
      filterKey: "",
      filterField: "all",
      sortKey: "",
      sortOrders: {},
      columnMenuOpen: false,
      displayCols: [],
      currentPage: 1,
      pageCount: 10
    };
  },
  ready () {
    var vm = this;
    this.setSortOrders();
    this.columns.forEach(function(column) {
      var obj = {};
      obj.title = column.title;
      obj.key = column.key;
      obj.visible = true;
      vm.displayCols.push(obj);
    });
  },
  watch: {
    columns () {
      this.displayCols = [];
      var self = this;
      this.columns.forEach(function(column) {
        var obj = {};
        obj.title = column.title;
        obj.key = column.key;
        obj.visible = true;
        self.displayCols.push(obj);
      });
      this.setSortOrders();
    },
    showFilter () {
      this.filterKey = "";
    },
    showColumnPicker () {
      this.columnMenuOpen = false;
      this.displayCols.forEach(function(column) {
        column.visible = true;
      });
    },
    filteredSize () {
      this.currentPage = 1;
    },
    pageCount () {
      if ( this.lastPage < this.currentPage) {
        this.currentPage = this.lastPage;
      }
    }
  },
  computed: {
    filteredValues () {
      var result = null;
      if ( this.filterField === 'all' ) {
        result = this.$options.filters.filterBy( this.values, this.filterKey );
      } else {
        result = this.$options.filters.filterBy( this.values, this.filterKey, 'in', this.filterField );
      }
      result = this.$options.filters.orderBy(result, this.sortKey, this.sortOrders[this.sortKey]);
      this.filteredSize = result.length;
      return result;
    },
    firstRow () {
      return ( this.currentPage - 1 ) * this.pageCount;
    },
    lastPage () {
      return Math.ceil(this.filteredSize / this.pageCount);
    },
    centerPartPage () {
      const arr = [];
      if (this.currentPage < 4) {
        for (let i=2; i<this.lastPage && i<5; i++) {
          arr.push(i);
        }
        return arr;
      } else if (this.currentPage < this.lastPage-1) {
        for (let i=this.currentPage-1; i<this.currentPage+2; i++) {
          arr.push(i);
        }
        return arr;
      } else {
        for (let i=this.lastPage-3; i<this.lastPage; i++) {
          arr.push(i);
        }
        return arr;
      }
    }
  },
  methods: {
    setSortOrders () {
      this.sortKey = "";
      var sortOrders = {};
      this.columns.forEach(function(column) {
        sortOrders[column.key] = 0;
      });
      this.sortOrders = sortOrders;
    },
    sortBy (key) {
      if (this.sortable) {
        var vm = this;
        this.sortKey = key;
        this.columns.forEach(function(column) {
          if (column.key !== key) {
            vm.sortOrders[column.key] = 0;
          }
        });
        if (this.sortOrders[key] === 0) {
          this.sortOrders[key] = 1;
        } else {
          this.sortOrders[key] = this.sortOrders[key] * -1;
        }
      }
    },
    getClasses (key) {
      var classes = [];
      if (this.sortable) {
        classes.push("arrow");
        if (this.sortOrders[key] === 1) {
          classes.push("asc");
        } else if (this.sortOrders[key] === -1) {
          classes.push("dsc");
        }
      }
      return classes;
    },
    toggleColumn (column) {
      column.visible = !column.visible;
    },
    togglePage(page) {
      switch (page) {
        case 'prev':
          if (this.currentPage <= 1) return;
          this.currentPage--;
          break;
        case 'next':
          if (this.currentPage >= this.lastPage) return;
          this.currentPage++;
          break;
        default:
          if (this.currentPage == page) return;
          this.currentPage = page;
      }
    }    
  }
}
</script>

<style scoped>
  table.vue-table thead > tr > th {
    cursor: pointer;
    padding-right: 30px !important;
  }
  .vue-table {
    margin-bottom: 10px;
  }
  .vue-table .arrow {
    opacity: 1;
    position: relative;
  }
  .vue-table .arrow:after {
    position: absolute;
    bottom: 8px;
    right: 8px;
    display: block;
    font-family: 'Glyphicons Halflings';
    content: "\e150";
  }
  .vue-table .arrow.asc:after {
    content: "\e155";
  }
  .vue-table .arrow.dsc:after {
    content: "\e156";
  }
  .showOK {
    opacity: 0;
  }
  .dropdown-menu {
    margin-top: 0;
  }  
  .table-footer {
    text-align: right;
  }
  .table-footer .pagination {
    margin: 0;
  }
  .table-footer .pagination li>span:hover {
    background-color: #fff;
  }
  .table-footer .pageCounts {
    display: inline-block;
    width: 120px;
  }
</style>

