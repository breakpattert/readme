<template>
  <div class="app-container">
    <div class="filter-container">
   
     <el-input  v-model="listQuery.nick_name"placeholder="昵称" class="filter-item" style="width:130px"></el-input>

      <el-date-picker
          class="filter-item"
          v-model="dateSelect"
          type="daterange"
          align="right"
          placeholder="创建时间"
          format="yyyy-MM-dd"
          @change="updateListQueryDate"
          :default-value="dateSelect"
          :editable="false"
          :clearable="false"
          >
      </el-date-picker>

      <el-button class="filter-item" type="primary" v-waves icon="search" @click="handleFilter">搜索</el-button>

    </div>

    <el-table  :key='tableKey' :data="list" v-loading.body="listLoading" border fit highlight-current-row style="width: 100%">
      <el-table-column align="center" label="序号" width="100" type="index">
      </el-table-column>

      <el-table-column  label="用户名"  align="center">
        <template scope="scope">
          <span>{{scope.row.nick_name}}</span>
        </template>
      </el-table-column>

       <el-table-column class-name="status-col" label="操作记录" align="center">
        <template scope="scope">
          <span>{{scope.row.birthday}}</span>
        </template>
      </el-table-column>

      <el-table-column  label="创建时间" align="center">
        <template scope="scope">
          <span>{{scope.row.gmt_created | parseTime}}</span>
        </template>
      </el-table-column>
    </el-table>

    <div v-show="!listLoading" class="pagination-container">
      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page.sync="listQuery.page" :page-sizes="[10]"
        :page-size="listQuery.limit" layout="total, sizes, prev, pager, next, jumper" :total="total">
      </el-pagination>
    </div>

  </div>
</template>

<script>
    import {list} from 'api/users';
    import { genderListFilter } from '@/filters';
    import { parseTime } from 'utils';
    import { Message } from 'element-ui';

    export default {
      name: 'users',
      data() {
        return {
          dateSelect: [],
          list: null,
          total: null,
          listLoading: true,
          listQuery: {
            begin_date: null,
            end_date: null,
        
            page_size: 10,
            current_page: 1,
            nick_name:'',
          },
          tableKey: 0
        }
      },
      components: {
      },
      created() {
        var now = Date.now(), lastWeek = now - 3600 * 1000 * 24 * 7;
        var s_time = parseTime(lastWeek), e_time = parseTime(now);
        this.dateSelect = [s_time, e_time];
        this.updateListQueryDate();
        this.getList();
      },
      filters: {
        parseTime,
        genderListFilter
      },
      methods: {
        updateListQueryDate(){
          var dateSelect = this.dateSelect;
          if(dateSelect.length == 2){
            this.listQuery.begin_date = parseTime(dateSelect[0]);
            this.listQuery.end_date = parseTime(dateSelect[1]);
          }
        },
        getList() {
          this.listLoading = true;
          list(this.listQuery).then(res => {
            var d = res.data;
//          console.log(d)
            if(d.data && d.data.pagination_d_t_o && d.data.pagination_d_t_o.datas){
              var data = d.data.pagination_d_t_o;
              this.list = data.datas;
              console.log(this.list);
              this.total = data.total_count;
            }else{
              this.list = [];
              this.total = 0;
            }
            this.listLoading = false;
          })
        },

        handleFilter() {
          this.getList();
        },
        handleSizeChange(val) {
          this.listQuery.limit = val;
          this.getList();
        },
        handleCurrentChange(val) {
          this.listQuery.current_page = val;
          this.getList();
        }

      }
    }
</script>
