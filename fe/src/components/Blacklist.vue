<template>

  <el-card class="item">
    <el-tooltip content="项目名称若包含黑名单关键字，则不会抓取" placement="bottom-end">
      <i class="el-icon-information"></i>
    </el-tooltip>
    <el-input
      class="input-new-tag"
      v-if="inputVisible"
      v-model="inputValue"
      ref="saveTagInput"
      size="mini"
      @keyup.enter.native="handleInputNoticeConfirm"
      @blur="handleInputNoticeConfirm"
    >
    </el-input>
    <el-button v-else size="small" @click="showInput">添加</el-button>
    <el-tag
      :key="item"
      v-for="item in blacklists"
      :closable="true"
      :close-transition="true"
      @close="handleDeleteBlacklist(item)"
    >
      {{item.keyword}}
    </el-tag>

  </el-card>


</template>
<script>
  export default {
    data () {
      return {
        blacklists: [],
        enable: false,
        inputVisible: false,
        inputValue: '',
      }
    },
    methods: {
      showInput() {
        this.inputVisible = true;
        this.$nextTick(_ => {
          this.$refs.saveTagInput.$refs.input.focus();
        });
      }, handleInputNoticeConfirm() {
        if (this.inputValue) {
          this.blacklists.push({'keyword': this.inputValue});
          this.axios.post(this.GLOBAL.settingBlacklist, {'keyword': this.inputValue})
            .then((response) => {
              this.$message({
                message: response.data.msg,
                type: 'success'
              });
              this.inputVisible = false;
              this.blacklists = response.data.result;

            })
            .catch((error) => {
              this.$message({
                message: error,
                type: 'error'
              });
              this.inputVisible = false;
              this.inputValue = ''

            });
        }

      },

      handleDeleteBlacklist(item) {
        this.axios.delete(`${this.GLOBAL.settingBlacklist}?keyword=${item.keyword}`)
          .then((response) => {
            this.$message({
              message: response.data.msg,
              type: 'success'
            });
            this.blacklists.splice(this.blacklists.indexOf(item), 1);
            this.inputVisible = false;
            this.blacklists = response.data.result;
          })
          .catch((error) => {
            this.$message({
              message: error,
              type: 'error'
            });
            this.inputVisible = false;
            this.inputValue = ''

          });
      },
      fetchBlacklists(){
        this.axios.get(this.GLOBAL.settingBlacklist)
          .then((response) => {
            this.blacklists = response.data.result;
          })
          .catch((error) => {
            this.$message({
              message: error,
              type: 'error'
            });


          });
      }

    },
    mounted: function () {
      this.fetchBlacklists();
      this.$nextTick(function () {
      });
    }
  }
</script>
