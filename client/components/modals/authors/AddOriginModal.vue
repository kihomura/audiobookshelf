<template>
  <div v-if="value" class="modal" style="padding: 20px; border: 1px solid black; width: 700px; margin: auto">
    <div v-if="author">Add original author for {{ author.name }}</div>
    <div class="p-2">
      <ui-multi-select-query-input ref="authorsSelect" v-model="authorCopy.authors" :label="$strings.LabelAuthors" filter-key="authors" />
    </div>

    <div class="flex justify-end pt-2 px-2">
      <ui-btn type="button" @click="cancel" class="cancel-btn mr-2">Cancel</ui-btn>
      <ui-btn type="submit" @click="confirm" class="confirm-btn">Confirm</ui-btn>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    value: {
      type: Boolean,
      required: true
    },
    author: {
      type: Object,
      required: true
    }
  },
  computed: {
    filterData() {
      return this.$store.state.libraries.filterData || {}
    },
    authors() {
      return this.filterData.authors || []
    }
  },
  watch: {
    author: {
      immediate: true,
      handler(newVal) {
        if (newVal) {
          this.authorCopy = {
            ...newVal,
            authors: []
          }
          if (newVal.originalAuthor) {
            this.authorCopy.authors.push(...newVal.originalAuthor)
          }
        }
      }
    }
  },
  data() {
    return {
      newAuthor: '',
      filteredAuthors: [],
      authorCopy: {
        name: '',
        authors: []
      }
    }
  },
  methods: {
    cancel() {
      this.$emit('input', false)
    },
    async confirm() {
      try {
        const selectedAuthors = this.authorCopy.authors
        const newAuthors = selectedAuthors.filter((author) => !author.id || author.id.startsWith('new'))

        let authorIds = selectedAuthors.filter((author) => author.id && !author.id.startsWith('new')).map((author) => author.id)

        if (newAuthors.length) {
          const mediaPayload = {
            metadata: {
              authors: selectedAuthors.map((author) => ({
                name: author.name,
                id: author.id || null
              }))
            }
          }

          const createdAuthors = await this.$axios.$post(`/api/authors/${this.author.id}/createNew`, mediaPayload)
          const createdAuthorIds = createdAuthors.map((author) => author.id)
          authorIds = [...authorIds.filter((id) => !!id), ...createdAuthorIds]
        }

        console.log('Final Author IDs:', authorIds)
        await this.$axios.post(`/api/authors/${this.author.id}/combined_alias`, {
          originalAuthors: authorIds
        })

        const updateOrginalAuthor = await this.$axios.$get(`/api/authors/${this.author.id}/origins`)
        this.$emit('update-authors', updateOrginalAuthor)
        this.$toast.success(`Successfully added original author to ${this.author.name}`)
      } catch (error) {
        if (error.response) {
          if (error.response.status === 409) {
            const data = error.response.data
            this.$toast.error('Failed: ' + data.message)
          }
        } else {
          console.error('Error combining authors:', error)
          this.$toast.error('Failed to combine authors')
        }
      }

      this.$emit('input', false)
    }
  },
  mounted() {}
}
</script>

<style scoped>
.modal {
  z-index: 9999;
  position: fixed;
  top: 50%;
  left: 50%;
  background-color: rgba(14, 17, 21, 0.895);
  transform: translate(-50%, -50%);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  padding: 20px;
}
</style>
