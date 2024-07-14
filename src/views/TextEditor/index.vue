<template>
  <el-container 
    style="height: 100vh; display: flex; flex-direction: column;" 
    ref="fileContRef" 
    @mousedown="notSee($event)" 
  >
    <ul 
      @mousedown="seeMenu()" 
      ref="menuRef"
      v-show="visibleMenu" 
      :style="{ 
        left: position.left + 'px', 
        top: position.top + 'px', 
        display: (visibleMenu ? 'grid' : 'none') 
      }" 
      class="context-menu"
    >
      <div v-for="(value, key) in AIList" :key="key" class="item"  @mousedown="getAIMeaage(key)">
        <svg class="remix">
          <use :xlink:href="`${remixiconUrl}#ri-${value.icon}`" />
        </svg>
        <span class="item-text">{{ value.name }}</span>
      </div>
    </ul>
    <el-card 
      v-show="visibleCard" 
      ref="cardRef"
      :style="{ 
        'max-width': 480 + 'px',
        left: position.left + 'px', 
        top: position.top + 'px', 
        display: (visibleCard ? 'grid' : 'none'),
        position: 'absolute',
      }" 
    >
      <p>{{ cardMsg }}</p>
      <el-button type="primary" @mousedown="replace()">替换</el-button>
      <el-button type="success" @mousedown="append()">追加</el-button>
      <el-button type="info" @mousedown="ignore()">舍弃</el-button>
    </el-card>

    <el-dialog
      v-model="visibleUploadDialog"
      title="上传文件"
      width="50%"
      @close="visibleUploadDialog = false"
    >
      <el-upload
        class="upload-demo"
        drag
        :before-upload="beforeUpload"
      >
        <el-icon class="el-icon--upload"><upload-filled /></el-icon>
        <div class="el-upload__text">
          将文件放在此处或 <em>点击上传</em>
        </div>
        <template #tip>
          <div class="el-upload__tip">
          </div>
        </template>
      </el-upload>
    </el-dialog>

    <el-header>
      <el-menu mode="horizontal" :ellipsis="false">
        <div class="flex-grow" />
        <el-menu-item @click="gotoUserProfile()">
          个人中心
          <svg class="remix">
            <use :xlink:href="`${remixiconUrl}#ri-${'arrow-right-s-line'}`" />
          </svg>
        </el-menu-item>
      </el-menu>
    </el-header>

    <el-container style="flex: 1;">
      <el-aside width="20%" style="height: 100%;">
        <Outline/>
      </el-aside>
      <el-main style="height: 100%;">
        <Menu :editor="editor as Editor" :showUploadDialog="showUploadDialog"/>
        <!-- <SelectionBubbleMenu ref="bubbleRef" :editor="editor as Editor" :showDataModal="showDataModal" /> -->
        <editor-content 
          :editor="editor"
          @scroll="hasScroll()"
          @mousedown="notSee($event)"
          @mousemove="mouseMove()" 
          @mouseup="selectText($event)" 
        />
      </el-main>
    </el-container>
  </el-container>
</template>

