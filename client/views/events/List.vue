<template>
  <div class="tile is-ancestor">
    <div class="tile is-parent is-vertical">
      <article class="tile is-child">
        <h4 class="title">Events list</h4>
        <div class="field">
          <label class="label">Search by name or description</label>
          <div class="field has-addons">
            <div class="control is-expanded">
              <input class="input" type="text" v-model="query" placeholder="Search by name or description" @input="refetch()" />
            </div>
            <div class="control">
              <a class="button is-info" @click="toggleDisplayPast()">
                {{ this.displayPast ? 'Display only future events' : 'Display past events' }}
              </a>
            </div>
          </div>
        </div>

        <div class="field">
          <label class="label">Event types</label>
          <label v-for="(value, index) in eventTypes" v-bind:key="index">
            <input class="checkbox" type="checkbox" v-model="value.enabled" @change="refetch()" />
            {{ value.name }}
          </label>
        </div>

        <div class="field">
          <div class="control">
            <router-link class="button is-primary" :to="{ name: 'oms.events.create' }">Create event</router-link>
          </div>
        </div>

        <div class="card" v-for="event in events" v-bind:key="event.id">
          <div class="card-content">
            <div class="media">
              <div class="media-left">
                <figure class="image is-96x96">
                  <img src="/images/logo.png">
                </figure>
              </div>
              <div class="media-content">
                <router-link :to="{ name: 'oms.events.view', params: { id: event.url || event.id } }">
                  <p class="title is-4">{{ event.name }}</p>
                </router-link>
              </div>
            </div>

            <div class="content">
              <span v-html="$options.filters.markdown(event.description)"></span>
              <ul>
                <li><strong>Type:</strong> {{ event.type }} </li>
                <li><strong>From:</strong> {{ event.starts | date }} </li>
                <li><strong>To:</strong> {{ event.ends | date }} </li>
                <li><strong>Application deadline:</strong>
                  <span>{{ event.application_ends | date }}</span>
                </li>
              </ul>

              <div class="field is-grouped">
                <p class="control">
                  <router-link
                    :to="{ name: 'oms.events.view', params: { id: event.url || event.id } }"
                    class="button">Go to event page</router-link>
                </p>
                <p class="control" v-if="event.application_status === 'open'">
                  <router-link
                    :to="{ name: 'oms.events.apply', params: { id: event.url || event.id } }"
                    class="button is-warning">
                    {{ scope === 'participating' ? 'Manage my application' : 'Apply' }}
                  </router-link>
                </p>
              </div>
            </div>
          </div>
        </div>

        <div class="card" v-show="events.length === 0 && !isLoading">
          <div class="card-content">
            <div class="media">
              <div class="media-content">
                <p class="title has-text-centered">No events found.</p>
              </div>
            </div>
          </div>
        </div>


        <div class="field">
          <button
            class="button is-primary is-fullwidth"
            :class="{ 'is-loading': isLoading }"
            :disabled="isLoading"
            v-show="canLoadMore"
            @click="fetchData()">Load more events</button>
        </div>
      </article>
    </div>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'

export default {
  name: 'EventsList',
  data () {
    return {
      events: [],
      eventTypes: [
        { value: 'wu', name: 'New Year Event', enabled: false },
        { value: 'es', name: 'ES', enabled: false },
        { value: 'nwm', name: 'NWM', enabled: false },
        { value: 'ltc', name: 'LTC', enabled: false },
        { value: 'rtc', name: 'RTC', enabled: false },
        { value: 'local', name: 'Local event', enabled: false },
        { value: 'other', name: 'Other', enabled: false }
      ],
      isLoading: false,
      query: '',
      limit: 30,
      offset: 0,
      displayPast: false,
      canLoadMore: true,
      source: null
    }
  },
  computed: {
    queryObject () {
      const queryObj = {
        limit: this.limit,
        offset: this.offset,
        type: this.eventTypes.filter(type => type.enabled).map(type => type.value),
        displayPast: this.displayPast
      }

      if (this.query) queryObj.search = this.query
      return queryObj
    },
    scope () {
      if (this.$route.name === 'oms.events.list.all') return 'all'
      if (this.$route.name === 'oms.events.list.participating') return 'participating'
      if (this.$route.name === 'oms.events.list.organizing') return 'organizing'

      throw new Error('Unknown scope: ' + this.$route.name)
    },
    ...mapGetters(['services'])
  },
  methods: {
    toggleDisplayPast () {
      this.displayPast = !this.displayPast
      this.refetch()
    },
    refetch () {
      this.events = []
      this.offset = 0
      this.canLoadMore = true
      this.fetchData()
    },
    fetchData (state) {
      this.isLoading = true
      if (this.source) this.source.cancel()
      this.source = this.axios.CancelToken.source()

      const urls = {
        all: this.services['oms-events'],
        organizing: this.services['oms-events'] + '/mine/organizing',
        participating: this.services['oms-events'] + '/mine/participating'
      }

      this.axios.get(urls[this.scope], { params: this.queryObject, cancelToken: this.source.token }).then((response) => {
        this.events = this.events.concat(response.data.data)
        this.offset += this.limit
        this.canLoadMore = this.scope === 'all' && response.data.data.length === this.limit

        this.isLoading = false
      }).catch((err) => {
        if (this.axios.isCancel(err)) {
          return console.debug('Request cancelled.')
        }

        this.$root.showDanger('Could not fetch events list: ' + err.message)
      })
    }
  },
  watch: {
    '$route.name' () {
      this.refetch()
    }
  },
  mounted () {
    this.refetch()
  }
}
</script>
