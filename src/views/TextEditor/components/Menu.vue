<template>
  <div class="menu">
    <div>
      <el-row>
        <el-col :span="12">
          <MenuGroup :title="'文本处理:'" :items="textHandleItems"/>
        </el-col>
        <el-col :span="12">
          <!-- <MenuGroup :title="'高亮标记:'" :items="highlightItems"/> -->
          <MenuGroup :title="'层次排版:'" :items="headerItems"/>
         
        </el-col>
      </el-row>
        
      <el-row>
        <el-col :span="12">
          <MenuGroup :title="'常用功能:'" :items="commonFunctionItems"/>
        </el-col>
        <el-col :span="12">
          <span class="title">AI助手：</span>
          <template v-for="item in AIItems">
            <el-tooltip
              class="box-item"
              effect="dark"
              :content="item.title"
              placement="top"
              hide-after="10"
            >
              <button 
                class="menu-item"
                @click="item.action(item.params)"
                >
                <svg class="remix">
                  <use :xlink:href="`${remixiconUrl}#ri-${item.icon}`" />
                </svg>
              </button>
            </el-tooltip>
          </template>
        </el-col>
      </el-row>
      <input ref="imageFileInput" type="file" accept="image/jpeg, image/png, image/gif" style="display: none" @change="handleImageFile">
    </div>
  </div>
</template>