<script setup lang="ts" name="TextEditor">
  import { ref, reactive } from "vue"
  import { useEditor, EditorContent, Editor } from "@tiptap/vue-3"
  import StarterKit from '@tiptap/starter-kit'
  import Highlight from '@tiptap/extension-highlight'
  import Subscript from '@tiptap/extension-subscript'
  import Superscript from '@tiptap/extension-superscript'
  import Underline from '@tiptap/extension-underline'
  import Placeholder from '@tiptap/extension-placeholder'
  import TaskItem from '@tiptap/extension-task-item'
  import TaskList from '@tiptap/extension-task-list'
  import remixiconUrl from 'remixicon/fonts/remixicon.symbol.svg'
  import { UploadFilled } from '@element-plus/icons-vue'
  import MenuGroup from './components/MenuGroup.vue'
  import axios from "axios"
  import { ElMessage } from 'element-plus'
  import {RouterView,RouterLink,useRouter} from 'vue-router'

  const router = useRouter()

  import Menu from "./components/Menu.vue"
  import Outline from "./components/Outline.vue"
  import SelectionBubbleMenu from "./components/SelectionBubbleMenu.vue"

  import { useEditorStore } from '@/store'
  import { fa } from "element-plus/es/locales.mjs"
  const editorStore = useEditorStore()
  // 编辑器
  const editor = useEditor({
    // content: "我正在使用 Vue.js 运行 Tiptap。",
    content: ``,
    extensions: [
      StarterKit,
      Highlight,
      Subscript,
      Superscript,
      Underline,
      Placeholder.configure({
        placeholder: '开始输入文本...'
      }),
      TaskList,
      TaskItem.configure({
        nested: true,
      }),
    ],
    onUpdate({ editor }) {
      loadHeadings()
      editorStore.setEditorInstance(editor)
    },
    onCreate({ editor }) {
      loadHeadings()
      editorStore.setEditorInstance(editor)
    },
  });
  // 润色功能
  const AIList = reactive({
    'translate': {name: "翻译", icon: "translate"},
    'abstract': {name: "摘要", icon: "file-text-line"},
    'decorate': {name: "修饰", icon: "magic-line"},
    'continue-write': {name: "续写", icon: "edit-2-line"},
    'rewrite': {name: "病句改写", icon: "refresh-line"},
    'improve-write': {name: "改进写作", icon: "pencil-ruler-line"},
    'summarize': {name: "总结", icon: "book-2-fill"},
    'analysis': {name: "分析内容", icon: "bar-chart-fill"},
  })
  async function getAIMeaage(name: string){
    try {
      console.log("on mounted")
      let data = {
        question: selectionMsg
      }
      const response = await axios.post(
        `/${name}/`,
        JSON.stringify(data),
      )
      // accountError.value = response.data.error
      let res = response.data
      if (res.status){
        console.log(res.answer)
        cardMsg.value = res.answer
        visibleCard.value = true
      } else{
        console.log(res.error)
      }
      console.log('POST 请求成功：', response.data)
      
    } catch (error) {
      console.error('POST 请求失败：', error)
      // throw error // 可选的抛出错误
    }
  }

  const fileContRef = ref(null)
  const visibleMenu = ref(false)
  const visibleCard = ref(false)
  const cardRef = ref()
  const menuRef = ref()
  const cardMsg = ref("")
  const position = ref({
    top: 0,
    left: 0
  })
  var hasMove = ref(false)
  var selectionMsg: any
  var selection: any
  // 获取选中的文字
  const selectText = (e: MouseEvent) => {
    selection = window.getSelection()
    if(selection != null && selectionMsg != selection){
      var content = selection.toString()
      if(content != ""){
          // var rect = fileContRef.value?.getBoundingClientRect()
          visibleMenu.value = true
          // alert(e.clientY)
          // alert(e.clientX)
          position.value.top =  e.clientY
          position.value.left = e.clientX
          selectionMsg = content
        }
      // alert(content)
    }
    else{
      selectionMsg = ""
    }
  }
  //鼠标移动
  const mouseMove = () => {
    hasMove.value = true
  }
  const isMouseNotInRect = (e: MouseEvent, rect: any) => {
    const { clientX: x, clientY: y } = e
    if (x < rect.left || x > rect.right || y < rect.top || y > rect.bottom) {
      return true
    } else return false
  }

  //鼠标点击
  const notSee = (e: MouseEvent) => {
    visibleMenu.value = false
    // if (isMouseNotInRect(e, menuRef.value.$el.getBoundingClientRect())) visibleMenu.value = false
    if (isMouseNotInRect(e, cardRef.value.$el.getBoundingClientRect())) visibleCard.value = false
  }
  const seeMenu = () => {
    visibleMenu.value = true
    // selection.value=""
  }
  //滚轮滚动
  const hasScroll = () => {
    visibleMenu.value = false
    // window.getSelection().removeAllRanges()
  }
  const replace = () => {
    const selection = editor.value?.state.selection
    if (selection) { // 检查 selection 是否为 undefined
      const { from, to } = selection
      if (from !== to) { // 确保有选中的内容
        editor.value?.chain().focus().deleteRange({ from, to }).insertContent(cardMsg.value).run()
        visibleCard.value = false
      }
    } 
  }
  const append = () => {
    const selection = editor.value?.state.selection
    if (selection) { // 检查 selection 是否为 undefined
      const { from, to } = selection
      if (from !== to) { // 确保有选中的内容
        editor.value?.chain().focus().insertContentAt(to, cardMsg.value).run()
        visibleCard.value = false
      }
    } 
  }
  const ignore = () => {
    visibleCard.value = false
  }

    // 获取标题（大纲）
  const loadHeadings = () => {
    const headings = [] as any[]
    if (!editor.value) return
    const transaction = editor.value.state.tr
    if (!transaction) return

    editor.value?.state.doc.descendants((node, pos) => {
      if (node.type.name === 'heading') {
        console.log(pos, node)
        const start = pos
        const end = pos + node.content.size
        // const end = pos + node
        const id = `heading-${headings.length + 1}`
        if (node.attrs.id !== id) {
          transaction?.setNodeMarkup(pos, undefined, {
            ...node.attrs,
            id
          })
        }

        headings.push({
          level: node.attrs.level,
          text: node.textContent,
          start,
          end,
          id
        })
      }
    })

    transaction?.setMeta('addToHistory', false)
    transaction?.setMeta('preventUpdate', true)

    editor.value?.view.dispatch(transaction)
    editorStore.setHeadings(headings)
  }

  function gotoUserProfile()
  {
    router.push({
      path: '/user-profile',
    })
  }

  const visibleUploadDialog = ref(false)
  const uploadUrl = ref("")
  const showUploadDialog = (param: string) => {
    visibleUploadDialog.value = true
    console.log(param)
    uploadUrl.value = param
  }
  const beforeUpload = async (file: any) => {
    ElMessage({
      message: '成功上传文件',
      type: 'success',
      plain: true,
    })
    visibleUploadDialog.value = false
    try {
      console.log("on mounted")
      const formData = new FormData()
      formData.append('file', file)
      const response = await axios.post(
        `/${uploadUrl.value}/`,
        formData,
      )
      let res = response.data
      console.log(res)
      if (res.status){
        // console.log(res.answer)
        // cardMsg.value = res.answer
        // visibleCard.value = true
      } else{
        console.log(res.error)
      }
      console.log('POST 请求成功：', response.data)
    } catch (error) {
      console.error('POST 请求失败：', error)
    }
    return false;
  }


