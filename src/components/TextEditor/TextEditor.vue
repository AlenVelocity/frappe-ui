<template>
  <div
    class="relative w-full table-editor"
    :class="$attrs.class"
    :style="$attrs.style"
    v-if="editor"
    ref="editorContainer"
  >
    <TextEditorBubbleMenu :buttons="bubbleMenu" :options="bubbleMenuOptions" />
    <TextEditorFixedMenu
      class="w-full overflow-x-auto rounded-t-lg border border-outline-gray-modals"
      :buttons="fixedMenu"
    />
    <TextEditorFloatingMenu :buttons="floatingMenu" />
    <slot name="top" />
    <slot name="editor" :editor="editor">
      <editor-content :editor="editor" />
    </slot>
    <slot name="bottom" />
  </div>
</template>

<script lang="ts">
import { normalizeClass, computed } from 'vue'
import { Editor, EditorContent, VueNodeViewRenderer } from '@tiptap/vue-3'
import StarterKit from '@tiptap/starter-kit'
import Placeholder from '@tiptap/extension-placeholder'
import TextAlign from '@tiptap/extension-text-align'
import { ImageExtension } from './extensions/image'
import ImageViewerExtension from './image-viewer-extension'
import VideoExtension from './video-extension'
import LinkExtension from './link-extension'
import Typography from '@tiptap/extension-typography'
import TextStyle from '@tiptap/extension-text-style'
import NamedColorExtension from './extensions/color'
import NamedHighlightExtension from './extensions/highlight'
import { common, createLowlight } from 'lowlight'
import CodeBlockLowlight from '@tiptap/extension-code-block-lowlight'
import CodeBlockComponent from './CodeBlockComponent.vue'
import configureMention from './mention'
import TextEditorFixedMenu from './TextEditorFixedMenu.vue'
import TextEditorBubbleMenu from './TextEditorBubbleMenu.vue'
import TextEditorFloatingMenu from './TextEditorFloatingMenu.vue'
import EmojiExtension from './extensions/emoji/emoji-extension'
import SlashCommands from './extensions/slash-commands/slash-commands-extension'
import { detectMarkdown, markdownToHTML } from '../../utils/markdown'
import { DOMParser } from 'prosemirror-model'
import { TagNode, TagExtension } from './extensions/tag/tag-extension'
import { Heading } from './extensions/heading/heading'
import { TableExtension } from './extensions/table'

const lowlight = createLowlight(common)

