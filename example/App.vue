<template>
  <div id="app">
    <hgroup :class="s.title">
      <h2>从左侧features中勾选定制你的数据表格</h2>
      <h3>
        本页代码实现可查看
        <a href="https://github.com/KevinMint55/vue-willtable/tree/master/example" target="blank">example</a>
      </h3>
    </hgroup>
    <div :class="s.main">
      <div :class="s.features_wrapper">
        <div :class="s.features_title">
          <span>Features</span>
          <button :class="s.github_link" @click="linkGithub"></button>
        </div>
        <ul :class="s.features_list">
          <li
            v-for="(item, index) in features"
            :key="index">
            <label>{{ item.label }}</label>
            <el-checkbox v-model="item.checked" @change="item.handleChange"></el-checkbox>
          </li>
        </ul>
        <div :class="s.more" @click="fullscreen">全屏</div>
      </div>
      <div :class="s.willtable_wrapper">
        <willtable
        ref="willtable"
        :columns="columns"
        v-model="data"
        :maxHeight="maxHeight"
        :disabled="disabled"
        :showIcon="showIcon"
        :cellStyle="cellStyle"
        :cellClassName="cellClassName"
        :rowHeight="28"
        :disabledCell="disabledCell"
        :showAddRow="showAddRow"
        :addRowAndCopy="true">
          <el-dialog :visible.sync="selectUserVisible">
            <el-radio-group v-model="username">
              <el-radio label="Will">Will</el-radio>
              <el-radio label="Tom">Tom</el-radio>
              <el-radio label="Jack">Jack</el-radio>
            </el-radio-group>
            <span slot="footer">
              <el-button @click="selectUserVisible = false">取 消</el-button>
              <el-button type="primary" @click="confirmUser">确 定</el-button>
            </span>
          </el-dialog>
        </willtable>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import {
  checkbox, Dialog, Button, Radio, RadioGroup,
} from 'element-ui';
import Willtable from '../src/components/Table.vue';
// import Willtable from '../dist/vue-willtable.min';
// import '../dist/vue-willtable.min.css';