</script>

<style lang="scss" scoped>
  .custom-modal {
    position: fixed;
    width: 500px;
    background-color: #fff;
    padding: 10px;
    border-radius: 4px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.3);
    z-index: 1000; /* 确保弹出框在最上层 */
  }
  .custom-list {
    list-style-type: none;
    padding: 0;
  }

  .flex-grow {
    flex-grow: 1;
  }

  .menu-item {
    background: transparent;
    border: none;
    border-radius: 0.4rem;
    color: #333;
    cursor: pointer;
    height: 1.75rem;
    padding: 0.25rem;
    margin-right: 0.25rem;
    width: 1.75rem;

    svg {
      fill: currentColor;
      height: 100%;
      width: 100%;
    }

    &.is-active,
    &:hover {
      background-color: #d6d6d6;
    }
  }

  .context-menu {
    width: 240px; /* 调整宽度以适应两列布局 */
    margin: 0;
    background: #fff;
    z-index: 3000;
    position: absolute;
    list-style-type: none;
    padding: 5px; /* 调整填充 */
    border-radius: 6px; /* 增加圆角 */
    font-size: 16px;
    font-weight: 400;
    color: #333;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1); /* 优化阴影 */
    display: grid;
    grid-template-columns: 1fr 1fr; /* 两列布局 */
    gap: 10px; /* 增加项之间的间距 */
  }

  .context-menu .item {
    height: 40px; /* 增加高度 */
    display: flex; /* 使用弹性布局 */
    align-items: center; /* 垂直居中 */
    padding: 0 10px; /* 增加内边距 */
    color: rgb(29, 33, 41);
    cursor: pointer;
    border-radius: 4px; /* 增加圆角 */
    background-color: #f9f9f9; /* 添加背景色 */
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1); /* 添加阴影 */
  }

  .context-menu .item-text {
    flex: 1; /* 占据剩余空间，使文本左对齐 */
    text-align: left; /* 确保文本左对齐 */
  }

  .context-menu .item:hover {
    background: rgb(229, 230, 235);
  }

  .remix {
    fill: currentColor;
    width: 1.5rem;
    height: 1.5rem;
    margin-right: 5px; /* 增加与文本的间距 */
  }
