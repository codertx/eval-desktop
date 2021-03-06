<template>
  <el-dialog
    title="代码仓库"
    :visible.sync="dialogVisible"
    width="400px"
    class="snippet-dialog"
    center
  >
    <div
      v-loading="loading"
    >
      <div class="empty-text" v-if="snippetList.length === 0">
        当前没有代码哦，快去创建一个吧
      </div>
      <el-card
        v-for="snippet in snippetList"
        :body-style="{ padding: '15px 15px 5px' }"
        class="snippet-item"
        shadow="hover"
        :key="snippet.id"
      >
        <div
          class="name"
          @click="fetch(snippet)"
        >
          {{snippet.get('name')}}
        </div>
        <div class="meta">
          <time class="time">{{ formatDate(snippet.get('createdAt')) }}</time>
          <el-tag size="mini">{{snippet.get('js_pre')}}</el-tag>
          <el-tag size="mini" type="success">{{snippet.get('css_pre')}}</el-tag>
          <el-button class="del" type="text" size="mini" @click="del(snippet)">
            <span>
              <i class="eval-icon del"></i>
            </span>
          </el-button>
        </div>
      </el-card>
      <el-pagination
        class="snippet-page"
        small
        layout="prev, pager, next"
        @current-change="pageSwitch"
        :page-count="totalPage">
      </el-pagination>
    </div>
  </el-dialog>
</template>

<script>
import Event from '@/util/event'
import { fetch } from '@/store/resource'

function appendZero (num) {
  if (!(String(num).length - 1)) {
    return '0' + String(num)
  } else {
    return num
  }
}

export default {
  data () {
    return {
      dialogVisible: false,
      snippetList: [],
      totalSnippet: 0,
      totalPage: 0,
      currentPage: 1,
      pageSize: 5,
      loading: true
    }
  },

  watch: {
    dialogVisible (newVal) {
      if (newVal) {
        this.list()
      }
    }
  },

  methods: {
    show () {
      this.dialogVisible = true
    },
    list () {
      this.loading = true
      Promise
        .all([this.$snippet.list(this.currentPage, this.pageSize), this.$snippet.meta(this.pageSize)])
        .then((result) => {
          let [list, meta] = result

          this.snippetList = list
          this.totalSnippet = meta[0]
          this.totalPage = meta[1]
          this.loading = false
        })
    },
    async del (snippet) {
      if (this.$snippet.snippet) {
        if (this.$snippet.snippet.get('id') === snippet.get('id')) {
          this.$snippet.destroy()
        }
      }

      try {
        await this.deleteConfirm()
        this.$snippet
          .delete(snippet)
          .then(() => {
            this.$msg.success('删除成功')
            this.list()
          })
      } catch (_) {}
    },
    fetch (snippet) {
      let loading = this.$loading({
        lock: true,
        fullscreen: true,
        text: '载入中',
        background: 'rgba(255, 255, 255, 0.8)'
      })
      this.$snippet.createWithInstance(snippet)
      this.$snippet
        .fetch()
        .then((snippet) => {
          Event.$emit('snippet-javascript', {
            code: snippet.get('js_code'),
            pre: snippet.get('js_pre')
          })
          Event.$emit('snippet-css', {
            code: snippet.get('css_code'),
            pre: snippet.get('css_pre')
          })
          Event.$emit('snippet-html', {
            code: snippet.get('html_code')
          })
          fetch(snippet.get('external_list'))
          this.dialogVisible = false
          loading.close()
        })
    },
    formatDate (date) {
      date = new Date(date)
      return `${date.getFullYear()}-${appendZero(date.getMonth() + 1)}-${appendZero(date.getDate())} ${appendZero(date.getHours())}:${appendZero(date.getMinutes())}`
    },
    pageSwitch (page) {
      this.currentPage = page
      this.snippet.list(this.currentPage, this.pageSize)
        .then((list) => {
          this.snippetList = list
        })
    },
    deleteConfirm () {
      return this.$confirm('删除该代码操作不可逆，是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
    }
  }
}
</script>

<style lang="sass">
.snippet-dialog
  .el-dialog__body
    padding: 10px 25px
  .empty-text
    text-align: center
    line-height: 3em

.snippet-page
  margin-top: 10px

.snippet-item
  cursor: pointer
  user-select: none
  & + &
    margin-top: .5em
  .name
    font-size: 16px
    font-weight: bold
    transition: color .3s ease-out
    &:hover
      color: #409EFF
  .time
    font-size: 12px
    color: #999
    margin-right: 2em
  .del
    float: right
    color: #666
</style>
