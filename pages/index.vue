/<template>
  <client-only>
    <div class="html2md-box">
      <div class="title-view">
        <div>
          <h1 class="h1"><img src="https://ico.vercel.app/html5/000" width="40" alt="html"> ➔ <img src="https://ico.vercel.app/markdown/000" width="60" alt="markdown"></h1>
        </div>
        <a href="https://github.com/helloworld-Co/html2md" target="_blank">
          <img width="40" src="/img/github.svg">
        </a>
      </div>

      <div class="url-box">
        <el-input
          @keyup.enter.native="transformUrl"
          @focus="getInputFocus($event)"
          v-model="url"
          size="small"
          placeholder="https://" />
        <div>&nbsp;</div>
        <el-button
          @click="transformUrl"
          :loading="isLoading"
          size="small"
          type="primary"
          class="newButton"
        >URL ➔ <markdown-icon class="newButton svgIcon"></markdown-icon>
        </el-button>
      </div>
      <br/>
      <el-input
        @focus="getInputFocus($event)"
        v-model="title"
        size="small"
        placeholder="Page Title..." />
      <br/>

      <div class="input-box">
        <mavon-editor ref="editor" v-model="md" @save="toDownload"/>
      </div>
    </div>
  </client-only>
</template>
<script>
import TurndownService from 'turndown'
import { gfm, tables, strikethrough } from 'turndown-plugin-gfm'

import { MarkdownIcon } from 'vue-simple-icons'

export default {
  name: 'Html2mdBox',
  components: {
    MarkdownIcon
  },
  data () {
    return {
      html: '<pre>Hello world!<br/>🚀🌎💩</pre>',
      // html: markdownSyntax,
      url: '',
      isLoading: false,
      title: ''
    }
  },
  computed: {
    md: {
      get () {
        return this.html2md(this.html)
      },
      set (val) {

      }
    }
  },
  methods: {
    downLoadFile (url) {
      const a = document.createElement('a')
      a.download = `${Date.now()}.md`
      a.href = url
      a.click()
    },
    toDownload () {
      const params = {
        md: this.md,
        url: window.location.origin
      }
      this.$axios.post(`${window.location.origin}/getMdFile`, params)
        .then((res) => {
          this.downLoadFile(res.path)
        })
    },
    html2md (str) {
      const turndownService = new TurndownService({ codeBlockStyle: 'fenced' })
      // Use the gfm plugin
      turndownService.use(gfm)

      // Use the table and strikethrough plugins only
      turndownService.use([tables, strikethrough])
      // 自定义配置
      turndownService.addRule('pre2Code', {
        filter: ['pre'],
        replacement (content) {
          const len = content.length
          // 除了pre标签，里面是否还有code标签包裹，有的话去掉首尾的`（针对微信文章）
          const isCode = content[0] === '`' && content[len - 1] === '`'
          const result = isCode ? content.substr(1, len - 2) : content
          return '```\n' + result + '\n```\n'
        }
      })
      const markdown = turndownService.turndown(str)
      return markdown
    },
    getInputFocus (event) {
      event.currentTarget.select()
    },
    transformUrl () {
      if (!this.url) { return }
      this.isLoading = true
      this.$axios.get(`${window.location.origin}/getUrlHtml`, { params: { url: this.url } })
        .then((res) => {
          this.html = res.html
          this.title = res.title
          this.isLoading = false
        }).catch(() => {
          this.isLoading = false
        })
    }
  }
}

</script>

<style lang="scss">
  .svgIcon {
  filter: invert(100%) sepia(2%) saturate(1760%) hue-rotate(179deg) brightness(99%) contrast(106%);
  color: white;
  }
  .title-view {
  padding-top: 1em;
}
  .newButton {
  display: inline-block;
  vertical-align: middle;
  font-weight: bold;
  }
  .html2md-box{
    height: 100vh;
    padding: 0 30px;
    overflow: hidden;
    .title-view{
      display: flex;
      align-items: center;
      justify-content: space-between;
      padding-bottom: 10px;
      padding-top: 10px;
      .h1{
        text-align: center;
        font-size: 30px;
      }
      .h3{
        text-align: center;
        color: #666;
      }
    }
    .url-box{
      display: flex;
      align-items: center;
      justify-content: space-between;
    }
    .input-box{
      margin: 20px auto;
      min-height: 80vh;
      display: flex;
      justify-content: space-between;
      align-items: flex-start;
    }
    .v-note-wrapper{
      &.shadow{
        width: 100%;
        height: 74vh;
      }
      &.fullscreen{
        height: auto!important;;
      }
    }
  }

</style>
