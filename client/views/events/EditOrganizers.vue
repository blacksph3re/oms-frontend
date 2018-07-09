<template>
  <div class="tile is-ancestor ">
    <div class="tile is-child">
      <div class="tabs is-centered is-boxed">
        <ul>
          <li :class="{'is-active': scope === 'organizers' }" @click="scope = 'organizers'">
            <a>
              <span class="icon is-small"><i class="fa fa-users" aria-hidden="true"></i></span>
              <span>Organizers</span>
            </a>
          </li>
          <li :class="{'is-active': scope === 'locals' }" @click="scope = 'locals'">
            <a>
              <span class="icon is-small"><i class="fa fa-flag" aria-hidden="true"></i></span>
              <span>Organizing locals</span>
            </a>
          </li>
        </ul>
      </div>

      <div v-show="scope === 'organizers'">
        <table class="table is-fullwidth" v-if="$route.params.id">
          <thead>
            <tr>
              <th>User ID</th>
              <th>User name</th>
              <th>Comment</th>
              <th>Roles</th>
              <th>Add role</th>
              <th></th>
              <th></th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(organizer, index) in event.organizers" v-bind:key="organizer.user_id">
              <td>{{ organizer.user_id }}</td>
              <td v-if="organizer.user">
                <router-link :to="{ name: 'oms.members.view', params: { id: organizer.user_id } }">
                  {{ organizer.user.first_name }} {{ organizer.user.last_name }}
                </router-link>
              </td>
              <td v-if="!organizer.user"><i>Loading...</td>
              <td>
                <div class="control">
                  <input class="input" type="text" v-model="organizer.comment" @input="organizer.changed = true"/>
                </div>
              </td>
              <td>
                <ul>
                  <li v-for="(role, roleIndex) in organizer.roles" v-bind:key="role._id">
                    <a class="tag is-danger is-medium">
                      {{ role.name }}
                      <button class="delete is-small" @click="deletePseudoRole(index, roleIndex)" />
                    </a>
                  </li>
                </ul>
              </td>
              <td>
                <div class="field has-addons">
                  <b-autocomplete
                    v-model="autoComplete.roles.name"
                    :data="roles"
                    open-on-focus="true"
                    @select="role => addPseudoRole(index, role)">
                    <template slot-scope="props">
                      <div class="media">
                        <div class="media-content">
                          {{ props.option.name }}
                          <br>
                          <small> {{ props.option.description }} </small>
                        </div>
                      </div>
                    </template>
                  </b-autocomplete>
                </div>
              </td>
              <td>
                <button v-if="event.organizers.length > 1" class="button is-danger" @click="askDeleteOrganizer(index)">
                  Delete organizer
                  </button>
              </td>
              <td>
                <button :disabled="!organizer.changed" class="button is-primary" @click="saveOrganizer(index)">Save</button>
              </td>
            </tr>
          </tbody>
        </table>

        <div class="field">
          <label class="label">Add organizer</label>
          <div class="control">
            <div class="field has-addons">
              <b-autocomplete
                v-model="autoComplete.members.name"
                :data="autoComplete.members.values"
                open-on-focus="true"
                :loading="autoComplete.members.loading"
                @input="query => fetchSomething(query, 'members', 'members')"
                @select="organizer => addOrganizer(organizer)">
                <template slot-scope="props">
                  <div class="media">
                    <div class="media-content">
                        {{ props.option.first_name }}
                        <br>
                        <small> {{ props.option.last_name }} </small>
                    </div>
                  </div>
                </template>
              </b-autocomplete>
            </div>
          </div>
        </div>
      </div>

      <div v-show="scope === 'locals'">
        <ul>
          <li v-for="(local, index) in event.organizing_locals" v-bind:key="local.body_id">
            <a class="tag is-danger is-medium" v-if="local.body">
              {{ local.body.name }}
              <button class="delete is-small" v-if="event.organizing_locals.length > 1" @click="askDeleteOrganizingLocal(index)" />
            </a>
            <a v-if="!local.body" class="tag is-medium">Loading...</a>
          </li>
        </ul>


        <div class="field">
          <label class="label">Add organizing body</label>
          <div class="control">
            <div class="field has-addons">
              <b-autocomplete
                v-model="autoComplete.bodies.name"
                :data="autoComplete.bodies.values"
                open-on-focus="true"
                :loading="autoComplete.bodies.loading"
                @input="query => fetchSomething(query, 'bodies', 'bodies')"
                @select="local => addOrganizingLocal(local)">
                <template slot-scope="props">
                  <div class="media">
                    <div class="media-content">
                        {{ props.option.name }}
                        <br>
                        <small> {{ props.option.description }} </small>
                    </div>
                  </div>
                </template>
              </b-autocomplete>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'