export default {
  name: 'App',
  components: {
    Willtable,
    'el-checkbox': checkbox,
    'el-dialog': Dialog,
    'el-button': Button,
    'el-radio': Radio,
    'el-radio-group': RadioGroup,
  },
  data() {
    return {
      features: [
        {
          label: '显示数据类型icon',
          checked: true,
          handleChange: (checked) => {
            this.showIcon = checked;
          },
        },
        {
          label: '显示行多选',
          checked: true,
          handleChange: (checked) => {
            if (checked) {
              this.columns.unshift({
                type: 'selection',
                width: 40,
                fixed: true,
              });
            } else {
              this.columns.shift();
            }
          },
        },
        {
          label: '显示添加行',
          checked: false,
          handleChange: (checked) => {
            this.showAddRow = checked;
          },
        },
        {
          label: '固定列（序号、姓名）',
          checked: true,
          handleChange: (checked) => {
            this.columns.forEach((col) => {
              if (['sid', 'name'].includes(col.key)) {
                col.fixed = checked;
              }
            });
          },
        },
        {
          label: '启用筛选与过滤（序号、姓名、日期）',
          checked: false,
          handleChange: (checked) => {
            this.columns.forEach((col) => {
              if (['sid', 'name', 'date'].includes(col.key)) {
                this.$set(col, 'action', checked);
              }
            });
          },
        },
        {
          label: '禁止整表操作',
          checked: false,
          handleChange: (checked) => {
            this.disabled = checked;
          },
        },
        {
          label: '禁止序号操作',
          checked: false,
          handleChange: (checked) => {
            this.columns.some((col) => {
              if (col.key === 'sid') {
                this.$set(col, 'disabled', checked);
                return true;
              }
              return false;
            });
          },
        },
        {
          label: '禁止部分单元格操作(当工作岗位为Web前端开发，禁止编辑地址)',
          checked: false,
          handleChange: (checked) => {
            if (checked) {
              this.disabledCell = ({
                row, column,
              }) => {
                if (column.key === 'address' && row.work === 'Web前端开发') {
                  return true;
                }
                return false;
              };
            } else {
              this.disabledCell = () => false;
            }
          },
        },
        {
          label: '自定义单元格样式',
          checked: false,
          handleChange: (checked) => {
            if (checked) {
              this.cellStyle = ({ rowIndex, columnIndex }) => {
                if (rowIndex % 2 === 1 && columnIndex % 2 === 0) {
                  return {
                    color: '#67C23A',
                  };
                }
              };
            } else {
              this.cellStyle = () => { };
            }
          },
        },
        {
          label: '自定义单元格类名',
          checked: false,
          handleChange: (checked) => {
            if (checked) {
              this.cellClassName = ({ rowIndex, columnIndex }) => {
                if (rowIndex % 2 === 0 && columnIndex % 2 === 1) {
                  return {
                    customChanged: true,
                  };
                }
              };
            } else {
              this.cellClassName = () => { };
            }
          },
        },
      ],
      columns: [],
      columnsData: [
        {
          type: 'selection',
          width: 40,
          fixed: true,
        },
        {
          title: '序号',
          key: 'sid',
          fixed: true,
          type: 'number',
          width: 100,
        },
        {
          title: '姓名',
          key: 'name',
          fixed: true,
          width: 120,
          customInput: ({ rowIndex, columnIndex }) => {
            this.curEditRowIndex = rowIndex;
            this.curEditColumnIndex = columnIndex;
            this.selectUserVisible = true;
          },
        },
        {
          title: '日期',
          key: 'date',
          type: 'date',
          width: 100,
        },
        {
          title: '工作岗位',
          key: 'work',
          width: 300,
          type: 'select',
          options: [
            {
              value: 'Web前端开发',
              label: 'Web前端开发',
            },
            {
              value: 'Java开发',
              label: 'Java开发',
            },
            {
              value: 'Python开发',
              label: 'Python开发',
            },
            {
              value: 'Php开发',
              label: 'Php开发',
            },
          ],
        },
        {
          title: '月份',
          key: 'month',
          type: 'month',
          width: 100,
        },
        {
          title: '地址',
          key: 'address',
          width: 200,
        },
        {
          title: '标题',
          key: 'title',
          width: 300,
        },
        {
          title: '内容',
          key: 'paragraph',
          width: 300,
        },
        {
          title: '链接',
          key: 'url',
          width: 200,
        },
        {
          title: 'ip',
          key: 'ip',
          width: 200,
          validate: (value) => {
            const pattern = /((2(5[0-5]|[0-4]\d))|[0-1]?\d{1,2})(\.((2(5[0-5]|[0-4]\d))|[0-1]?\d{1,2})){3}/g;
            return pattern.test(value);
          },
        },
        {
          title: '总金额',
          key: 'sum',
          width: 200,
        },
        {
          title: 'ID',
          key: 'id',
          width: 200,
        },
        {
          title: '色值',
          key: 'color',
          width: 200,
        },
      ],
      data: [],
      disabledCell: () => false,
      showIcon: true,
      showAddRow: false,
      disabled: false,
      maxHeight: 800,
      cellStyle: () => { },
      cellClassName: () => { },
      selectUserVisible: false,
      username: 'Will',
      curEditRowIndex: null,
      curEditColumnIndex: null,
    };
  },
  mounted() {
    this.getList();
    this.maxHeight = document.documentElement.clientHeight - 200;
  },
  methods: {
    getList() {
      axios.get('https://demo.kevinmint.com/1.json').then((res) => {
        this.columns = this.columnsData;
        this.$refs.willtable.setData(res.data.list);
      }).catch(() => { });
    },
    getErrorRows() {
      console.log(this.$refs.willtable.getErrorRows());
    },
    getChangeData() {
      console.log(this.$refs.willtable.getChangeData());
    },
    addRow() {
      this.$refs.willtable.addRow(1, true);
    },
    fullscreen() {
      this.$refs.willtable.fullscreen();
    },
    // cellStyle({ rowIndex, columnIndex }) {
    //   if (rowIndex === 1) {
    //     return {
    //       color: 'red',
    //     };
    //   }
    //   if (columnIndex === 5) {
    //     return {
    //       color: 'green',
    //     };
    //   }
    // },
    add() {
      const obj = {};
      obj.id = new Date().getTime();
      this.columns.forEach((item) => {
        if (item.key) {
          obj[item.key] = '';
        }
      });
      this.$refs.willtable.addItem(obj);
    },
    remove() {
      this.$refs.willtable.removeItems('sid', this.selection.map((s) => s.sid));
    },
    linkGithub() {
      window.open('https://github.com/KevinMint55/vue-willtable', '_blank');
    },
    confirmUser() {
      this.$refs.willtable.setCellData(this.curEditRowIndex, this.curEditColumnIndex, this.username);
      this.selectUserVisible = false;
    },
  },
};
</script>

<style lang="scss" module="s">
:global {
  html,
  body,
  #app {
    height: 100%;
  }
  .customChanged {
    background-color: rgba(247, 181, 0, 0.1);
  }
}

.title {
  text-align: center;
  padding-top: 20px;
  h2 {
    font-size: 24px;
    margin-bottom: 6px;
  }
  h3 {
    font-size: 18px;
    font-weight: normal;
    a {
      color: #0366d6;
      cursor: pointer;
      &:hover {
        text-decoration: underline;
      }
    }
  }
}

.main {
  display: flex;
  align-items: flex-start;
  padding: 20px;
  overflow: auto;
  height: calc(100% - 81px);
}

.features_wrapper {
  flex: none;
  width: 280px;
  margin-right: 30px;
}

.features_title {
  display: flex;
  align-items: center;
  justify-content: space-between;
  color: #363636;
  margin-bottom: 20px;
  span {
    font-size: 18px;
  }
}

.features_list {
  padding-left: 20px;
  li {
    display: flex;
    align-items: center;
    justify-content: space-between;
    font-size: 14px;
    padding: 10px 2px;
    border-bottom: 1px solid #e3e3e3;
  }
}

.willtable_wrapper {
  flex: auto;
  overflow-x: auto;
  height: 100%;
}

.github_link {
  display: inline-block;
  width: 24px;
  height: 24px;
  background-image: url("./github.jpg");
  background-position: center;
  background-repeat: no-repeat;
  background-size: cover;
  transition: all 0.3s;
  opacity: 0.7;
  cursor: pointer;
  &:hover {
    transform: rotate(360deg) scale(2);
    opacity: 1;
  }
}

.more {
  font-size: 14px;
  text-align: right;
  margin-top: 12px;
}
</style>
