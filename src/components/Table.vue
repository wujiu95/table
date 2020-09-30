<template>
  <div class="table">
    <el-table
      v-loading="isLoading"
      :data="tableList"
      style="width: 100%;"
      :ref="tableName"
      border
      :height="height"
      :cell-class-name="tableCellClassName"
      :row-class-name="tableRowClassName"
      :header-cell-class-name="tableCellClassName"
      :row-key="row => rowKeyFunc(row)"
      @selection-change="handleSelectionChange"
      :tree-props="{ children: 'children', hasChildren: 'hasChildren' }"
    >
      <slot name="expand"></slot>
      <el-table-column
        v-if="showSelection"
        type="selection"
        :resizable="false"
        width="55"
      >
      </el-table-column>
      <el-table-column
        label="序号"
        v-if="showIndex"
        type="index"
        width="50"
        :resizable="false"
      >
      </el-table-column>
      <template v-for="(config, index) in tableConfig">
        <template v-if="config.type === 'selection'">
          <el-table-column
            :key="index"
            :prop="config.prop"
            :selectable="
              (row, i) => {
                return selectable(row, i, config.prop, config);
              }
            "
            type="selection"
            :label="config.label || ''"
            :reserve-selection="true"
            :resizable="false"
            width="55"
          ></el-table-column>
        </template>
        <el-table-column
          v-else
          :key="index"
          :prop="config.prop"
          :label="config.label || ''"
          :width="config.width"
          :resizable="false"
          :align="center ? 'center' : 'left'"
        >
          <template slot-scope="scope">
            <img
              style="height: 45px;"
              v-if="config.isImage"
              :src="scope.row[config.prop]"
              alt=""
            />
            <div
              class="link-div"
              :class="{ 'link-gray': scope.row[config.prop] == '0' }"
              v-else-if="config.isLink"
              @click="$emit('clickLink', scope.row, config)"
            >
              <el-link
                type="primary"
                :class="{
                  'font-green': config.flag === '2',
                  'font-red': config.flag === '1'
                }"
              >
                {{ scope.row[config.prop] }}
              </el-link>
            </div>
            <template v-else-if="config.type && config.type == 'file'">
              <template
                v-if="
                  scope.row[config.prop] &&
                    jsonParseMethod(scope.row[config.prop]).length > 0
                "
              >
                <template
                  v-for="(item, key) in jsonParseMethod(scope.row[config.prop])"
                >
                  <el-link
                    :key="key"
                    type="primary"
                    :title="item.url"
                    :class="{
                      'font-green': config.flag === '2',
                      'font-red': config.flag === '1'
                    }"
                    >【
                    <a
                      charset="UTF-8"
                      :href="item.url"
                      target="_blank"
                      :download="item.name"
                      >{{ item.name }}</a
                    >】
                  </el-link>
                </template>
              </template>
              <template v-else>-</template>
            </template>
            <span
              v-else
              :class="{
                'font-green': config.flag === '2',
                'font-red': config.flag === '1',
                'font-red':
                  scope.row['tableDiff'] &&
                  scope.row['tableDiff'].tableEntity &&
                  scope.row['tableDiff'].tableEntity[config.prop] !=
                    undefined &&
                  scope.row[config.prop] !=
                    scope.row['tableDiff'].tableEntity[config.prop]
              }"
              >{{
                scope.row[config.prop]
                  | filterArguments(config.prop, filtersConfig)
              }}{{ config.unit || "" }}</span
            >
          </template>
        </el-table-column>
      </template>
      <slot name="after"></slot>
      <slot name="buttons"></slot>
    </el-table>
    <Pagination
      v-if="showPagination && paginationData.total > 0"
      :total="paginationData.total"
      :size="paginationData.size"
      :current="paginationData.current"
      @pagination="emitFn"
    />
  </div>
</template>