<script setup lang="ts" name="Menu">
  import { ref } from "vue"
  import { Editor } from '@tiptap/vue-3'
  import remixiconUrl from 'remixicon/fonts/remixicon.symbol.svg'
  import MenuGroup from './MenuGroup.vue'

  const imageFileInput = ref()

  const props = defineProps<{ 
    editor: Editor, 
    showUploadDialog: Function, 
    showVoiceInput: Function,
    showTextInput: Function,
    addImageByBase64: Function,
    autoTypography: Function,
  }>()

  const textHandleItems = [
    {
      icon: 'bold',
      title: '加粗',
      action: () => props.editor?.chain().focus().toggleBold().run(),
      isActive: () => props.editor?.isActive('bold')
    },
    {
      icon: 'italic',
      title: '斜体',
      action: () => props.editor?.chain().focus().toggleItalic().run(),
      isActive: () => props.editor?.isActive('italic')
    },
    {
      icon: 'strikethrough',
      title: '删除线',
      action: () => props.editor?.chain().focus().toggleStrike().run(),
      isActive: () => props.editor?.isActive('strike')
    },
    {
      icon: 'code-view',
      title: '代码',
      action: () => props.editor?.chain().focus().toggleCode().run(),
      isActive: () => props.editor?.isActive('code')
    },
    {
      icon: 'code-block',
      title: '代码块',
      action: () => props.editor?.chain().focus().toggleCodeBlock().run(),
      isActive: () => props.editor?.isActive('codeBlock')
    },
    {
      icon: 'subscript',
      title: '上标',
      action: () => props.editor?.chain().focus().toggleSubscript().run(),
      isActive: () => props.editor?.isActive('subscript')
    },
    {
      icon: 'superscript',
      title: '下标',
      action: () => props.editor?.chain().focus().toggleSuperscript().run(),
      isActive: () => props.editor?.isActive('superscript')
    },
    {
      icon: 'underline',
      title: '下划线',
      action: () => props.editor?.chain().focus().toggleUnderline().run(),
      isActive: () => props.editor?.isActive('underline')
    },
    {
      icon: 'separator',
      title: '分割线',
      action: () => props.editor?.chain().focus().setHorizontalRule().run(),
      // isActive: () => props.editor?.isActive('underline')
    },
    {
      icon: 'chat-quote-line',
      title: '引用块',
      action: () => props.editor?.chain().focus().toggleBlockquote().run(),
      isActive: () => props.editor?.isActive('blockquote')
    },
    {
      icon: 'mark-pen-line',
      title: '高亮',
      action: () => props.editor?.chain().focus().toggleHighlight().run(),
      isActive: () => props.editor?.isActive('highlight')
    },
    {
      icon: 'arrow-go-back-line',
      title: '撤销',
      action: () => props.editor?.chain().focus().undo().run()
    },
    {
      icon: 'arrow-go-forward-line',
      title: '重做',
      action: () => props.editor?.chain().focus().redo().run()
    }
  ]

  // const highlightItems = [
  //   {
  //     icon: 'mark-pen-line',
  //     title: '高亮',
  //     action: () => props.editor?.chain().focus().toggleHighlight().run(),
  //     isActive: () => props.editor?.isActive('highlight')
  //   },
  //   // {
  //   //   icon: 'external-link-line',
  //   //   title: 'ExternalLink',
  //   //   action: () => props.editor?.chain().focus().toggleCode().run(),
  //   //   isActive: () => props.editor?.isActive('code')
  //   // },
  // ]

  const headerItems = [
    {
      icon: 'h-1',
      title: '一级标题',
      action: () => props.editor?.chain().focus().toggleHeading({ level: 1 }).run(),
      isActive: () => props.editor?.isActive('heading', { level: 1 })
    },
    {
      icon: 'h-2',
      title: '二级标题',
      action: () => props.editor?.chain().focus().toggleHeading({ level: 2 }).run(),
      isActive: () => props.editor?.isActive('heading', { level: 2 })
    },
    {
      icon: 'h-3',
      title: '三级标题',
      action: () => props.editor?.chain().focus().toggleHeading({ level: 3 }).run(),
      isActive: () => props.editor?.isActive('heading', { level: 3 })
    },
    {
      icon: 'h-4',
      title: '四级标题',
      action: () => props.editor?.chain().focus().toggleHeading({ level: 4 }).run(),
      isActive: () => props.editor?.isActive('heading', { level: 4})
    },
    {
      icon: 'h-5',
      title: '五级标题',
      action: () => props.editor?.chain().focus().toggleHeading({ level: 5 }).run(),
      isActive: () => props.editor?.isActive('heading', { level: 5})
    },
    {
      icon: 'h-6',
      title: '六级标题',
      action: () => props.editor?.chain().focus().toggleHeading({ level: 6 }).run(),
      isActive: () => props.editor?.isActive('heading', { level: 6})
    },
  ]

  const commonFunctionItems = [
    {
      icon: 'list-unordered',
      title: '无序列表',
      action: () => props.editor?.chain().focus().toggleBulletList().run(),
      isActive: () => props.editor?.isActive('bulletList')
    },
    {
      icon: 'list-ordered',
      title: '有序列表',
      action: () => props.editor?.chain().focus().toggleOrderedList().run(),
      isActive: () => props.editor?.isActive('orderedList')
    },
    {
      icon: 'list-check-2',
      title: '任务列表',
      action: () => props.editor?.chain().focus().toggleTaskList().run(),
      isActive: () => props.editor?.isActive('taskList')
    },
    {
      icon: 'image-line',
      title: '图片',
      action: () => imageFileInput.value.click()
    },
    {
      icon: 'video-line',
      title: '视频',
      action: () => {
        const url = prompt('Enter YouTube URL') as string

        props.editor.commands.setYoutubeVideo({
          src: url,
        })
      }
    },
    {
      icon: 'table-3',
      title: '插入表格',
      action: () => props.editor?.chain().focus().insertTable({ rows: 3, cols: 3, withHeaderRow: true }).run()
    },
    {
      icon: 'delete-bin-2-line',
      title: '删除表格',
      action: () => props.editor?.chain().focus().deleteTable().run()
    },
    {
      icon: 'insert-column-left',
      title: '向左侧添加列',
      action: () => props.editor?.chain().focus().addColumnBefore().run()
    },
    {
      icon: 'insert-column-right',
      title: '向右侧添加列',
      action: () => props.editor?.chain().focus().addColumnAfter().run()
    },
    {
      icon: 'delete-column',
      title: '删除列',
      action: () => props.editor?.chain().focus().deleteColumn().run()
    },
    {
      icon: 'insert-row-top',
      title: '向上方添加行',
      action: () => props.editor?.chain().focus().addRowBefore().run()
    },
    {
      icon: 'insert-row-bottom',
      title: '向下方添加行',
      action: () => props.editor?.chain().focus().addRowAfter().run()
    },
    {
      icon: 'delete-row',
      title: '删除行',
      action: () => props.editor?.chain().focus().deleteRow().run()
    },
  ]

  const AIItems = [
    {
      icon: "character-recognition-line",
      title: "OCR识别",
      action: props.showUploadDialog,
      parmas: {
        url: "ocr",
      }
    },
    {
      icon: "mic-line",
      title: "语音识别",
      action: props.showVoiceInput,
      params: {
        url: "voice-recognise",
      }
    },
    {
      icon: "video-upload-line",
      title: "视频内容识别",
      action: props.showUploadDialog,
      params: {
        url: "video-recognise",
      }
    },
    {
      icon: "article-line",
      title: "撰写大创项目书",
      action: props.showTextInput,
      params: {
        url: "project-document",
        prompt: "请您简要描述您的大创项目，AI助手将会为您生成一份详细的大创项目书",
        title: "撰写大创项目书",
      }
    },
    {
      icon: "file-code-line",
      title: "代码编写",
      action: props.showTextInput,
      params: {
        url: "code-edit",
        prompt: "请您简要描述您的需求和使用的语言，AI助手将会为您生成一份详细的代码",
        title: "代码编写",
      }
    },
    {
      icon: "bar-chart-2-line",
      title: "生成柱状图",
      action: props.showTextInput,
      params: {
        url: "make-bar",
        prompt: "请您简要描述您的需求和数据，AI助手将会为您生成一张对应的柱状图",
        title: "生成柱状图",
      }
    },
    {
      icon: "mind-map",
      title: "生成思维导图",
      action: props.showTextInput,
      params: {
        url: "make-mind-map",
        prompt: "请您简要描述您的需求，AI助手将会为您生成一张对应的思维导图",
        title: "生成思维导图",
      }
    },
    {
      icon: "align-justify",
      title: "自动排版",
      action: props.autoTypography,
      params: {
        url: "auto-typography",
      }
    },
  ]

  const handleImageFile = (event: Event) => {
    const input = event.target as HTMLInputElement
    if (!input.files || input.files.length === 0) return

    const file = input.files[0]
    const allowedTypes = ['image/jpeg', 'image/png', 'image/gif']
    if (!allowedTypes.includes(file.type)) {
      alert('请选择 JPEG, PNG 或 GIF 格式的图片。')
      return
    }

    const reader = new FileReader()

    reader.onload = () => {
      const result = reader.result as string
      props.addImageByBase64(result, file.name, file.type)
    }

    if (file) {
      reader.readAsDataURL(file)
    }
  }
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

