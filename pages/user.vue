<template>
  <b-row>
    <b-col>
      <h1>{{ $t("titleUser") }}</h1>
      <global-notification />
      <ValidationObserver ref="observer" v-slot="{ passes }">
        <b-form @submit.prevent="passes(onSubmit)" @reset.prevent="onReset">
          <user v-model="user" read-only />
          <h2>{{ $t("Mandatory Approvals") }}</h2>
          <mandatory-questions v-if="user" v-model="user.mandatory_approvals" />
          <ActionBarProject
            v-if="user"
            update
            :del="user.delete_possible"
            reset
            @deleteProject="onDelete"
          />
        </b-form>
      </ValidationObserver>
    </b-col>
  </b-row>
</template>
<script>
import { ValidationObserver } from 'vee-validate'

export default {
  middleware: 'authenticated',
  components: {
    ValidationObserver
  },
  async asyncData ({ store, query, app, redirect, route }) {
    const user = await app.$services.userinfo.get()
    return {
      user
    }
  },
  data () {
    return {
      user: {}
    }
  },
  methods: {
    async onSubmit (evt) {
      await this.$services.userinfo.patch(this.user)
      window.scrollTo(0, 0)
    },
    async onReset (evt) {
      this.user = await this.$services.userinfo.get()
    },
    onDeleteInfo (evt) {
      this.deleteInfo = true
    },
    async onDelete (evt) {
      await this.$services.userinfo.delete()
      await this.logout()
      this.$router.push(this.localePath('login'))
    },
    async logout () {
      await this.$services.logout.post()
      await this.$store.dispatch('auth/logout')
    }
  }
}
</script>

<style>
</style>
