<template>
  <div class="app">
    <div class="container-fluid">
      <vue-bootstrap-table
        :columns="columns"
        :actions="actions"
        :values="values"
        :show-filter="showFilter" 
        :show-column-picker="showPicker">
        <p >One</p>
        <button slot="operation"></button>
      </vue-bootstrap-table>
    </div>
  </div>
</template>
<script>
import VueBootstrapTable from './components/VueBootstrapTable.vue';
// 模拟数据
var items = [];
for (let i=0; i<100; i++) {
  let item = {};
  item.id = i;
  item.name = 'name';
  item.country = 'country';
  item.age = i + 12;
  items.push(item);
}
export default {
  data() {
    return {
      // 可过滤
      showFilter: true,
      // 可动态选择显示列
      showPicker: true,
      // 表头的 key 对应 行数据 values 的 key
      columns: [{
        title: "ID",
        key: "id"
      }, {
        title: "姓名",
        key: "name"
      }, {
        title: "年龄",
        key: "age"
      }, {
        title: "国家",
        key: "country"
      }],
      // 配置操作字段 里面是 a 标签
      actions: [{
        text: '编辑',
        fn (row, ev) {
          alert('编辑id：' + row.id);
          console.log(ev);
        }
      }, {
        text: '删除',
        fn (row, ev) {
          alert('删除id：' + row.id);
          console.log(ev);
        }
      }],
      values: items
    }
  },
  components: {
    VueBootstrapTable
  }
}
</script>
