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

  import {RouterView,RouterLink,useRouter} from 'vue-router'

  const router = useRouter()

  import Menu from "./components/Menu.vue"
  import Outline from "./components/Outline.vue"
  import SelectionBubbleMenu from "./components/SelectionBubbleMenu.vue"

  import { useEditorStore } from '@/store'
  const editorStore = useEditorStore()

  const editor = useEditor({
    content: "我正在使用 Vue.js 运行 Tiptap。",
    extensions: [
      StarterKit,
      Highlight,
      Subscript,
      Superscript,
      Underline,
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
  
  code {
    background-color: var(--purple-light);
    border-radius: 0.4rem;
    color: var(--black);
    font-size: 0.85rem;
    padding: 0.25em 0.3em;
  }
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
  .flex-grow {
    flex-grow: 1;
  }
  .ProseMirror:focus {
    outline: none;
  }
  .ProseMirror {
    overflow-y: scroll;
  }
</style>
