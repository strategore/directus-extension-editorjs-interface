<template>
  <div>
    <v-dialog :active="fileDialogState" @toggle="closeFileDialog" @esc="closeFileDialog">
      <v-card>
        <v-card-title>
          {{ $t('interfaces.file.description') }}
        </v-card-title>
        <v-card-text>
          <v-upload ref="vUploaderComponentRef" @input="fileHandler" :multiple="false" from-library from-url />
        </v-card-text>
        <v-card-actions>
          <v-button secondary @click="closeFileDialog">
            {{ $t('Cancel') }}
          </v-button>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <div :class="className" ref="editorElement"></div>
  </div>
</template>

<script>
import EditorJS from '@editorjs/editorjs'
import SimpleImageTool from '@editorjs/simple-image'
import ParagraphTool from '@editorjs/paragraph'
import QuoteTool from '@editorjs/quote'
import WarningTool from '@editorjs/warning'
import ChecklistTool from '@editorjs/checklist'
import DelimiterTool from '@editorjs/delimiter'
import TableTool from '@editorjs/table'
import CodeTool from '@editorjs/code'
import HeaderTool from '@editorjs/header'
import UnderlineTool from '@editorjs/underline'
import ListTool from 'editorjs-list'
import EmbedTool from '@editorjs/embed'
import MarkerTool from '@editorjs/marker'
import RawToolTool from '@editorjs/raw'
import InlineCodeTool from '@editorjs/inline-code'
import TextAlignTool from '@canburaks/text-align-editorjs'
import AlertTool from 'editorjs-alert'
import StrikethroughTool from '@itech-indrustries/editorjs-strikethrough'
import ImageTool from './custom-blocks/plugin-image-patch'
import AttachesTool from './custom-blocks/plugin-attaches-patch'
import PersonalityTool from './custom-blocks/plugin-personality-patch'

export default {
  props: {
    value: {
      type: Object,
      default: null,
    },
    disabled: {
      type: Boolean,
      default: false,
    },
    placeholder: {
      type: String,
      default: null,
    },
    tools: {
      type: Array,
      default: () => [
        'header',
        'list',
        'code',
        'image',
        'paragraph',
        'delimeter',
        'checklist',
        'quote',
        'underline',
      ],
    },
    toolsConfigOverride: {
      type: Object,
      default: null,
    },
    font: {
      type: String,
      default: 'sans-serif',
    },
  },

  mounted: function () {
    this.editorInstance = new EditorJS({
      logLevel: 'ERROR',
      holder: this.$refs.editorElement,
      data: this.getPreparedValue(this.$props.value),
      readOnly: this.$props.disabled,
      placeholder: this.$props.placeholder,
      tools: this.buildToolsOptions(),
      minHeight: 24,
      onChange: this.editorValueEmitter,
    })
  },

  unmounted: function () {
    if (this.editorInstance) {
      this.editorInstance.destroy()
    }
  },

  data: function() {
    return {
      fileDialogState: false,
      className: {
        [this.$props.font]: true
      }
    }
  },

  watch: {
    value: function (newVal, oldVal) {
      // @TODO make value property change handling

      if (oldVal || !this.editorInstance) {
        // If already have value, then skip,
        // this is because render method always touch time property.
        return
      }

      this.editorInstance.isReady.then(() => {
        this.editorInstance.render(this.getPreparedValue(newVal))
      })
    },
  },

  methods: {

    closeFileDialog: function() {
      this.fileDialogState = false
      this._fileHandler = null
    },

    openFileDialog: function(handler) {
      this.fileDialogState = true
      this._fileHandler = handler
    },

    fileHandler: function(event) {
      this._fileHandler(event)
      this.closeFileDialog()
    },

    getUploadFieldRef: function() {
      return this.$refs.vUploaderComponentRef
    },

    getPreparedValue: function (value) {
      return {
        time: value?.time,
        version: value?.version,
        blocks: value?.blocks || [],
      }
    },

    editorValueEmitter: async function (context) {
      this.$emit('input', await context.saver.save())
    },

    buildToolsOptions: function () {
      const defaults = {
        header: {
          class: HeaderTool,
          shortcut: 'CMD+SHIFT+H',
          inlineToolbar: true,
        },
        list: {
          class: ListTool,
          inlineToolbar: true,
          shortcut: 'CMD+SHIFT+O',
        },
        embed: {
          class: EmbedTool,
          inlineToolbar: true,
        },
        paragraph: {
          class: ParagraphTool,
          inlineToolbar: true,
        },
        code: {
          class: CodeTool,
        },
        warning: {
          class: WarningTool,
          inlineToolbar: true,
          shortcut: 'CMD+SHIFT+W',
        },
        underline: {
          class: UnderlineTool,
          shortcut: 'CMD+SHIFT+U',
        },
        textalign: {
          class: TextAlignTool,
          inlineToolbar: true,
          shortcut: 'CMD+SHIFT+A',
        },
        strikethrough: {
          class: StrikethroughTool,
        },
        alert: {
          class: AlertTool,
        },
        table: {
          class: TableTool,
          inlineToolbar: true,
        },
        quote: {
          class: QuoteTool,
          inlineToolbar: true,
          shortcut: 'CMD+SHIFT+O',
        },
        marker: {
          class: MarkerTool,
          shortcut: 'CMD+SHIFT+M',
        },
        inlinecode: {
          class: InlineCodeTool,
          shortcut: 'CMD+SHIFT+I',
        },
        delimiter: {
          class: DelimiterTool,
        },
        raw: {
          class: RawToolTool,
        },
        checklist: {
          class: ChecklistTool,
          inlineToolbar: true,
        },
        simpleimage: {
          class: SimpleImageTool,
        },
        image: {
          class: ImageTool,
          config: {
            uploader: {
              getUploadFieldRef: this.getUploadFieldRef,
              picker: this.openFileDialog,
            },
          },
        },
        attaches: {
          class: AttachesTool,
          config: {
            uploader: {
              getUploadFieldRef: this.getUploadFieldRef,
              picker: this.openFileDialog,
            },
          },
        },
        personality: {
          class: PersonalityTool,
          config: {
            uploader: {
              getUploadFieldRef: this.getUploadFieldRef,
              picker: this.openFileDialog,
            },
          },
        },
      }

      // Build current tools config.
      const tools = {}
      for (const toolName of this.$props.tools) {
        if (toolName in defaults) {
          tools[toolName.toString()] = Object.assign(
              {},
              defaults[toolName],
              this.$props.toolsConfigOverride && this.$props.toolsConfigOverride?.[toolName],
          )
        }
      }

      return tools
    },
  },
}
</script>

<style lang="css" scoped>
.monospace {
  --v-input-font-family: var(--family-monospace);
}

.serif {
  --v-input-font-family: var(--family-serif);
}

.sans-serif {
  --v-input-font-family: var(--family-sans-serif);
}
</style>

<style src="./editorjs-content-reset.css"></style>
<style src="./editorjs-components.css"></style>