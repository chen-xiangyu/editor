<template>
  <el-container style="height: 100vh; display: flex; flex-direction: column;">
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
        <Menu :editor="editor as Editor" />
        <SelectionBubbleMenu :editor="editor as Editor" />
        <editor-content :editor="editor" />
      </el-main>
    </el-container>
  </el-container>
</template>

<script setup lang="ts" name="TextEditor">
  import { ref } from "vue"
  import { useEditor, EditorContent, Editor } from "@tiptap/vue-3"
  import StarterKit from '@tiptap/starter-kit'
  import Highlight from '@tiptap/extension-highlight'
  import Subscript from '@tiptap/extension-subscript'
  import Superscript from '@tiptap/extension-superscript'
  import Underline from '@tiptap/extension-underline'
  import Placeholder from '@tiptap/extension-placeholder'

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
</script>

<style lang="scss" scoped>
  
  // code {
  //   background-color: var(--purple-light);
  //   border-radius: 0.4rem;
  //   color: var(--black);
  //   font-size: 0.85rem;
  //   padding: 0.25em 0.3em;
  // }

  .flex-grow {
    flex-grow: 1;
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

