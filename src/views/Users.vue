<template>
  <v-app id="inspire" dark>
    <navigationDrawer/>
    <v-content>
      <v-container fluid fill-height text-xs-center>
        <v-layout justify-center align-center row wrap>
          <chips />
          <v-flex xs12>
            <v-btn large style="background-color:#4286f4" to="/users" id='usersButton'><v-icon>account_circle</v-icon> <span v-if="this.$store.state.site.desktop">View Users</span></v-btn>
            <v-btn large style="background-color:#4286f4" to="/guilds" id='guildsButton'><i class="fas fa-users"></i>&nbsp;<span v-if="this.$store.state.site.desktop">View Guilds</span></v-btn>
            <v-btn large style="background-color:#4286f4" to="/giveaways" id='giveawaysButton'><v-icon>card_giftcard</v-icon> <span v-if="this.$store.state.site.desktop">View Giveaways</span></v-btn>
            <v-btn large style="background-color:#4286f4" to="/referrals" id='referralsButton'><v-icon>arrow_forward</v-icon> <span v-if="this.$store.state.site.desktop">View Referrals</span></v-btn>
            <div id='usersData'>
              <p style="font-size:20px">{{ msg }}</p>
              <v-tooltip right>
                <v-btn slot="activator" @click="loadUsers();changeVisibility()" icon large>
                  <v-icon large id="usersArrow">arrow_drop_down</v-icon>
                </v-btn>
                <span id="viewUsers">Show Users</span>
              </v-tooltip>
              <div id='usersList' style="display:none">
                <table style="margin:auto" class="centered">
                  <thead>
                    <tr>
                      <th>Name</th>
                    </tr>
                  </thead>
                  <tbody v-for="user in users" :key="JSON.stringify(user.id)" style="font-weight:normal">
                    <tr style="cursor:pointer" @click="loadUserInfo(user.id)">
                      <td>
                        {{ user.name }}
                        <div :id="user.id"></div>
                      </td>
                    </tr>
                  </tbody>
                </table>
              </div>
            </div>
          </v-flex>
          <v-flex shrink>
            <v-tooltip right>
              <v-btn slot="activator" href="https://github.com/jasonhaxstuff/nightwatchgui" icon large target="_blank">
                <v-icon large>code</v-icon>
              </v-btn>
              <span>Source</span>
            </v-tooltip>
          </v-flex>
        </v-layout>
      </v-container>
    </v-content>
    <customFooter></customFooter>
  </v-app>
</template>

<script lang='ts'>
import axios from 'axios'
import Footer from '@/components/Footer.vue'
import NavigationDrawer from '@/components/NavigationDrawer.vue'
import Chips from '@/components/Chips.vue'
import Vue from 'vue'
import { apiUrl } from '@/config';

export default Vue.extend({
  name: 'Users',
  components: {
    customFooter: Footer,
    navigationDrawer: NavigationDrawer,
    chips: Chips
  },
  data () {
    return {
      msg: 'Hey, look! Users!',
      users: null,
      headers: [
        {
          text: 'Property',
          align: 'center',
          sortable: false,
          value: 'property'
        },
        { text: 'Value', value: 'setting', align: 'center' }
      ],
      api: apiUrl
    }
  },
  created () {
    const x = window.innerWidth >= 1000
    this.$store.commit('setDesktop', x)
  },
  methods: {
    loadUsers () {
      if (this.users) {
        return true
      }
      axios.get(`${this.api}/users`).then(response => {
        this.users = response.data
      })
    },
    loadUserInfo (id: string) {
      if (this.$store.state.opened.usersOpen.includes(id)) {
        const user = document.getElementById(id)

        if (!user) {
          return false
        }

        user.setAttribute('style', 'display:none')
        this.$store.commit('removeUsersOpen', id)
        return true
      }
      if (this.$store.state.opened.usersOpened.includes(id)) {
        const user = document.getElementById(id)

        if (!user) {
          return false
        }

        user.setAttribute('style', 'display:initial')
        this.$store.commit('addUsersOpen', id)
        return true
      }
      this.$store.commit('addUsersOpen', id)
      this.$store.commit('addUsersOpened', id)
      axios.get(`${this.api}/users/${id}`).then(response => {
        const div = document.getElementById(id)

        if (!div) {
          return false
        }

        div.innerHTML = `
        <hr>
        <table style="margin:auto" class="highlight centered">
        <thead>
          <tr>
            <th>Property</th>
            <th>Value</th>
          </tr>
        </thead>
        <tbody>
        <tr>
          <td>Date created</td>
          <td>${response.data.dateCreated}</td>
        </tr>
        <tr>
          <td>ID</td>
          <td>${response.data.id}</td>
        </tr>
        <tr>
          <td>Avatar</td>
          <td><img src="${response.data.avatarUrl}" height="64" width="64"></td>
        </tr>
        <tr>
          <td>Date of last message</td>
          <td>${response.data.dateLastMessage}</td>
        </tr>
        <tr>
          <td>XP</td>
          <td>${response.data.level.xp}</td>
        </tr>
        <tr>
          <td>Level</td>
          <td>${response.data.level.level}</td>
        </tr>
        <tr>
          <td>Last level up</td>
          <td>${response.data.level.timestamp}</td>
        </tr>
        <tr>
          <td>Balance</td>
          <td>${response.data.balance.balance}</td>
        </tr>
        <tr>
          <td>Net worth</td>
          <td>${response.data.balance.netWorth}</td>
        </tr>
        <tr>
          <td>Last claimed dailies</td>
          <td>${response.data.balance.dateLastClaimedDailies}</td>
        </tr>
        <tr>
          <td>Title</td>
          <td>${response.data.profile.title}</td>
        </tr>
        <tr>
          <td>Bio</td>
          <td>${response.data.profile.bio}</td>
        </tr>
        <tr>
          <td>Background</td>
          <td>${response.data.profile.background}</td>
        </tr>
        <tr>
          <td>Levels enabled</td>
          <td>${response.data.settings.levelsEnabled}</td>
        </tr>
        <tr>
          <td>DMs enabled</td>
          <td>${response.data.settings.directMessagesEnabled}</td>
        </tr>
        <tr>
          <td>DB ID</td>
          <td>${response.data.settings.id}</td>
        </tr>
        </tbody>
        </table>
        <hr>
        `
      })
    },
    changeVisibility () {
      const button = document.getElementById('viewUsers')
      const usersArrow = document.getElementById('usersArrow')
      const div = document.getElementById('usersList')

      if (!button || !div || !usersArrow) {
        return false
      }

      const x = div.offsetParent

      if (x === null) {
        div.setAttribute('style', 'display:initial')
        button.textContent = 'Hide Users'
        usersArrow.textContent = 'arrow_drop_up'
      } else {
        div.setAttribute('style', 'display:none')
        button.textContent = 'Show Users'
        usersArrow.textContent = 'arrow_drop_down'
      }
    }
  }
})
</script>
