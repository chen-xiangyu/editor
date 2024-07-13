<template>
  <el-container 
    style="height: 100vh; display: flex; flex-direction: column;" 
    ref="fileContRef" 
    @mousedown="notSee()" 
  >
    <ul 
      @mousedown="see()" 
      v-show="visibleMenu" 
      :style="{ 
        left: position.left + 'px', 
        top: position.top + 'px', 
        display: (visibleMenu ? 'grid' : 'none') 
      }" 
      class="context-menu"
    >
      <div v-for="(value, key) in AIList" :key="key" class="item"  @mousedown="getAIMeaage(key)">
          <!-- <el-icon><Brush /></el-icon> -->
          {{ value }}
      </div>
    </ul>
    <el-header>
      <el-menu mode="horizontal" :ellipsis="false">
        <div class="flex-grow" />
        <el-menu-item @click="gotoUserProfile()">个人中心</el-menu-item>
      </el-menu>
    </el-header>

    <el-container style="flex: 1;">
      <el-aside width="20%" style="height: 100%;">
        <Outline/>
      </el-aside>
      <el-main style="height: 100%;">
        <Menu :editor="editor as Editor"/>
        <!-- <SelectionBubbleMenu ref="bubbleRef" :editor="editor as Editor" :showDataModal="showDataModal" /> -->
        <editor-content 
          :editor="editor"
          @scroll="hasScroll()"
          @mousedown="notSee()"
          @mousemove="mouseMove()" 
          @mouseup="selectText($event)" 
        />
      </el-main>
    </el-container>
  </el-container>
  <div class="custom-modal" v-if="dataModalVisible" :style="modalStyle">
    <div class="modal-content">
      <p>asdsa</p>
    </div>
    <ul class="custom-list">
      <li v-for="(item, index) in AIList" :key="index" @click="getAIMeaage(index)">{{ item }}</li>
    </ul>
  </div>
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
  import remixiconUrl from 'remixicon/fonts/remixicon.symbol.svg'
  import MenuGroup from './components/MenuGroup.vue'
  import axios from "axios"

  import {RouterView,RouterLink,useRouter} from 'vue-router'

  const router = useRouter()

  import Menu from "./components/Menu.vue"
  import Outline from "./components/Outline.vue"
  import SelectionBubbleMenu from "./components/SelectionBubbleMenu.vue"

  import { useEditorStore } from '@/store'
  const editorStore = useEditorStore()

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
  const AIList = reactive({
    'translate': "翻译",
    'abstract': "摘要",
    'decorate': "修饰",
    'continue-write': "续写",
    'rewrite': "病句改写",
    'improve-write': "改进写作",
    'summarize': "总结",
    'analysis': "分析内容",
  });

  // 模态框
  let dataModalVisible = ref(false)
  const modalStyle = ref({})
  function showDataModal(btnBottom: any, btnLeft: any) {
    dataModalVisible.value = true;
    modalStyle.value = {
      top: `${btnBottom}px`,
      left: `${btnLeft}px`,
    }
    // 从编辑器中移除焦点
    // editor.commands.blur();
  }
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

  async function getAIMeaage(name: string)
  {
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
        console.log(res.AIresponse)
      } else{
        console.log(res.error)
      }
      console.log('POST 请求成功：', response.data)
      
    } catch (error) {
      console.error('POST 请求失败：', error)
      // throw error // 可选的抛出错误
    }

  }
  
  async function setAccessToken(){
    try {
      console.log("on mounted")
      let data = {
        accessToken: formInfo.AccessToken,
      }
      const response = await axios.post(
        '/set-access-token/',
        JSON.stringify(data),
      )
      // accountError.value = response.data.error
      let res = response.data
      if (res.status){
        ElMessage({
          message: res.message,
          type: 'success',
        })
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
  const position = ref({
    top: 0,
    left: 0
  })
  var hasMove = ref(false)
  var selectionMsg: any
  var selection: any
  //进行润色的函数
  function polish(name: string){
    console.log("@", name)
        // ailoading.value=true
        // visibleMenu.value = false;
        // let formData = new FormData();
        // formData.append("username","123456");
        // formData.append("key","xxxxxxx");
        // formData.append("cont",selectionMsg);
        // let url = 'http://127.0.0.1:5000/getpolish' //访问后端接口的url
        // let method = 'post'
        // axios({
        //   method,
        //   url,
        //   data: formData,
        // }).then(res => {
        //   console.log(res.data);
        //   var tpcard1={"title":"ai辅助评审","cont":selectionMsg,"review":res.data}
        //   ailist.value.push(tpcard1)
        //   navigator.clipboard.writeText(res.data)
        //   showMessage()
        //   ailoading.value=false
        // });
    }
  //进行aireview
  // const continuation=()=>{
  //   console.log("@")
  //       // ailoading.value=true
  //       // visibleMenu.value = false;
  //       // let formData = new FormData();
  //       // formData.append("username","123456");
  //       // formData.append("key","xxxxxxx");
  //       // formData.append("cont",selectionMsg);
  //       // let url = 'http://127.0.0.1:5000/getpolish' //访问后端接口的url
  //       // let method = 'post'
  //       // axios({
  //       //   method,
  //       //   url,
  //       //   data: formData,
  //       // }).then(res => {
  //       //   console.log(res.data);
        
  //       //   showMessage()
  //       //   ailoading.value=false
  //       // });
  //   }
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
  //鼠标点击
  const notSee = () => {
    visibleMenu.value = false
    // selection.value=""
  }
  const see = () => {
    visibleMenu.value = true
    // selection.value=""
  }
  //滚轮滚动
  const hasScroll = () => {
    visibleMenu.value = false
    // window.getSelection().removeAllRanges()
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
    width: 120px;
    margin: 0;
    background: #fff;
    z-index: 3000;
    position: absolute;
    list-style-type: none;
    padding:5px;
    padding-left: 15px;
    border-radius: 4px;
    font-size: 12px;
    font-weight: 400;
    color: #333;
    box-shadow: 1px 1px 2px 1px rgba(0, 0, 0, 0.3);
    display: grid;
    grid-template-columns:50% 50%;

  }
  .context-menu .item {
      height: 35px;
      width:100%;
      line-height: 35px;
      color: rgb(29, 33, 41);
      cursor: pointer;
    }
    .context-menu .item {
      height: 35px;
      width:100%;
      line-height: 35px;
      color: rgb(29, 33, 41);
      cursor: pointer;
    }

    .context-menu .item:hover {
      background: rgb(229, 230, 235);
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
</style>

