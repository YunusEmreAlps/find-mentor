<template>
  <div class="container">
    <h1>Mentees</h1>
    <input v-model="search.keyword" class="filter" placeholder="Search in Mentees" @input="searchMentee">
    <input v-model="search2.keyword" class="filter" placeholder="Search in Topics" @input="searchTopic">
    <ul class="persons">
      <h5 v-if="postList.mentee.items.length <= 0">
        No results...
      </h5>
      <Card
        v-for="(mentee, index) in postList.mentee.items"
        v-else
        :key="index"
        class="person"
        :person="mentee"
        person-type="mentee"
      />
    </ul>
    <client-only>
      <infinite-loading v-if="postList.mentee.items.length >= postList.mentee.limit && !search.isFilled" @infinite="loadMoreMentees" />
    </client-only>
  </div>
</template>

<script>
export default {
  async fetch () {
    this.postList.mentee.items = await this.$content('persons').where({ mentor: { $in: ['Mentee', 'İkisi de'] } })
      .limit(this.postList.mentee.limit)
      .skip(this.postList.mentee.skip)
      .fetch()
  },
  data () {
    return {
      search: {
        keyword: null,
        isFilled: false
      },
      search2: {
        keyword: null,
        isFilled: false
      },
      postList: {
        mentee: {
          items: [],
          limit: 16,
          skip: 0
        }
      }
    }
  },
  methods: {
    async loadMoreMentees ($state) {
      this.postList.mentee.skip += this.postList.mentee.limit

      const mentees = await this.$content('persons')
        .where({ mentor: { $in: ['Mentee', 'İkisi de'] } })
        .limit(this.postList.mentee.limit)
        .skip(this.postList.mentee.skip)
        .fetch()

      this.postList.mentee.items.push(...mentees)
      $state.loaded()

      if (mentees.length <= 0) {
        $state.complete()
      }
    },
    async searchMentee () {
      this.search2.keyword = ''
      const result = await this.$content('persons')
        .where({ mentor: { $in: ['Mentee', 'İkisi de'] } })
        .search(this.search.keyword)
        .fetch()

      if (this.search.keyword.length > 0) {
        this.search.isFilled = true

        this.postList.mentee.items = result
      } else {
        this.postList.mentee.skip = 0
        this.$fetch()
        this.search.isFilled = false
      }
    },
    async searchTopic () {
      this.search.keyword = ''
      const result = await this.$content('persons')
        .where({ interests: { $contains: this.search2.keyword }, mentor: { $in: ['Mentee', 'İkisi de'] } })
        .fetch()

      if (this.search2.keyword.length > 0) {
        this.search2.isFilled = true

        this.postList.mentee.items = result
      } else {
        this.postList.mentee.skip = 0
        this.$fetch()
        this.search2.isFilled = false
      }
    }
  }
}
</script>