<script>
import Pagination from './Pagination'
export default {
  name: "Table",
  components: {
    Pagination
  },
  props: {
    height: [String, Number],
    tableName: {
      type: String,
      default: () => {
        return "table";
      }
    },
    // 表格列表
    tableList: {
      type: Array,
      default: () => {
        return [];
      }
    },
    // 表格配置
    tableConfig: {
      type: Array,
      default: () => {
        return [];
      }
    },

    // 是否居中
    center: {
      type: Boolean,
      default: () => false
    },
    // 是否加载中
    isLoading: {
      type: Boolean,
      default: () => false
    },
    // 表格字段过滤配置
    filtersConfig: {
      type: Object,
      default: () => {
        return {};
      }
    },
    // 是否需要序号显示
    showIndex: {
      type: Boolean
    },
    // 是否展示checkBox
    showSelection: {
      type: Boolean
    },
    showPagination: {
      type: Boolean
    },
    paginationData: {
      type: Object,
      default: () => {
        return {
          total: 0,
          current: 1,
          size: 10
        };
      }
    },
    checkedRecode: {
      type: Array,
      default: () => {
        return [];
      }
    },
    cellColor: {
      type: Object,
      default: () => {
        return {
          colSet: [],
          rowSet: []
        };
      }
    }
  },
  data() {
    return {
      multipleSelection: []
    };
  },
  watch: {
    tableList: {
      handler(val) {
        if (val) {
          this.$nextTick(() => {
            this.$refs[this.tableName].doLayout();
          });
        }
      },
      deep: true
    },
    filtersConfig: {
      handler(val) {
        if (val) {
          this.$nextTick(() => {
            this.$refs[this.tableName].doLayout();
          });
        }
      },
      deep: true
    },
    tableConfig: {
      handler(val) {
        if (val) {
          this.$nextTick(() => {
            this.$refs[this.tableName].doLayout();
          });
        }
      },
      deep: true
    }
  },
  methods: {
    initCheckedRecode(rows) {
      if (this.tableConfig[0].type != "selection") return;
      try {
        if (rows && rows.length > 0) {
          this.$refs[this.tableName].clearSelection();
          rows.forEach(row => {
            console.log("row", row);
            this.$refs[this.tableName].toggleRowSelection(row, true);
          });
        } else {
          this.$refs[this.tableName].clearSelection();
        }
      } catch (e) {
        console.log(e);
      }
    },
    doLayout() {
      this.$nextTick(() => {
        this.$refs[this.tableName].doLayout();
      });
    },
    rowKeyFunc(row) {
      // console.log(row)
      if (this.tableConfig[0].type != "selection") return;
      return row["unionId"] || row.cid || row.id;
    },
    jsonParseMethod(value) {
      let box = [];
      try {
        box = JSON.parse(value);
      } catch (e) {
        console.log(e);
      }
      if (typeof box !== "object") {
        return [];
      }
      return box;
    },
    emitFn(data) {
      this.$emit("pagination", data);
    },
    // 表格列变颜色方法
    tableCellClassName(cell) {
      return this.cellColor["colSet"][cell.columnIndex];
    },
    tableRowClassName(row) {
      return this.cellColor["rowSet"][row.columnIndex];
    },
    // conditionName 条件字段名称
    // conditionValue 条件字段值
    // row: 当前行数据, index：下标, confKey：配置字段健值, conf：配置对象
    // 方法：selectable ， 是否可以选择 （用于表格第一列多选功能）
    selectable(row, index, confKey, conf) {
      if (!conf.conditionName) return true;
      if (row[conf.conditionName] == conf.conditionValue) {
        return false;
      } else {
        return true;
      }
    },
    handleSelectionChange(val) {
      this.checkedRecode.splice(0, this.checkedRecode.length);
      for (let key in val) {
        this.checkedRecode.push(val[key]);
      }
    }
  },
  filters: {
    filterArguments(value, prop, config) {
      if (value === "" || value === undefined) return "-";
      if (config && config[prop]) {
        return config[prop]["" + value.toString()] || value;
      } else {
        return value;
      }
    },
    fileName(value, k) {
      let key = k || "name";
      if (value === "" || value === undefined) return "-";
      if (typeof value === "string") {
        let temp = "-";
        try {
          temp = JSON.parse(value)[0][key] || "-";
        } catch (e) {
          console.log(e);
        }
        return temp;
      }
    }
  },
  mounted() {}
};
</script>
<style scoped>
  .table   /deep/ .el-table td{
      border-bottom: 1px solid #ebeef5 !important;
  }

 .table   .el-table th.is-leaf {
    border-bottom: 1px solid #ebeef5 !important;
  }
  /* .el-table th.gutter {
    display: table-cell !important;
  }

  /deep/ .el-table colgroup.gutter {
    display: table-cell !important;
  }*/
  .link-div .link-gray:hover {
     cursor: default;
    }
  .link-div .link-gray .el-link--inner {
     text-decoration: none;
          color: #666;
  }
  .link-div .link-gray /deep/ .el-link--inner {
      text-decoration: none;
        color: #666;
  }
   .link-div .link-gray /deep/ .el-link--inner {
      text-decoration: none;
        color: #666;
  }
    .link-div .link-gray /deep/ .el-link--inner  /deep/ el-link:hover {
       text-decoration: none;
          color: #666;
    }
  .font-red {
    color: red;
  }
  .font-green {
    color: green;
  }
</style>
