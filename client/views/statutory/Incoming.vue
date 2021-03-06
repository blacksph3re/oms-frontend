<template>
  <div class="tile is-ancestor">
    <div class="tile is-parent">
      <div class="tile is-child">
        <div class="title">Incoming</div>

        <div class="field">
          <label class="label">Search by name or email</label>
          <div class="control is-expanded">
            <input class="input" type="text" v-model="query" placeholder="Search by name, surname or email" />
          </div>
        </div>

        <div>Total participants: {{ filteredApplications.length }}</div>

        <b-table
          :data="filteredApplications"
          :loading="isLoading"
          paginated
          :per-page="limit"
          default-sort="id"
          default-sort-direction="desc">
          <template slot-scope="props">
            <b-table-column field="id" label="#" numeric sortable>
              {{ props.row.id }}
            </b-table-column>

            <b-table-column field="user_id" label="User ID" sortable>
              {{ props.row.user_id }}
            </b-table-column>

            <b-table-column field="first_name" label="First name" centered sortable>
              {{ props.row.first_name | beautify }}
            </b-table-column>

            <b-table-column field="last_name" label="Last name" centered sortable>
              {{ props.row.last_name | beautify }}
            </b-table-column>

            <b-table-column field="email" label="Email" centered sortable>
              {{ props.row.email }}
            </b-table-column>

            <b-table-column field="body_name" label="Body" centered sortable>
              {{ props.row.body_name }}
            </b-table-column>

            <b-table-column field="meals" label="Meals" centered>
              {{ props.row.meals }}
            </b-table-column>

            <b-table-column field="allergies" label="Allergies" centered>
              {{ props.row.allergies }}
            </b-table-column>

            <b-table-column field="paid_fee" label="Confirmed?" centered sortable>
              <div class="select" :class="{ 'is-loading': props.row.isSavingPaidFee }">
                <select v-model="props.row.newPaidFee" @change="switchPaxPaidFee(props.row)" :disabled="props.row.attended">
                  <option :value="true">Yes</option>
                  <option :value="false">No</option>
                </select>
              </div>
            </b-table-column>

            <b-table-column field="attended" label="Attended?" centered sortable>
              <div class="select" :class="{ 'is-loading': props.row.isSavingAttended }">
                <select v-model="props.row.newAttended" @change="switchPaxAttended(props.row)" :disabled="!props.row.paid_fee">
                  <option :value="true">Yes</option>
                  <option :value="false">No</option>
                </select>
              </div>
            </b-table-column>

            <b-table-column label="View" centered>
              <router-link :to="{ name: 'oms.statutory.applications.view', params: { id: event.url || event.id, application_id: props.row.id } }">
                View
              </router-link>
            </b-table-column>
          </template>

          <template slot="empty">
            <section class="section">
              <div class="content has-text-grey has-text-centered">
                <p>
                  <b-icon icon="fa fa-times-circle" size="is-large"></b-icon>
                </p>
                <p>Nothing here.</p>
              </div>
            </section>
          </template>
        </b-table>
      </div>
    </div>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'

export default {
  name: 'StatutoryIncoming',
  data () {
    return {
      applications: [],
      event: {
        questions: []
      },
      query: '',
      limit: 50,
      isLoading: false,
      isSaving: false
    }
  },
  computed: {
    ...mapGetters({
      services: 'services',
      loginUser: 'user'
    }),
    filteredApplications () {
      const lowercaseQuery = this.query.toLowerCase()

      return this.applications.filter(app =>
        ['first_name', 'last_name', 'email'].some(field => app[field].toLowerCase().includes(lowercaseQuery)))
    }
  },
  methods: {
    switchPaxAttended (pax) {
      pax.isSavingAttended = true
      const url = this.services['oms-statutory'] + '/events/' + this.$route.params.id + '/applications/' + pax.id + '/attended'

      this.axios.put(url, { attended: pax.newAttended }).then(() => {
        pax.attended = pax.newAttended
        pax.isSavingAttended = false
        this.$root.showSuccess(`Successfully updated attendance info of application for user #${pax.user_id}`)
      }).catch((err) => {
        pax.isSavingAttended = false
        this.$root.showDanger('Could not update participant attendance info: ' + err.message)
      })
    },
    switchPaxPaidFee (pax) {
      pax.isSavingPaidFee = true
      const url = this.services['oms-statutory'] + '/events/' + this.$route.params.id + '/applications/' + pax.id + '/paid_fee'

      this.axios.put(url, { paid_fee: pax.newPaidFee }).then(() => {
        pax.paid_fee = pax.newPaidFee
        pax.isSavingPaidFee = false
        this.$root.showSuccess(`Successfully updated fee info of application for user #${pax.user_id}`)
      }).catch((err) => {
        pax.isSavingPaidFee = false
        this.$root.showDanger('Could not update participant fee info: ' + err.message)
      })
    }
  },
  mounted () {
    this.isLoading = true

    this.axios.get(this.services['oms-statutory'] + '/events/' + this.$route.params.id).then((event) => {
      this.event = event.data.data

      return this.axios.get(this.services['oms-statutory'] + '/events/' + this.$route.params.id + '/applications/incoming')
    }).then((application) => {
      this.applications = application.data.data
      this.isLoading = false

      // Fetching users and bodies.
      for (const pax of this.applications) {
        this.$set(pax, 'newPaidFee', pax.paid_fee)
        this.$set(pax, 'newAttended', pax.attended)
        this.$set(pax, 'isSavingPaidFee', false)
        this.$set(pax, 'isSavingAttended', false)
      }
    }).catch((err) => {
      this.isLoading = false
      let message = (err.response.status === 404) ? 'Event is not found' : 'Some error happened: ' + err.message

      this.$root.showDanger(message)
      this.$router.push({ name: 'oms.statutory.list' })
    })
  }
}
</script>
