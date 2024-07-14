<template>
  <div class="menu">
    <div>
      <el-row>
        <el-col :span="12">
          <MenuGroup :title="'文本处理:'" :items="textHandleItems"/>
        </el-col>
        <el-col :span="12">
          <MenuGroup :title="'高亮标记:'" :items="highlightItems"/>
        </el-col>
      </el-row>
      
      <el-row>
        <el-col :span="12">
          <MenuGroup :title="'标题:'" :items="headerItems"/>
        </el-col>
        <el-col :span="12">
          <MenuGroup :title="'常用功能:'" :items="commonFunctionItems"/>
        </el-col>
      </el-row>
      <el-row>
        <span class="title">AI助手：</span>
        <button v-for="item in AIItems"
          class="menu-item"
          @click="item.action(item.url)"
          :title="item.title"
          >
          <svg class="remix">
            <use :xlink:href="`${remixiconUrl}#ri-${item.icon}`" />
          </svg>
        </button>
      </el-row>
    </div>
  </div>
</template>

<script setup lang="ts" name="Menu">
  import { Editor } from '@tiptap/vue-3'
  import remixiconUrl from 'remixicon/fonts/remixicon.symbol.svg'
  import MenuGroup from './MenuGroup.vue'

  const props = defineProps<{ editor: Editor, showUploadDialog: Function, showVoiceInput: Function }>()

  const textHandleItems = [
    {
      icon: 'bold',
      title: 'Bold',
      action: () => props.editor?.chain().focus().toggleBold().run(),
      isActive: () => props.editor?.isActive('bold')
    },
    {
      icon: 'italic',
      title: 'Italic',
      action: () => props.editor?.chain().focus().toggleItalic().run(),
      isActive: () => props.editor?.isActive('italic')
    },
    {
      icon: 'strikethrough',
      title: 'Strike',
      action: () => props.editor?.chain().focus().toggleStrike().run(),
      isActive: () => props.editor?.isActive('strike')
    },
    {
      icon: 'code-view',
      title: 'Code',
      action: () => props.editor?.chain().focus().toggleCode().run(),
      isActive: () => props.editor?.isActive('code')
    },
    {
      icon: 'subscript',
      title: 'Subscript',
      action: () => props.editor?.chain().focus().toggleSubscript().run(),
      isActive: () => props.editor?.isActive('subscript')
    },
    {
      icon: 'superscript',
      title: 'Superscript',
      action: () => props.editor?.chain().focus().toggleSuperscript().run(),
      isActive: () => props.editor?.isActive('superscript')
    },
    {
      icon: 'underline',
      title: 'Underline',
      action: () => props.editor?.chain().focus().toggleUnderline().run(),
      isActive: () => props.editor?.isActive('underline')
    },
    {
      icon: 'arrow-go-back-line',
      title: 'Undo',
      action: () => props.editor?.chain().focus().undo().run()
    },
    {
      icon: 'arrow-go-forward-line',
      title: 'Redo',
      action: () => props.editor?.chain().focus().redo().run()
    }
  ]

  const highlightItems = [
    {
      icon: 'mark-pen-line',
      title: 'Highlight',
      action: () => props.editor?.chain().focus().toggleHighlight().run(),
      isActive: () => props.editor?.isActive('highlight')
    },
    // {
    //   icon: 'external-link-line',
    //   title: 'ExternalLink',
    //   action: () => props.editor?.chain().focus().toggleCode().run(),
    //   isActive: () => props.editor?.isActive('code')
    // },
  ]

  const headerItems = [
    {
      icon: 'h-1',
      title: 'Heading 1',
      action: () => props.editor?.chain().focus().toggleHeading({ level: 1 }).run(),
      isActive: () => props.editor?.isActive('heading', { level: 1 })
    },
    {
      icon: 'h-2',
      title: 'Heading 2',
      action: () => props.editor?.chain().focus().toggleHeading({ level: 2 }).run(),
      isActive: () => props.editor?.isActive('heading', { level: 2 })
    },
    {
      icon: 'h-3',
      title: 'Heading 3',
      action: () => props.editor?.chain().focus().toggleHeading({ level: 3 }).run(),
      isActive: () => props.editor?.isActive('heading', { level: 3 })
    },
    {
      icon: 'h-4',
      title: 'Heading 4',
      action: () => props.editor?.chain().focus().toggleHeading({ level: 4 }).run(),
      isActive: () => props.editor?.isActive('heading', { level: 4})
    },
    {
      icon: 'h-5',
      title: 'Heading 5',
      action: () => props.editor?.chain().focus().toggleHeading({ level: 5 }).run(),
      isActive: () => props.editor?.isActive('heading', { level: 5})
    },
    {
      icon: 'h-6',
      title: 'Heading 6',
      action: () => props.editor?.chain().focus().toggleHeading({ level: 6 }).run(),
      isActive: () => props.editor?.isActive('heading', { level: 6})
    },
  ]

  const commonFunctionItems = [
    {
      icon: 'list-unordered',
      title: 'Bullet List',
      action: () => props.editor?.chain().focus().toggleBulletList().run(),
      isActive: () => props.editor?.isActive('bulletList')
    },
    {
      icon: 'list-ordered',
      title: 'Ordered List',
      action: () => props.editor?.chain().focus().toggleOrderedList().run(),
      isActive: () => props.editor?.isActive('orderedList')
    },
    {
      icon: 'list-check-2',
      title: 'Task List',
      action: () => props.editor?.chain().focus().toggleTaskList().run(),
      isActive: () => props.editor?.isActive('taskList')
    },
  ]

  const AIItems = [
    {
      icon: "character-recognition-line",
      title: "OCR识别",
      action: props.showUploadDialog,
      url: "ocr",
    },
    {
      icon: "mic-line",
      title: "语音识别",
      action: props.showVoiceInput,
      url: "voice-recognise",
    },
    {
      icon: "video-line",
      title: "视频内容识别",
      action: props.showUploadDialog,
      url: "video-recognise",
    }
  ]
</script>

<style lang="scss" scoped>
  .title {
    height: 1.5rem;
    line-height: 1.5rem;
    font-family: "KaiTi", "楷体", "STKaiti", "楷体_GB2312", serif; /* 设置楷体字体 */;
    font-size: 1.4rem;
    /* font-weight: bolder; */
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
</style>

