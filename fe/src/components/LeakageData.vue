<template>
  <div>
    <el-card>
      <div slot="header">
        <el-radio-group v-model="filters.status" @change="handleFilter">
          <el-radio-button :label="{}">不限</el-radio-button>
          <el-radio-button :label="{security: 0}"><i class="el-icon-time"></i> 待处理
          </el-radio-button>
          <el-radio-button :label="{security: 0, desc: {
                $exists: true}}">
            <i class="el-icon-star-on"></i>
            正在处理
          </el-radio-button>
          <el-radio-button :label="{security: 1}"><i class="el-icon-check"></i> 归档</el-radio-button>
        </el-radio-group>
      </div>
      <el-row>
        <el-col :key="result._id" :xs="24" :sm="24" :md="24" :lg="24" v-for="result in leakagesData">
          <el-card class="box-card">
            <el-button size="mini">
              <router-link :to="'/view/leakage/'+result._id">
                <i class="el-icon-d-arrow-left"></i>
                <i class="el-icon-more"></i>
                <i class="el-icon-d-arrow-right"></i>
              </router-link>
            </el-button>
            <span class="float-right f6 text-gray">{{result.language}}</span>
            <el-tag type="success">
              <i class="el-icon-time"></i>
              {{result.datetime | timeago}}
            </el-tag>

            <el-tag type="gray">
              <router-link :to="'/view/tag/'+result.tag">
                {{result.tag}}
              </router-link>
            </el-tag>

            <el-tag type="success" v-if="result.security">
              <a target="_blank" :href="'https://github.com/'+result.project">{{result.project}}</a>
            </el-tag>
            <el-tag type="danger" v-else="result.security==1">
              <a target="_blank" :href="'https://github.com/'+result.project">{{result.project}}</a>
            </el-tag>

            <el-tag type="warning">
              <a target="_blank" :href="result.link">{{result.filename.split('/').pop()}}</a>
            </el-tag>
            <el-tag type="gray" v-if="result.desc">
              {{result.desc | b64decode}}
            </el-tag>

          </el-card>
        </el-col>
      </el-row>
      <div class="page" v-if="leakagesData">
        <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
                       :current-page="from" :page-sizes="[5,10, 20, 50]" :page-size="limit"
                       layout="total, sizes, prev, pager, next, jumper" :total="total">
        </el-pagination>

      </div>
      <div v-if="leakagesData.length===0">
        <el-row :gutter="20">
          <el-col :span="12" :offset="6">
            <img src="./../assets/no.png" width="500" height="300">
          </el-col>
        </el-row>
      </div>
    </el-card>
  </div>

</template>

<script>
  import Timeago from 'timeago.js';
  import {Base64} from 'js-base64';

  const timeagoInstance = new Timeago();

  export default {
    data() {
      return {
        filters: {status: {}, tag: this.$route.params.tag || ''},
        leakagesData: [],
        total: 100,
        limit: 10,
        from: 1
      }
    }, methods: {
      handleFilter(){
        this.from = 1;
        this.fetchLeakagesData();

      },
      handleSizeChange(val) {
        this.limit = val;
        this.from = 1;
        this.fetchLeakagesData();
      },
      handleCurrentChange(val) {
        this.from = val;
        this.fetchLeakagesData();
      },
      fetchLeakagesData() {
        this.axios.get(this.GLOBAL.leakage,
          {
            params: {
              status: this.filters.status,
              tag: this.filters.tag,
              limit: this.limit,
              from: this.from,
            }
          })
          .then((response) => {
            this.$message({
              message: response.data.msg,
              type: 'success'
            });
            this.leakagesData = response.data.result;
            this.total = response.data.total;

          })
          .catch((error) => {
            console.log(error);
          });
      }
    }, filters: {
      timeago(val) {
        return timeagoInstance.format(val, 'zh_CN')
      },
      b64decode(val) {
        return Base64.decode(val)
      },
      more(val) {
        if (val.split('.').length > 4) {
          return val.split('.').pop()
        } else {
          return val
        }
      }
    },
    mounted() {
      this.fetchLeakagesData();
      this.$nextTick(function () {

      });
    }
  }
</script>

<style>
  .float-right {
    padding-top: 0;
    float: right !important;
  }

  .text-gray {
    color: #586069 !important;
  }

  .f6 {
    font-size: 12px !important;
  }

  .page {
    margin-top: 20px;
    margin-bottom: 20px;
    float: right;
  }
</style>
