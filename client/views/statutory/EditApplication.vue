<template>
  <div class="tile is-ancestor">
    <div class="tile is-parent">
      <div class="tile is-child">
        <div class="title" v-show="isNew">Apply to {{ event.name }}</div>
        <div class="title" v-show="!isNew">Edit application on {{ event.name }}</div>

        <form @submit.prevent="saveApplication()">
          <div class="tile is-parent">
            <div class="tile is-child">
              <div class="field" v-show="autoComplete.bodies.values.length > 0">
                <label class="label">Select body to apply from</label>
                <div class="control">
                  <div class="field has-addons">
                    <b-autocomplete
                      v-model="autoComplete.bodies.name"
                      :data="autoComplete.bodies.values"
                      open-on-focus="true"
                      @select="body => { $set(application, 'body_id', body.id); $set(application, 'body', body) }">
                      <template slot-scope="props">
                        <div class="media">
                          <div class="media-content">
                            {{ props.option.name }}
                          </div>
                        </div>
                      </template>
                    </b-autocomplete>
                    <p class="control">
                      <a class="button is-danger"
                        @click="body => { $set(application, 'body_id', null); $delete(application, 'body') }"
                        v-if="application.body">{{ application.body.name }} (Click to unset)</a>
                      <a class="button is-static" v-if="!application.body">Not set.</a>
                    </p>
                  </div>
                </div>
              </div>

              <div class="notification is-danger" v-show="autoComplete.bodies.values.length == 0">
                <div class="content">
                  <p>You are not a member of any body, therefore you cannot apply.</p>
                  <p>To apply to a statutory event, you need to be a member of at least one body.</p>
                  <p>For that, go to Bodies, send a join request to the body you are in and wait for the board to approve it</p>
                </div>
              </div>

              <div class="notification is-danger" v-if="errors.answers || errors.body_id || errors.gender|| errors.date_of_birth">
                <div class="content">
                Could not apply because of these reasons:
                  <ul>
                    <li v-for="(error, index) in errors.answers" v-bind:key="index">{{ error }}</li>
                  </ul>
                  <ul>
                    <li v-for="(error, index) in errors.body_id" v-bind:key="index">{{ error }}</li>
                  </ul>
                  <ul v-if="errors.gender">
                    <li>Please set the gender in <router-link :to="{ name: 'oms.members.view', params: { id: 'me' } }" target='_blank'>your profile.</router-link></li>
                  </ul>
                  <ul v-if="errors.date_of_birth">
                    <li>Please set the date of birth in <router-link :to="{ name: 'oms.members.view', params: { id: 'me' } }" target='_blank'>your profile.</router-link></li>
                  </ul>
                </div>
              </div>

              <div class="field">
                <label class="label">Nationality <span class="has-text-danger">*</span></label>
                <div class="control">
                  <div class="select">
                    <select v-model="application.nationality">
                      <option v-for="(country, index) in countries" v-bind:key="index">{{ country }}</option>
                    </select>
                  </div>
                </div>
                <p class="help is-danger" v-if="errors.nationality">{{ errors.nationality.join(', ') }}</p>
              </div>

              <div class="field">
                <label class="label">Meals type <span class="has-text-danger">*</span></label>
                <div class="control">
                  <div class="select">
                    <select v-model="application.meals">
                      <option>Meat-eater</option>
                      <option>Vegetarian</option>
                    </select>
                  </div>
                </div>
                <p class="help is-danger" v-if="errors.meals">{{ errors.meals.join(', ') }}</p>
              </div>

              <div class="field is-fullwidth">
                <label class="label">Allergies</label>
                <div class="control">
                  <textarea
                    class="textarea"
                    v-model="application.allergies" />
                </div>
              </div>

              <div class="field is-fullwidth">
                <label class="label">Number of {{ event.type === 'agora' ? 'Agorae' : 'EPM' }} visited <span class="has-text-danger">*</span></label>
                <div class="control">
                  <input
                    class="input"
                    type="number"
                    required
                    min="0"
                    v-model="application.number_of_events_visited" />
                </div>
              </div>

              <div class="field is-fullwidth">
                <label class="has-text-weight-bold checkbox">
                  Visa required?
                  <input
                    class="checkbox"
                    type="checkbox"
                    v-model="application.visa_required" />
                </label>
              </div>

              <div class="field is-fullwidth" v-if="application.visa_required">
                <label class="has-text-weight-bold">
                  Passport number <span class="has-text-danger">*</span>
                </label>
                <div class="control">
                  <input
                    class="input"
                    type="text"
                    required
                    v-model="application.visa_passport_number" />
                </div>
              </div>

              <div class="field is-fullwidth" v-if="application.visa_required">
                <label class="has-text-weight-bold">
                  Passport issue date <span class="has-text-danger">*</span>
                </label>
                <div class="control">
                  <input
                    class="input"
                    type="text"
                    required
                    v-model="application.visa_passport_issue_date" />
                </div>
              </div>

              <div class="field is-fullwidth" v-if="application.visa_required">
                <label class="has-text-weight-bold">
                  Passport expiration date <span class="has-text-danger">*</span>
                </label>
                <div class="control">
                  <input
                    class="input"
                    type="text"
                    required
                    v-model="application.visa_passport_expiration_date" />
                </div>
              </div>

              <div class="field is-fullwidth" v-if="application.visa_required">
                <label class="has-text-weight-bold">
                  Passport issue authority <span class="has-text-danger">*</span>
                </label>
                <div class="control">
                  <input
                    class="input"
                    type="text"
                    required
                    v-model="application.visa_passport_issue_authority" />
                </div>
              </div>

              <div class="field is-fullwidth" v-if="application.visa_required">
                <label class="has-text-weight-bold">
                  Place of birth <span class="has-text-danger">*</span>
                </label>
                <div class="control">
                  <input
                    class="input"
                    type="text"
                    required
                    v-model="application.visa_place_of_birth" />
                </div>
              </div>

              <div class="field is-fullwidth" v-if="application.visa_required">
                <label class="has-text-weight-bold">
                  Embassy <span class="has-text-danger">*</span>
                </label>
                <div class="control">
                  <input
                    class="input"
                    type="text"
                    required
                    v-model="application.visa_embassy" />
                </div>
              </div>

              <div class="field is-fullwidth" v-if="application.visa_required">
                <label class="has-text-weight-bold">
                  Street and house number <span class="has-text-danger">*</span>
                </label>
                <div class="control">
                  <input
                    class="input"
                    type="text"
                    required
                    v-model="application.visa_street_and_house" />
                </div>
              </div>

              <div class="field is-fullwidth" v-if="application.visa_required">
                <label class="has-text-weight-bold">
                  Postal code <span class="has-text-danger">*</span>
                </label>
                <div class="control">
                  <input
                    class="input"
                    type="text"
                    required
                    v-model="application.visa_postal_code" />
                </div>
              </div>

              <div class="field is-fullwidth" v-if="application.visa_required">
                <label class="has-text-weight-bold">
                  City <span class="has-text-danger">*</span>
                </label>
                <div class="control">
                  <input
                    class="input"
                    type="text"
                    required
                    v-model="application.visa_city" />
                </div>
              </div>

              <div class="field is-fullwidth" v-if="application.visa_required">
                <label class="has-text-weight-bold">
                  Country <span class="has-text-danger">*</span>
                </label>
                <div class="control">
                  <input
                    class="input"
                    type="text"
                    required
                    v-model="application.visa_country" />
                </div>
              </div>

              <div class="field is-fullwidth" v-for="(question, index) in event.questions" v-bind:key="index">
                <div class="control">
                  <label class="has-text-weight-bold">
                    {{ question.description }} <span class="has-text-danger" v-if="question.required">*</span>
                  </label>
                </div>
                <div class="control" v-if="question.type === 'text'">
                  <textarea
                    class="textarea"
                    :required="question.required"
                    v-model="application.answers[index]" />
                </div>
                <div class="control" v-if="question.type === 'string'">
                  <input
                    class="input"
                    type="text"
                    :required="question.required"
                    v-model="application.answers[index]" />
                </div>
                <div class="control" v-if="question.type === 'number'">
                  <input
                    class="input"
                    type="number"
                    v-model.number="application.answers[index]" />
                </div>
                <div class="control" v-if="question.type === 'checkbox'">
                  <input
                    class="checkbox"
                    type="checkbox"
                    :required="question.required"
                    v-model="application.answers[index]" />
                </div>
                <div class="control" v-if="question.type === 'select'">
                  <div class="select">
                    <select v-model="application.answers[index]" required>
                      <option v-for="(value, index) in question.values" v-bind:key="index">{{ value }}</option>
                    </select>
                  </div>
                </div>
              </div>

              <div class="field">
                <button type="submit" class="button is-primary">
                  Save application!
                </button>
              </div>
            </div>
          </div>

          <b-loading is-full-page="false" :active.sync="isLoading"></b-loading>
        </form>

        <hr v-show="!isNew && can.set_board_comment_and_participant_type_global" />

        <!-- Editing board stuff for Chair Team/CD -->
        <div class="tile is-parent" v-show="!isNew && can.set_board_comment_and_participant_type_global">
          <div class="tile is-child">
            <div class="field is-fullwidth">
              <div class="control">
                <label class="has-text-weight-bold">Board comment</label>
              </div>
              <div class="control">
                <textarea
                  class="textarea"
                  required
                  v-model="application.board_comment" />
              </div>
            </div>

            <div class="field is-fullwidth">
              <div class="control">
                <label class="has-text-weight-bold">Application type</label>
              </div>
              <div class="control">
                <div class="select">
                  <select v-model="application.participant_type">
                    <option value="delegate">Delegate</option>
                    <option value="visitor">Visitor</option>
                    <option value="envoy">Envoy</option>
                    <option value="observer">Observer</option>
                  </select>
                </div>
              </div>
            </div>

            <div class="field is-fullwidth">
              <div class="control">
                <label class="has-text-weight-bold">Participant order</label>
              </div>
              <div class="control">
                <input
                  class="input"
                  type="number"
                  min="1"
                  v-model.number="application.participant_order" />
              </div>
            </div>

            <div class="field">
              <button type="submit" class="button is-primary" @click="saveBoard()">
                Save application!
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>

