<template>
  <div class="w-full h-full px-1 md:px-4 py-1 mb-4">
    <div class="flex items-center py-2">
      <ui-toggle-switch v-model="useSquareBookCovers" @input="formUpdated" />
      <ui-tooltip :text="$strings.LabelSettingsSquareBookCoversHelp">
        <p class="pl-4 text-base">
          {{ $strings.LabelSettingsSquareBookCovers }}
          <span class="material-icons icon-text text-sm">info_outlined</span>
        </p>
      </ui-tooltip>
    </div>
    <div class="py-3">
      <div class="flex items-center">
        <ui-toggle-switch v-if="!globalWatcherDisabled" v-model="disableWatcher" @input="formUpdated" />
        <ui-toggle-switch v-else disabled :value="false" />
        <p class="pl-4 text-base">{{ $strings.LabelSettingsDisableWatcherForLibrary }}</p>
      </div>
      <p v-if="globalWatcherDisabled" class="text-xs text-warning">*{{ $strings.MessageWatcherIsDisabledGlobally }}</p>
    </div>
    <div v-if="mediaType == 'book'" class="py-3">
      <div class="flex items-center">
        <ui-toggle-switch v-model="skipMatchingMediaWithAsin" @input="formUpdated" />
        <p class="pl-4 text-base">{{ $strings.LabelSettingsSkipMatchingBooksWithASIN }}</p>
      </div>
    </div>
    <div v-if="mediaType == 'book'" class="py-3">
      <div class="flex items-center">
        <ui-toggle-switch v-model="skipMatchingMediaWithIsbn" @input="formUpdated" />
        <p class="pl-4 text-base">{{ $strings.LabelSettingsSkipMatchingBooksWithISBN }}</p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    library: {
      type: Object,
      default: () => null
    },
    processing: Boolean
  },
  data() {
    return {
      provider: null,
      useSquareBookCovers: false,
      disableWatcher: false,
      skipMatchingMediaWithAsin: false,
      skipMatchingMediaWithIsbn: false
    }
  },
  computed: {
    librarySettings() {
      return this.library.settings || {}
    },
    globalWatcherDisabled() {
      return this.$store.getters['getServerSetting']('scannerDisableWatcher')
    },
    mediaType() {
      return this.library.mediaType
    },
    providers() {
      if (this.mediaType === 'podcast') return this.$store.state.scanners.podcastProviders
      return this.$store.state.scanners.providers
    }
  },
  methods: {
    getLibraryData() {
      return {
        settings: {
          coverAspectRatio: this.useSquareBookCovers ? this.$constants.BookCoverAspectRatio.SQUARE : this.$constants.BookCoverAspectRatio.STANDARD,
          disableWatcher: !!this.disableWatcher,
          skipMatchingMediaWithAsin: !!this.skipMatchingMediaWithAsin,
          skipMatchingMediaWithIsbn: !!this.skipMatchingMediaWithIsbn
        }
      }
    },
    formUpdated() {
      this.$emit('update', this.getLibraryData())
    },
    init() {
      this.useSquareBookCovers = this.librarySettings.coverAspectRatio === this.$constants.BookCoverAspectRatio.SQUARE
      this.disableWatcher = !!this.librarySettings.disableWatcher
      this.skipMatchingMediaWithAsin = !!this.librarySettings.skipMatchingMediaWithAsin
      this.skipMatchingMediaWithIsbn = !!this.librarySettings.skipMatchingMediaWithIsbn
    }
  },
  mounted() {
    this.init()
  }
}
</script>