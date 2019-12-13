<template>
  <div>
    <folders />
    <files />
    <transition name="slide">
      <fdetails v-if="show" class="detailsContainer" />
    </transition>
    <div :class="{ overlay: hasOverlay }" @click="closeDetails()"></div>
  </div>
</template>

<script>
import folders from '@/components/Folders'
import files from '@/components/Files'
import fdetails from '@/components/Details'

export default {
  components: {
    folders,
    files,
    fdetails
  },
  data () {
    return {
      curFolderName: '',
      curFolderId: '',
      show: false,
      hasOverlay: false
    }
  },
  computed: {
  },
  methods: {
    closeDetails () {
      this.show = false
      this.hasOverlay = false
    }
  },
  mounted () {
    this.$root.$on('showFileDetailsPane', (obj) => {
      this.show = true
      this.hasOverlay = true
    })

    this.$root.$on('closeDetailsPane', () => {
      this.closeDetails()
    })
  }
}
</script>

<style lang="scss" scoped>
.overlay {
  z-index: 998;
  position: fixed;
  height: 100%;
  width: 100%;
  top: 0;
  left: 0;
  background: rgba(0, 0, 0, 0.6);
}

.fab-main-container {
  right: 5% !important;
  z-index: 2000 !important;
}

.detailsContainer {
  position: fixed;
  height: 100vh;
  width: 30%;
  top: 0;
  right: 0;
  background: #f2f2f2;
  z-index: 999;
  border-left: 1px solid #ddd;
}
</style>