export default {
  name: 'TextEditor',
  inheritAttrs: false,
  components: {
    EditorContent,
    TextEditorFixedMenu,
    TextEditorBubbleMenu,
    TextEditorFloatingMenu,
  },
  props: {
    content: {
      type: String,
      default: null,
    },
    placeholder: {
      type: [String, Function],
      default: '',
    },
    editorClass: {
      type: [String, Array, Object],
      default: '',
    },
    editable: {
      type: Boolean,
      default: true,
    },
    bubbleMenu: {
      type: [Boolean, Array],
      default: false,
    },
    bubbleMenuOptions: {
      type: Object,
      default: () => ({}),
    },
    fixedMenu: {
      type: [Boolean, Array],
      default: false,
    },
    floatingMenu: {
      type: [Boolean, Array],
      default: false,
    },
    extensions: {
      type: Array,
      default: () => [],
    },
    starterkitOptions: {
      type: Object,
      default: () => ({}),
    },
    mentions: {
      type: Array,
      default: () => [],
    },
    tags: {
      type: Array,
      default: () => [],
    },
    uploadFunction: {
      type: Function,
      default: () => null,
    },
  },
  emits: ['change', 'focus', 'blur'],
  expose: ['editor'],
  provide() {
    return {
      editor: computed(() => this.editor),
    }
  },
  data() {
    return {
      editor: null,
    }
  },
  watch: {
    content(val) {
      let currentHTML = this.editor.getHTML()
      if (currentHTML !== val) {
        this.editor.commands.setContent(val)
      }
    },
    editable(value) {
      this.editor.setEditable(value)
    },
    editorProps: {
      deep: true,
      handler(value) {
        if (this.editor) {
          this.editor.setOptions({
            editorProps: value,
          })
        }
      },
    },
  },
  mounted() {
    this.editor = new Editor({
      content: this.content || null,
      editorProps: this.editorProps,
      editable: this.editable,
      extensions: [
        StarterKit.configure({
          ...this.starterkitOptions,
          codeBlock: false,
          heading: false,
        }),
        Heading.configure({
          ...(typeof this.starterkitOptions?.heading === 'object' &&
          this.starterkitOptions.heading !== null
            ? this.starterkitOptions.heading
            : {}),
        }),
        TableExtension.configure({
          showActionHandles: this.editable,
        }),
        Typography,
        TextAlign.configure({
          types: ['heading', 'paragraph'],
        }),
        TextStyle,
        NamedColorExtension,
        NamedHighlightExtension,
        CodeBlockLowlight.extend({
          addNodeView() {
            return VueNodeViewRenderer(CodeBlockComponent)
          },
        }).configure({ lowlight }),
        ImageExtension.configure({
          uploadFunction: this.uploadFunction,
        }),
        ImageViewerExtension,
        VideoExtension.configure({
          uploadFunction: this.uploadFunction,
        }),
        LinkExtension.configure({
          openOnClick: false,
        }),
        Placeholder.configure({
          placeholder:
            typeof this.placeholder === 'function'
              ? this.placeholder
              : () => this.placeholder,
        }),
        configureMention(this.mentions),
        EmojiExtension,
        SlashCommands,
        TagNode,
        TagExtension.configure({
          tags: () => this.tags,
        }),
        ...(this.extensions || []),
      ],
      onUpdate: ({ editor }) => {
        this.$emit('change', editor.getHTML())
      },
      onFocus: ({ editor, event }) => {
        this.$emit('focus', event)
      },
      onBlur: ({ editor, event }) => {
        this.$emit('blur', event)
      },
    })
  },
  beforeUnmount() {
    this.editor.destroy()
    this.editor = null
  },
  computed: {
    editorProps() {
      return {
        attributes: {
          class: normalizeClass([
            'prose prose-table:table-fixed prose-td:p-2 prose-th:p-2 prose-td:border prose-th:border prose-td:border-outline-gray-2 prose-th:border-outline-gray-2 prose-td:relative prose-th:relative prose-th:bg-surface-gray-2',
            this.editorClass,
          ]),
        },
        clipboardTextParser: (text, $context) => {
          if (!detectMarkdown(text)) return
          if (
            !confirm(
              'Do you want to convert markdown content to HTML before pasting?',
            )
          )
            return

          let dom = document.createElement('div')
          dom.innerHTML = markdownToHTML(text)
          let parser =
            this.editor.view.someProp('clipboardParser') ||
            this.editor.view.someProp('domParser') ||
            DOMParser.fromSchema(this.editor.schema)
          return parser.parseSlice(dom, {
            preserveWhitespace: true,
            context: $context,
          })
        },
      }
    },
  },
}
</script>
<style>
@import './extensions/color/color-styles.css';
@import './extensions/highlight/highlight-styles.css';
@import './extensions/table/table-styles.css';

.ProseMirror {
  outline: none;
  caret-color: var(--ink-gray-9);
  word-break: break-word;
}

/* Firefox */
.ProseMirror-focused:focus-visible {
  outline: none;
}

/* Placeholder */
.ProseMirror:not(.ProseMirror-focused) p.is-editor-empty::before {
  content: attr(data-placeholder);
  float: left;
  color: var(--ink-gray-4);
  pointer-events: none;
  height: 0;
}

.ProseMirror-selectednode video,
img.ProseMirror-selectednode {
  outline: 2px solid var(--outline-gray-2);
}

/* Mentions */
.mention {
  font-weight: 600;
  box-decoration-break: clone;
}

/* Table styles */
.prose table p {
  margin: 0;
}

.resize-cursor {
  cursor: ew-resize;
  cursor: col-resize;
}

.tag-item,
.tag-suggestion-active {
  background-color: var(--surface-gray-1, #f8f8f8);
  color: inherit;
  border: 1px solid transparent;
  padding: 0px 2px;
  border-radius: 4px;
  font-size: 1em;
  white-space: nowrap;
  cursor: default;
}

.tag-item.ProseMirror-selectednode {
  border-color: var(--outline-gray-3, #c7c7c7);
}

.tag-suggestion-active {
  background-color: var(--surface-gray-2, #f3f3f3);
}

</style>