</template>


<script>
import { mapGetters } from 'vuex'
import countries from '../../countries'

export default {
  name: 'EditApplication',
  data () {
    return {
      event: {
        name: null,
        questions: []
      },
      autoComplete: { bodies: { name: '', values: [] } },
      application: {
        body: null,
        body_id: null,
        id: null,
        visa_required: false,
        answers: [],
        meals: '',
        allergies: '',
        number_of_events_visited: 0
      },
      countries,
      can: {
        apply: false
      },
      errors: {},
      isLoading: false,
      isSaving: false
    }
  },
  methods: {
    saveApplication () {
      if (!this.application.body_id) {
        return this.$root.showDanger('Please select a body.')
      }

      this.errors = {}

      const promise = this.isNew
        ? this.axios.post(this.services['oms-statutory'] + '/events/' + this.$route.params.id + '/applications', this.application)
        : this.axios.put(this.services['oms-statutory'] + '/events/' + this.$route.params.id + '/applications/' + this.$route.params.application_id, this.application)

      promise.then((application) => {
        this.$root.showSuccess('Application is saved.')

        return this.$router.push({
          name: 'oms.statutory.applications.view',
          params: { id: this.$route.params.id, application_id: application.data.data.id }
        })
      }).catch((err) => {
        this.isSaving = false

        if (err.response.status === 422) { // validation errors
          this.errors = err.response.data.errors
          return this.$root.showDanger('Some of the application data is invalid.')
        }

        this.$root.showDanger('Could not save application: ' + err.message)
      })
    },
    saveBoard () {
      // Copy data from the form into an object to submit it in the format the backend needs it
      this.isSaving = true

      const toServer = {
        participant_type: this.application.participant_type,
        participant_order: this.application.participant_order,
        board_comment: this.application.board_comment
      }

      this.axios.put(this.services['oms-statutory'] + '/events/' + this.$route.params.id + '/applications/' + this.$route.params.application_id + '/board', toServer).then(() => {
        this.$root.showSuccess('Application is saved.')
      }).catch((err) => {
        this.isSaving = false

        if (err.response.status === 422) { // validation errors
          this.errors = err.response.data.errors
          return this.$root.showDanger('Some of the application data is invalid.')
        }

        this.$root.showDanger('Could not save application: ' + err.message)
      })
    },
    askSetCancelled (value) {
      this.$dialog.confirm({
        title: 'Cancel application',
        message: 'Are you sure you want to <b>cancel your application</b>? You can uncancel it only during application period.',
        confirmText: 'Cancel application',
        type: 'is-danger',
        hasIcon: true,
        onConfirm: () => this.setCancelled(value)
      })
    },
    setCancelled (value) {
      this.axios.put(this.services['oms-statutory'] + '/events/' + this.$route.params.id + '/applications/' + this.application.id + '/cancel', {
        cancelled: value
      }).then((response) => {
        this.application.cancelled = value
        this.$root.showInfo(value ? 'Application is cancelled.' : 'Application is uncancelled.')
      }).catch((err) => this.$root.showDanger('Could not cancel application: ' + err.message))
    }
  },
  computed: {
    ...mapGetters({
      services: 'services',
      loginUser: 'user'
    }),
    isNew () {
      return !this.$route.params.application_id
    }
  },
  watch: {
    'application.visa_required' (val) {
      if (!val) {
        this.application.visa_passport_number = ''
        this.application.visa_place_of_birth = ''
        this.application.visa_passport_number = ''
        this.application.visa_passport_issue_date = ''
        this.application.visa_passport_expiration_date = ''
        this.application.visa_passport_issue_authority = ''
        this.application.visa_embassy = ''
        this.application.visa_street_and_house = ''
        this.application.visa_postal_code = ''
        this.application.visa_city = ''
        this.application.visa_country = ''
      }
    }
  },
  mounted () {
    this.isLoading = true
    this.axios.get(this.services['oms-statutory'] + '/events/' + this.$route.params.id).then((response) => {
      this.event = response.data.data
      this.can = response.data.data.permissions

      // Prefilling default values for application answers
      this.application.answers = Array.from({ length: this.event.questions.length }, (value, index) => {
        switch (this.event.questions[index].type) {
          case 'number':
            return 0
          case 'checkbox':
            return false
          case 'select':
            return this.event.questions[index].values[0]
          default:
            return ''
        }
      })

      // If this is a new application, then allowing selecting bodies which the current user is a member of
      // and not fetching application.
      if (this.isNew) {
        this.autoComplete.bodies.values = this.loginUser.bodies
        this.isLoading = false
        return
      }

      // Fetching application
      this.axios.get(this.services['oms-statutory'] + '/events/' + this.$route.params.id + '/applications/' + this.$route.params.application_id).then((application) => {
        this.application = application.data.data
        this.can = application.data.data.permissions

        // Fetching user to get his/her bodies
        return this.axios.get(this.services['oms-core-elixir'] + '/members/' + this.application.user_id)
      }).then((user) => {
        this.autoComplete.bodies.values = user.data.data.bodies
        this.application.body = user.data.data.bodies.find(body => body.id === this.application.body_id)
        this.isLoading = false
      }).catch((err) => {
        this.isLoading = false

        // if there's no application, just ignore, otherwise re-throw the error
        if (!err.response || err.response.status !== 404) {
          throw err
        }
      })
    }).catch((err) => {
      let message = (err.response && err.response.status === 404) ? 'Application is not found' : 'Some error happened: ' + err.message

      this.$root.showDanger(message)
      this.$router.push({
        name: 'oms.statutory.applications.view',
        params: { id: this.$route.params.id, application_id: this.$route.params.application_id }
      })
    })
  }
}
</script>
