<template>
  <div>
    <figure>
      <template v-if="attrs.src">
        <div
          ref="element"
          :style="style"
          :data-responsive="attrs.ratio"
          class="k-editor-image-block-wrapper"
          tabindex="0"
          @keydown.delete="$emit('remove')"
          @keydown.enter="$emit('append')"
          @keydown.up.prevent="$emit('prev')"
          @keydown.down.prevent="$emit('next')"
        >
          <img ref="image" :src="attrs.src" :key="attrs.src" @dblclick="selectFile" @load="onLoad">
        </div>
        <figcaption>
          <k-editable
            :content="attrs.caption"
            :breaks="true"
            :placeholder="$t('editor.blocks.image.caption.placeholder') + '…'"
            @input="caption"
          />
        </figcaption>
      </template>
      <template v-else>
        <div class="k-editor-image-block-placeholder" ref="element" tabindex="0">
          <k-button icon="upload" @click="uploadFile">{{ $t('editor.blocks.image.upload') }}</k-button>
          {{ $t('editor.blocks.image.or') }}
          <k-button icon="image" @click="selectFile">{{ $t('editor.blocks.image.select') }}</k-button>
        </div>
      </template>
    </figure>

    <k-files-dialog ref="fileDialog" @submit="insertFile($event)" />
    <k-upload ref="fileUpload" @success="insertUpload" />

    <k-dialog ref="settings" @submit="saveSettings" size="medium">
      <k-form :fields="fields" v-model="attrs" @submit="saveSettings" />
    </k-dialog>

  </div>
</template>

<script>
export default {
  icon: "image",
  props: {
    endpoints: Object,
    attrs: {
      type: Object,
      default() {
        return {};
      }
    }
  },
  computed: {
    style() {
      if (this.attrs.ratio) {
        return 'padding-bottom:' + 100 / this.attrs.ratio + '%';
      }
    },
    fields() {
      return {
        alt: {
          label: this.$t('editor.blocks.image.alt.label'),
          type: "text",
          icon: "text"
        },
        link: {
          label: this.$t('editor.blocks.image.link.label'),
          type: "text",
          icon: "url",
          placeholder: this.$t('editor.blocks.image.link.placeholder')
        },
        css: {
          label: this.$t('editor.blocks.image.css.label'),
          type: "text",
          icon: "code",
        }
      };
    }
  },
  methods: {
    caption(html) {
      this.input({
        caption: html
      });
    },
    edit() {
      if (this.attrs.guid) {
        this.$router.push(this.attrs.guid);
      }
    },
    focus() {
      this.$refs.element.focus();
    },
    input(data) {
      this.$emit("input", {
        attrs: {
          ...this.attrs,
          ...data
        }
      });
    },
    fetchFile(link) {
      this.$api.get(link).then(response => {
        this.input({
          guid: response.link,
          src: response.url,
          id: response.id,
          ratio: response.dimensions.ratio
        });
      });
    },
    insertFile(files) {
      const file = files[0];
      this.fetchFile(file.link);
    },
    insertUpload(files, response) {
      this.fetchFile(response[0].link);
    },
    menu() {

      if (this.attrs.src) {
        return [
          {
            icon: "open",
            label: this.$t("editor.blocks.image.open.browser"),
            click: this.open
          },
          {
            icon: "edit",
            label: this.$t("editor.blocks.image.open.panel"),
            click: this.edit,
            disabled: !this.attrs.guid
          },
          {
            icon: "cog",
            label: this.$t("editor.blocks.image.settings"),
            click: this.$refs.settings.open
          },
          {
            icon: "image",
            label: this.$t("editor.blocks.image.replace"),
            click: this.replace
          },
        ];
      } else {
        return [];
      }

    },
    open() {
      window.open(this.attrs.src);
    },
    onLoad() {
      const image = this.$refs.image;

      if (!this.attrs.ratio && image && image.width && image.height) {
        this.input({
          ratio: image.width / image.height
        });
      }
    },
    replace() {
      this.$emit("input", {
        attrs: {}
      });
    },
    selectFile() {
      this.$refs.fileDialog.open({
        endpoint: this.endpoints.field + "/files",
        multiple: false,
        selected: [this.attrs.id]
      });
    },
    settings() {
      this.$refs.settings.open();
    },
    saveSettings() {
      this.$refs.settings.close();
      this.input(this.attrs);
    },
    uploadFile() {
      this.$refs.fileUpload.open({
        url: window.panel.api + "/" + this.endpoints.field + "/upload",
        multiple: false,
        accept: "image/*"
      });
    },
  }
};
</script>

<style lang="scss">
.k-editor-image-block {
  margin: 1.5rem 0;
}
.k-editor-image-block figure {
  line-height: 0;
}
.k-editor-image-block-wrapper img {
  width: 100%;
}
.k-editor-image-block-wrapper[data-responsive] img {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  object-fit: contain;
  height: 100%;
}
.k-editor-image-block-wrapper[data-responsive] {
  position: relative;
  padding-bottom: 66.66%;
  background: #2d2e36;
}
.k-editor-image-block-wrapper:focus {
  outline: 0;
}
.k-editor-image-block-wrapper:focus img {
  outline: 2px solid rgba(#4271ae, 0.25);
  outline-offset: 2px;
}
.k-editor-image-block figcaption {
  display: block;
  margin-top: .75rem;
}
.k-editor-image-block .k-editable-placeholder,
.k-editor-image-block .ProseMirror {
  text-align: center;
  font-size: .875rem;
  line-height: 1.5em;
}
.k-editor-image-block-placeholder {
  display: flex;
  line-height: 1;
  justify-content: center;
  align-items: center;
  font-style: italic;
  font-size: .875rem;
  width: 100%;
  border: 1px dashed #ddd;
  border-radius: 3px;
  text-align: center;
  color: #bbb;
}
.k-editor-image-block-placeholder:focus {
  outline: 0;
}
.k-editor-image-block-placeholder .k-button {
  padding: .75rem;
  display: flex;
  align-items: center;
  color: #000;
  margin: 0 .5rem;
}
</style>
