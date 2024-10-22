<template>
  <div class="modal-overlay" @click.self="close">
    <div class="modal-container">
      <h2 class="modal-title">Make Alias</h2>
      <div class="author-sections">
        <div class="author-left">
          <div class="author-details">
            <div class="author-image-container">
              <covers-author-image :author="authorA" :default-image="defaultImage" />
            </div>
            <div class="author-info">
              <p id="author-info-name"><strong>Name:</strong> {{ authorA.name }}</p>
              <p><strong>ASIN:</strong> {{ authorA.asin }}</p>
              <p><strong>Description:</strong> {{ authorA.description }}</p>
              <p class="alias-text"><strong>Alias:</strong> {{ authorB.name }}</p>
            </div>
          </div>
          <ui-btn type="button" color="success" class="mt-5" @click="makeAlias('AtoB')">Make {{ authorB.name }} as {{ authorA.name }}'s alias</ui-btn>
        </div>

        <div class="author-right">
          <div class="author-details">
            <div class="author-info">
              <p id="author-info-name"><strong>Name:</strong> {{ authorB.name }}</p>
              <p><strong>ASIN:</strong> {{ authorB.asin }}</p>
              <p><strong>Description:</strong> {{ authorB.description }}</p>
              <p class="alias-text"><strong>Alias:</strong> {{ authorA.name }}</p>
            </div>
            <div class="author-image-container">
              <covers-author-image :author="authorB" :default-image="defaultImage" />
            </div>
          </div>
          <ui-btn type="button" color="success" class="mt-5" @click="makeAlias('BtoA')">Make {{ authorA.name }} as {{ authorB.name }}'s alias</ui-btn>
        </div>
      </div>

      <div class="modal-actions">
        <ui-btn type="button" @click="close">Cancel</ui-btn>
      </div>
    </div>
  </div>
</template>

<script>
import CoversAuthorImage from '@/components/covers/AuthorImage.vue'

export default {
  components: {
    CoversAuthorImage
  },
  props: {
    authorA: {
      type: Object,
      required: true
    },
    authorB: {
      type: Object,
      required: true
    },
    notificationId: {
      type: String,
      required: false
    },
    shouldClearNotification: {
      type: Boolean,
      required: false,
      default: false
    }
  },
  methods: {
    async makeAlias(direction) {
      const originalAuthor = direction === 'AtoB' ? this.authorA : this.authorB
      const alias = direction === 'AtoB' ? this.authorB : this.authorA

      try {
        const response = await this.$axios.$post(`/api/authors/${originalAuthor.id}/make_alias`, { aliasId: alias.id })
        this.$toast.success(`Success`)
        if (this.shouldClearNotification) {
          const notificationId = this.notificationId
          await this.$axios.get('/api/clearNotifications', {
            params: { notificationId }
          })
        }

        this.$emit('alias', this.notificationId)
      } catch (error) {
        if (error.response) {
          if (error.response.status === 409) {
            const data = error.response.data
            this.$toast.error('Failed: ' + data)
          }
        } else {
          console.error('Error making alias:', error)
          this.$toast.error('Failed to make alias')
        }
      }
      this.close()
    },
    close() {
      this.isMakeAliasModalVisible = false
      this.$emit('close')
    }
  }
}
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-container {
  background-color: #1a1a1a;
  padding: 20px;
  border-radius: 10px;
  width: 70%;
  max-width: 1000px;
  max-height: 80vh;
  display: flex;
  flex-direction: column;
  overflow: auto;
}

.modal-title {
  text-align: center;
  font-size: 24px;
  margin-bottom: 20px;
}

.author-sections {
  display: grid;
  grid-template-columns: 1fr 1fr;
  grid-gap: 20px;
  padding: 20px;
  align-items: center;
}

.author-details {
  display: flex;
  justify-content: space-around;
  align-content: stretch;
}

.author-left,
.author-right {
  background-color: rgba(207, 207, 207, 0.1);
  padding: 20px;
  border-radius: 8px;
  margin-bottom: 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.author-info,
.author-image-container {
  padding: 20px;
}

.author-info {
  max-width: 200px;
}

.modal-actions {
  text-align: center;
  margin-top: auto;
  padding-top: 10px;
  border-top: 1px solid #333;
  position: relative;
}

.author-image-container {
  background-size: cover;
  background-position: center;
  width: 150px;
  height: 180px;
  border-radius: 10px;
  overflow: hidden;
}

.alias-text {
  font-size: 19px;
  color: #ede6cd;
}
</style>
