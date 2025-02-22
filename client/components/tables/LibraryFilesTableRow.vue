<template>
  <tr>
    <td class="px-4">
      {{ showFullPath ? file.metadata.path : file.metadata.relPath }}
    </td>
    <td>
      {{ $bytesPretty(file.metadata.size) }}
    </td>
    <td class="text-xs">
      <div class="flex items-center">
        <p>{{ file.fileType }}</p>
      </div>
    </td>
    <td v-if="contextMenuItems.length" class="text-center">
      <ui-context-menu-dropdown :items="contextMenuItems" menu-width="110px" @action="contextMenuAction" />
    </td>
  </tr>
</template>

<script>
export default {
  props: {
    libraryItemId: String,
    showFullPath: Boolean,
    file: {
      type: Object,
      default: () => {}
    },
    inModal: Boolean
  },
  data() {
    return {}
  },
  computed: {
    userToken() {
      return this.$store.getters['user/getToken']
    },
    userCanDownload() {
      return this.$store.getters['user/getUserCanDownload']
    },
    userCanDelete() {
      return this.$store.getters['user/getUserCanDelete']
    },
    userIsAdmin() {
      return this.$store.getters['user/getIsAdminOrUp']
    },
    downloadUrl() {
      return `${process.env.serverUrl}/s/item/${this.libraryItemId}/${this.$encodeUriPath(this.file.metadata.relPath).replace(/^\//, '')}?token=${this.userToken}`
    },
    contextMenuItems() {
      const items = []
      if (this.userCanDownload) {
        items.push({
          text: this.$strings.LabelDownload,
          action: 'download'
        })
      }
      if (this.userCanDelete) {
        items.push({
          text: this.$strings.ButtonDelete,
          action: 'delete'
        })
      }
      // Currently not showing this option in the Files tab modal
      if (this.userIsAdmin && this.file.audioFile && !this.inModal) {
        items.push({
          text: this.$strings.LabelMoreInfo,
          action: 'more'
        })
      }
      return items
    }
  },
  methods: {
    contextMenuAction(action) {
      if (action === 'delete') {
        this.deleteLibraryFile()
      } else if (action === 'download') {
        this.downloadLibraryFile()
      } else if (action === 'more') {
        this.$emit('showMore', this.file.audioFile)
      }
    },
    deleteLibraryFile() {
      const payload = {
        message: 'This will delete the file from your file system. Are you sure?',
        callback: (confirmed) => {
          if (confirmed) {
            this.$axios
              .$delete(`/api/items/${this.libraryItemId}/file/${this.file.ino}`)
              .then(() => {
                this.$toast.success('File deleted')
              })
              .catch((error) => {
                console.error('Failed to delete file', error)
                this.$toast.error('Failed to delete file')
              })
          }
        },
        type: 'yesNo'
      }
      this.$store.commit('globals/setConfirmPrompt', payload)
    },
    downloadLibraryFile() {
      const a = document.createElement('a')
      a.style.display = 'none'
      a.href = this.downloadUrl
      a.download = this.file.metadata.filename
      document.body.appendChild(a)
      a.click()
      setTimeout(() => {
        a.remove()
      })
    }
  },
  mounted() {}
}
</script>