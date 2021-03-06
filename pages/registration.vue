<template>
  <b-row>
    <b-col>
      <h1>{{ $t("titleReg") }}</h1>
      <global-notification />
      <h2>{{ $t("personal_info") }}</h2>
      <ValidationObserver ref="observer" v-slot="{ passes }">
        <b-form @submit.prevent="passes(onSubmit)" @reset.prevent="onReset">
          <user
            ref="user"
            v-model="user"
            :field-status="user_field_status"
          />
          <h1>{{ $t("Project") }}</h1>
          <b-form-group>
            <b-form-radio v-model="is_own_project" name="own_project" value="own">
              {{ $t("eigenProject") }}
            </b-form-radio>
            <b-form-radio v-model="is_own_project" name="own_project" value="other">
              {{ $t("medeProject") }}
            </b-form-radio>
          </b-form-group>
          <div v-if="is_own_project === 'own'">
            <own-project
              ref="ownProject"
              v-model="own_project"
              :field-status="own_project_field_status"
            />
          </div>
          <div v-else>
            <other-project
              ref="otherProject"
              v-model="other_project"
              :field-status="other_project_field_status"
            />
          </div>
          <mandatory-questions ref="mandatoryQuestions" v-model="mandatory_approvals" />
          <b-form-group>
            <b-button
              :disabled="loading"
              type="submit"
              variant="info"
              class="button-hero"
            >
              <span
                v-if="loading"
              ><b-spinner type="grow" /> {{ $t("pleaseWait") }}</span>
              <span
                v-else
              ><font-awesome-icon
                 v-if="!loading"
                 :icon="['fas', 'paper-plane']"
               />
                {{ $t("Ik schrijf me in") }}</span>
            </b-button>
            <b-button
              :disabled="loading"
              type="reset"
              variant="warning"
              class="button-hero"
            >
              <font-awesome-icon :icon="['fas', 'trash-restore']" />
              {{ $t("verwijder alles") }}
            </b-button>
          </b-form-group>
        </b-form>
      </ValidationObserver>
    </b-col>
  </b-row>
</template>
<script>
import { ValidationObserver } from 'vee-validate'

export default {
  middleware: 'notAuthenticated',
  components: {
    ValidationObserver
  },
  async asyncData ({ store, app, redirect }) {
    const settings = await app.$services.settings.get()
    if (!settings) {
      app.router.push(app.localePath('no_event'))
    }
  },
  data () {
    return {
      loading: false,
      is_own_project: 'own',
      readWrite: true
    }
  },
  computed: {
    other_project_field_status () {
      return {
        project_code: {
          rw: this.readWrite,
          hidden: false
        }
      }
    },
    own_project_field_status () {
      return {
        project_name: {
          rw: this.readWrite,
          hidden: false
        },
        project_descr: {
          rw: this.readWrite,
          hidden: false
        },
        project_type: {
          rw: this.readWrite,
          hidden: false
        },
        project_lang: {
          rw: this.readWrite,
          hidden: false
        }
      }
    },
    user_field_status () {
      return {
        language: {
          rw: this.readWrite,
          hidden: true // not needed for registration
        },
        year: {
          rw: this.readWrite,
          hidden: false
        },
        month: {
          rw: this.readWrite,
          hidden: false
        },
        email: {
          rw: this.readWrite,
          hidden: false
        },
        firstname: {
          rw: this.readWrite,
          hidden: false
        },
        lastname: {
          rw: this.readWrite,
          hidden: false
        },
        sex: {
          rw: this.readWrite,
          hidden: false
        },
        address: {
          postalcode: {
            rw: this.readWrite,
            hidden: false
          },
          street: {
            rw: this.readWrite,
            hidden: false
          },
          house_number: {
            rw: this.readWrite,
            hidden: false
          },
          bus_number: {
            rw: this.readWrite,
            hidden: false
          },
          municipality_name: {
            rw: this.readWrite,
            hidden: false
          }
        },
        gsm: {
          rw: this.readWrite,
          hidden: false
        },
        via: {
          rw: this.readWrite,
          hidden: false
        },
        medical: {
          rw: this.readWrite,
          hidden: false
        },
        email_guardian: {
          rw: this.readWrite,
          hidden: false
        },
        gsm_guardian: {
          rw: this.readWrite,
          hidden: false
        },
        t_size: {
          rw: this.readWrite,
          hidden: false
        },
        general_questions: {
          rw: this.readWrite,
          hidden: false
        }
      }
    },
    own_project: {
      async set (value) {
        await this.$store.dispatch('registration/own_project', Object.assign({}, value))
      },
      get () {
        return this.$store.state.registration.own_project
      }
    },
    other_project: {
      async set (value) {
        await this.$store.dispatch('registration/other_project', Object.assign({}, value))
      },
      get () {
        return this.$store.state.registration.other_project
      }
    },
    user: {
      async set (value) {
        await this.$store.dispatch('registration/user', Object.assign({}, value))
      },
      get () {
        return this.$store.state.registration.user
      }
    },
    mandatory_approvals: {
      async set (value) {
        const u = Object.assign({}, this.$store.state.registration.user)
        u.mandatory_approvals = value
        await this.$store.dispatch('registration/user', u)
      },
      get () {
        const u = this.$store.state.registration.user
        return u.mandatory_approvals
      }
    },
    residence: {
      set (value) {
        this.$store.dispatch('registration/residence', value)
      },
      get () {
        return this.$store.state.registration.residence
      }
    }
  },
  mounted () {
    this.$nextTick(() => {
      // redirect to user page (token in GET string)
      const token = this.$route.query.token
      if (token) {
        this.is_own_project = 'other'
        this.other_project.project_code = token
        this.$router.replace(this.localePath('registration'))
      }
    })
  },
  methods: {
    async onSubmit (evt) {
      this.loading = true

      const registration = { project: {} }
      if (this.is_own_project === 'own') {
        registration.project.own_project = this.own_project
      } else {
        registration.project.other_project = this.other_project
      }
      const user = Object.assign({}, this.user)
      user.language = this.$i18n.locale

      registration.user = user
      await this.$services.registration.post(registration)
      this.onReset(evt)

      this.loading = false
      window.scrollTo(0, 0)
    },
    onReset (evt) {
      // copy default state to vuex store
      this.mandatory_approvals = this.$refs.mandatoryQuestions.$options.props.responses.default()
      this.user = this.$refs.user.$options.props.user.default()
      if (this.$refs.otherProject) {
        this.other_project = this.$refs.otherProject.$options.props.project.default()
      }
      if (this.$refs.ownProject) {
        this.own_project = this.$refs.ownProject.$options.props.project.default()
      }
      this.$nextTick(() => {
        this.$refs.observer.reset()
      })
    }
  }
}
</script>
<style></style>