</style>

<style lang="scss">

.tiptap .bubble-menu {
    background-color: var(--white);
    border: 1px solid var(--gray-1);
    border-radius: 0.7rem;
    box-shadow: var(--shadow);
    display: flex;
    padding: 0.2rem;

    button {
      background-color: unset;

      &:hover {
        background-color: var(--gray-3);
      }

      &.is-active {
        background-color: var(--purple);

        &:hover {
          background-color: var(--purple-contrast);
        }
      }
    }
  }
b {
  font-weight: bold;
}
.ProseMirror {
  overflow-y: scroll;
}
.ProseMirror p {
  margin: 0;
}
.ProseMirror:focus {
  outline: none;
}
.tiptap p.is-editor-empty:first-child::before {
  color: #adb5bd;
  content: attr(data-placeholder);
  float: left;
  height: 0;
  pointer-events: none;
}

.tiptap {
  > * + * {
    margin-top: 0.75em;
  }

  ul {
    padding: 0 2rem;
    list-style: square;
  }
  ol {
    padding: 0 2rem;
    list-style: decimal;
  }
  table {
    border-collapse: collapse;
    table-layout: fixed;
    width: 100%;
    margin: 0;
    overflow: hidden;

    td,
    th {
      min-width: 1em;
      border: 2px solid #ced4da;
      padding: 3px 5px;
      vertical-align: top;
      box-sizing: border-box;
      position: relative;

      > * {
        margin-bottom: 0;
      }
    }

    th {
      font-weight: bold;
      text-align: left;
      background-color: #f1f3f5;
    }

    .selectedCell:after {
      z-index: 2;
      position: absolute;
      content: '';
      left: 0;
      right: 0;
      top: 0;
      bottom: 0;
      background: rgba(200, 200, 255, 0.4);
      pointer-events: none;
    }

    .column-resize-handle {
      position: absolute;
      right: -2px;
      top: 0;
      bottom: -2px;
      width: 4px;
      background-color: #adf;
      pointer-events: none;
    }

    p {
      margin: 0;
    }
  }
  pre {
    background: #0d0d0d;
    color: #fff;
    font-family: 'JetBrainsMono', monospace;
    padding: 0.75rem 1rem;
    border-radius: 0.5rem;

    code {
      color: inherit;
      padding: 0;
      background: none;
      font-size: 0.8rem;
    }

    .hljs-comment,
    .hljs-quote {
      color: #616161;
    }

    .hljs-variable,
    .hljs-template-variable,
    .hljs-attribute,
    .hljs-tag,
    .hljs-name,
    .hljs-regexp,
    .hljs-link,
    .hljs-name,
    .hljs-selector-id,
    .hljs-selector-class {
      color: #f98181;
    }
    .hljs-number,
    .hljs-meta,
    .hljs-built_in,
    .hljs-builtin-name,
    .hljs-literal,
    .hljs-type,
    .hljs-params {
      color: #fbbc88;
    }

    .hljs-string,
    .hljs-symbol,
    .hljs-bullet {
      color: #b9f18d;
    }

    .hljs-title,
    .hljs-section {
      color: #faf594;
    }

    .hljs-keyword,
    .hljs-selector-tag {
      color: #70cff8;
    }

    .hljs-emphasis {
      font-style: italic;
    }

    .hljs-strong {
      font-weight: 700;
    }
  }
}

.tableWrapper {
  overflow-x: auto;
}

.resize-cursor {
  cursor: ew-resize;
  cursor: col-resize;
}
ul[data-type="taskList"] {
    list-style: none;
    margin-left: 0;
    padding: 0;

    li {
      align-items: flex-start;
      display: flex;

      > label {
        flex: 0 0 auto;
        margin-right: 0.5rem;
        user-select: none;
      }

      > div {
        flex: 1 1 auto;
      }
    }

    input[type="checkbox"] {
      cursor: pointer;
    }

    ul[data-type="taskList"] {
      margin: 0;
    }
  }

</style>

