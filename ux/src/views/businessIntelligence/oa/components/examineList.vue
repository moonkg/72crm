<template>
  <slide-view
    v-loading="loading"
    :listener-ids="listenerIDs"
    :no-listener-ids="noListenerIDs"
    :no-listener-class="noListenerClass"
    :body-style="{padding: 0, height: '100%'}"
    class="d-view"
    @side-close="hideView">
    <div
      v-if="!loading"
      class="t-section">
      <span class="t-name">{{ name }}</span>&nbsp;
      <span class="t-des">({{ name }}申请：<span class="t-value">{{ totalCount }}次</span></span>
      <span
        v-if="showDes"
        class="t-des">&nbsp;&nbsp;&nbsp;&nbsp;{{ desInfo }}：<span class="t-value">{{ sumData + desUnit }}</span></span><span class="t-des">) </span>
      <img
        class="t-close"
        src="@/assets/img/task_close.png"
        @click="hideView" >
    </div>

    <examine-section
      id="oa-log-statistics"
      :list="list"
      class="t-content"
      @handle="refreshList">
      <p
        slot="load"
        class="load">
        <el-button
          :loading="loadMoreLoading"
          type="text">{{ loadMoreLoading ? '加载更多' : '没有更多了' }}</el-button>
      </p>
    </examine-section>
  </slide-view>
</template>

<script>
import SlideView from '@/components/SlideView'
import ExamineSection from '@/views/OAManagement/examine/components/examineSection'

export default {
  /** 审批统计 列表 */
  name: 'ExamineList',

  components: {
    SlideView,
    ExamineSection
  },

  props: {
    request: Function,
    params: Object,
    // 监听的dom 进行隐藏详情
    listenerIDs: {
      type: Array,
      default: () => {
        return ['crm-main-container']
      }
    },
    // 不监听
    noListenerIDs: {
      type: Array,
      default: () => {
        return []
      }
    },
    noListenerClass: {
      type: Array,
      default: () => {
        return ['el-table__body']
      }
    }
  },

  data() {
    return {
      loading: false,
      loadMoreLoading: true,
      name: '',
      totalCount: 0,
      sumData: '',
      // 判断是否发请求
      isPost: false,
      page: 1,
      list: []
    }
  },

  computed: {
    // 展示说明信息
    showDes() {
      return this.params.category_id > 1 && this.params.category_id <= 6
    },
    // 说明信息
    desInfo() {
      if (this.params.category_id == 2) {
        return '请假总天数'
      } else if (this.params.category_id == 3) {
        return '出差总天数'
      } else if (this.params.category_id == 4) {
        return '加班总天数'
      } else if (this.params.category_id == 5) {
        return '报销总金额'
      } else if (this.params.category_id == 6) {
        return '借款总金额'
      }
      return ''
    },
    // 说明单位
    desUnit() {
      if (this.params.category_id > 1 && this.params.category_id <= 4) {
        return '天'
      }
      return '元'
    }
  },

  watch: {
    params() {
      this.refreshList()
    }
  },

  mounted() {
    // 分批次加载
    const dom = document.getElementById('oa-log-statistics')
    dom.onscroll = () => {
      const scrollOff = dom.scrollTop + dom.clientHeight - dom.scrollHeight
      // 滚动条到底部的条件
      if (Math.abs(scrollOff) < 10 && this.loadMoreLoading == true) {
        if (!this.isPost) {
          this.isPost = true
          this.page++
          this.getList()
        } else {
          this.loadMoreLoading = false
        }
      }
    }
    this.getList()
  },

  methods: {
    refreshList() {
      this.page = 1
      this.list = []
      this.name = ''
      this.sumData = ''
      this.totalCount = 0
      this.getList()
    },

    /**
     * 获取列表
     */
    getList() {
      this.loading = true
      this.request({ page: this.page, limit: 15, ...this.params })
        .then(res => {
          this.name = res.data.categoryName
          this.sumData = res.data.sumData
          this.list = this.list.concat(res.data.list.list)
          this.totalCount = res.data.list.dataCount

          if (res.data.list.list.length < 15) {
            this.loadMoreLoading = false
          } else {
            this.loadMoreLoading = true
          }
          this.isPost = false
          this.loading = false
        })
        .catch(() => {
          this.isPost = false
          this.loading = false
        })
    },

    /**
     * 点击关闭按钮隐藏视图
     */
    hideView() {
      this.$emit('hide')
    }
  }
}
</script>

<style lang="scss" scoped>
.t-section {
  position: relative;
  padding: 10px 17px;
  min-height: 60px;
  line-height: 40px;
  .t-name {
    font-size: 16px;
    color: #333333;
  }

  .t-des {
    color: #999;
    .t-value {
      color: #f3a633;
    }
  }

  .t-close {
    display: block;
    float: right;
    width: 40px;
    height: 40px;
    margin-left: 20px;
    padding: 10px;
    cursor: pointer;
  }
}

.load {
  color: #999;
  font-size: 13px;
  margin: 0 auto 15px;
  text-align: center;
  .el-button,
  .el-button:focus {
    color: #ccc;
    cursor: auto;
  }
}

.d-view {
  position: fixed;
  width: 950px;
  top: 60px;
  bottom: 0px;
  right: 0px;
}

.t-content {
  margin: 0 30px;
  height: calc(100% - 80px);
  overflow-y: auto;
}
</style>