export default {
  name: 'EditEvent',
  data () {
    return {
      event: {
        organizers: [],
        organizing_locals: []
      },
      autoComplete: {
        members: { name: '', values: [], loading: false },
        bodies: { name: '', values: [], loading: false },
        roles: { name: '' }
      },
      roles: [],
      errors: {},
      scope: 'organizers',
      isLoading: false,
      isSaving: false
    }
  },
  methods: {
    fetchSomething (query, key, context) {
      if (!query) return

      this.autoComplete[key].values = []
      this.autoComplete[key].loading = true

      if (this.token) this.token.cancel()
      this.token = this.axios.CancelToken.source()

      this.axios.get(this.services['oms-core-elixir'] + '/' + context, {
        cancelToken: this.token.token,
        params: { query }
      }).then((response) => {
        this.autoComplete[key].values = response.data.data
        this.autoComplete[key].loading = false
      }).catch((err) => {
        if (this.axios.isCancel(err)) {
          return console.log('Request canceled')
        }

        this.autoComplete[key].loading = false

        this.$toast.open({
          duration: 3000,
          message: 'Could not fetch ' + context + 's: ' + err.message,
          type: 'is-danger'
        })
      })
    },
    addOrganizer (organizer) {
      const data = {
        user_id: organizer.id,
        roles: []
      }
      this.axios.post(this.services['oms-events'] + '/single/' + this.event.id + '/organizers', data).then((response) => {
        this.$toast.open({
          duration: 3000,
          message: 'Organizer is successfully added.',
          type: 'is-success'
        })

        this.event.organizers.push({
          user_id: organizer.id,
          user: organizer,
          roles: []
        })
      }).catch((err) => {
        this.$toast.open({
          duration: 3000,
          message: 'Could not add organizer: ' + err.message,
          type: 'is-danger'
        })
      })
    },
    addPseudoRole (index, role) {
      const organizer = this.event.organizers[index]
      organizer.roles.push(role)

      organizer.changed = true
    },
    deletePseudoRole (index, roleIndex) {
      this.event.organizers[index].roles.splice(roleIndex, 1)
      this.event.organizers[index].changed = true
    },
    saveOrganizer (index) {
      const organizer = this.event.organizers[index]

      const data = {
        roles: organizer.roles,
        comment: organizer.comment
      }

      this.axios.put(this.services['oms-events'] + '/single/' + this.event.id + '/organizers/' + organizer.user_id, data).then((response) => {
        this.$toast.open({
          duration: 3000,
          message: 'Organizer is successfully updated.',
          type: 'is-success'
        })
        organizer.changed = false
        this.$forceUpdate()
      }).catch((err) => {
        this.$toast.open({
          duration: 3000,
          message: 'Could not update organizer: ' + err.message,
          type: 'is-danger'
        })
      })
    },
    askDeleteOrganizer (index) {
      this.$dialog.confirm({
        title: 'Deleting organizer',
        message: 'Are you sure you want to <b>delete</b> organizer from this event? This action cannot be undone.',
        confirmText: 'Delete organizer',
        type: 'is-danger',
        hasIcon: true,
        onConfirm: () => this.deleteOrganizer(index)
      })
    },
    deleteOrganizer (index) {
      const organizer = this.event.organizers[index]
      this.axios.delete(this.services['oms-events'] + '/single/' + this.event.id + '/organizers/' + organizer.user_id).then((response) => {
        this.$toast.open({
          duration: 3000,
          message: 'Organizer is successfully deleted.',
          type: 'is-success'
        })
        this.event.organizers.splice(index, 1)
      }).catch((err) => {
        this.$toast.open({
          duration: 3000,
          message: 'Could not delete organizer: ' + err.message,
          type: 'is-danger'
        })
      })
    },
    addOrganizingLocal (local) {
      const data = {
        body_id: local.id
      }
      this.axios.post(this.services['oms-events'] + '/single/' + this.event.id + '/locals', data).then((response) => {
        this.$toast.open({
          duration: 3000,
          message: 'Organizer is successfully added.',
          type: 'is-success'
        })

        this.event.organizing_locals.push({
          body_id: local.id,
          body: local,
          roles: []
        })
      }).catch((err) => {
        this.$toast.open({
          duration: 3000,
          message: 'Could not add organizing local: ' + err.message,
          type: 'is-danger'
        })
      })
    },
    askDeleteOrganizingLocal (index) {
      this.$dialog.confirm({
        title: 'Deleting organizing body',
        message: 'Are you sure you want to <b>delete</b> organizing body from this event? This action cannot be undone.',
        confirmText: 'Delete organizing body',
        type: 'is-danger',
        hasIcon: true,
        onConfirm: () => this.deleteOrganizingLocal(index)
      })
    },
    deleteOrganizingLocal (index) {
      const local = this.event.organizing_locals[index]
      this.axios.delete(this.services['oms-events'] + '/single/' + this.event.id + '/locals/' + local.body_id).then((response) => {
        this.$toast.open({
          duration: 3000,
          message: 'Organizing body is successfully deleted.',
          type: 'is-success'
        })
        this.event.organizing_locals.splice(index, 1)
      }).catch((err) => {
        this.$toast.open({
          duration: 3000,
          message: 'Could not delete organizing body: ' + err.message,
          type: 'is-danger'
        })
      })
    }
  },
  computed: mapGetters(['services']),
  mounted () {
    this.axios.get(this.services['oms-events'] + '/eventroles').then((response) => {
      this.roles = response.data.data
    }).catch((err) => {
      this.$toast.open({
        duration: 3000,
        message: 'Could not fetch event roles: ' + err.message,
        type: 'is-danger'
      })
    })

    this.axios.get(this.services['oms-events'] + '/single/' + this.$route.params.id).then((response) => {
      this.event = response.data.data
      this.can = response.data.permissions.can

      this.event.starts = new Date(this.event.starts)
      this.event.ends = new Date(this.event.ends)
      if (this.event.application_deadline) this.event.application_deadline = new Date(this.event.application_deadline)

      this.isLoading = false
      this.$forceUpdate()

      for (const body of this.event.organizing_locals) {
        this.axios.get(this.services['oms-core-elixir'] + '/bodies/' + body.body_id).then((response) => {
          body.body = response.data.data
          this.$forceUpdate()
        }).catch(console.error)
      }

      for (const member of this.event.organizers) {
        this.axios.get(this.services['oms-core-elixir'] + '/members/' + member.user_id).then((response) => {
          member.user = response.data.data
          this.$forceUpdate()
        }).catch(console.error)
      }
    }).catch((err) => {
      let message = (err.response.status === 404) ? 'Event is not found' : 'Some error happened: ' + err.message

      this.$toast.open({
        duration: 3000,
        message,
        type: 'is-danger'
      })
      this.$router.push({ name: 'oms.events.list' })
    })
  }
}
</script>