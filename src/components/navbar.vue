<template>
  <b-navbar fixed="top" toggleable="lg" type="light">
    <b-navbar-brand>
      <b-link
        class="sidebar-nav
          navbar-brand ml-1"
        to="/"
      >
        <b-img
          class="ml-3 mt-4"
          width="190"
          height="20"
          :src="require('../../public/img/brainstorm635.png')"
        />
      </b-link>
    </b-navbar-brand>
    <b-navbar-toggle target="nav-collapse"></b-navbar-toggle>
    <b-collapse id="nav-collapse" is-nav>
      <b-navbar-nav class="ml-auto">
        <!-- <b-spinner
          v-if="load"
          style="width: 1.5rem; height: 1.5rem;"
          type="grow"
          variant="info"
          label="Spinning">
        </b-spinner> -->
        <div v-if="verifyLocalStorage" class="d-flex mr-3">
          <b-nav-item right>
            <b-avatar circle :src="user.photoURL" alt="Foto do usuário">
            </b-avatar>
            <span class="user ml-2">{{ user.displayName }}</span>
          </b-nav-item>
          <b-nav-item-dropdown
            class="avatar d-flex justify-content-end pl-0"
            right
          >
            <b-dropdown-item href="#">Profile</b-dropdown-item>
            <b-dropdown-item @click="logout()">Sign Out</b-dropdown-item>
          </b-nav-item-dropdown>
        </div>
        <b-nav-item-dropdown text="Lang" right>
          <b-dropdown-item href="#"
            >PT
            <b-img
              class="ml-1"
              rounded="circle"
              :src="require('../../public/img/brasil.png')"
              alt="PT"
              height="17"
            >
            </b-img>
          </b-dropdown-item>
          <b-dropdown-item href="#"
            >EN
            <b-img
              class="usa"
              :src="require('../../public/img/eua.png')"
              rounded="circle"
              alt="EN"
              height="17"
            >
            </b-img>
          </b-dropdown-item>
          <b-dropdown-item href="#"
            >ES
            <b-img
              class="ml-1"
              rounded="circle"
              :src="require('../../public/img/espanha.png')"
              alt="ES"
              height="17"
            >
            </b-img>
          </b-dropdown-item>
        </b-nav-item-dropdown>
      </b-navbar-nav>
    </b-collapse>
  </b-navbar>
</template>

<script>
import { EventBus } from '@/eventBus'

export default {
  data () {
    return {
      load: false,
      verifyLocalStorage: false,
      user: {
        photoURL: '',
        displayName: ''
      }
    }
  },

  created () {
    EventBus.$on('user', () => {
      this.getLocalStorage()
      this.verifyLocalStorage = true
    })
    this.getLocalStorage()
  },

  methods: {
    logout () {
      this.verifyLocalStorage = false
      this.$firebase
        .auth()
        .signOut()
        .then(() => {
          localStorage.removeItem('currentUser')
          this.user = { photoURL: null, displayName: null }
        })
        .catch((error) => {
          console.error(error)
        })
    },

    getLocalStorage () {
      this.user = JSON.parse(localStorage.getItem('currentUser'))
      if (this.user) {
        this.verifyLocalStorage = true
      }
      if (this.user === null) {
        this.user = { photoURL: '', displayName: '' }
      }
    }
  }
}
</script>

<style lang="css">
/* Barra lateral Azul da navbar e footer */
.sidebar-nav {
  border-left: 4px solid #17a2b8;
  height: 60px !important;
}

.navbar-brand {
  padding: 0 !important;
}

.navbar {
  background-color: #fff !important;
  border-bottom: 1px solid rgb(0, 0, 0, 0.125);
  box-shadow: rgba(0, 0, 0, 0.1) 0px 1px 5px 0px;
  width: 100%;
  height: 75px !important;
  padding: 0 7rem 0 7rem !important;
}

.usa {
  margin-left: 1px;
}

span,
.user {
  font-size: 1.2rem;
}
/* Media queries for responsive nav bar */
@media only screen and (max-width: 576px) {
  .navbar {
    padding: 0 !important;
  }
}

/* .navbar-brand {
  display: inline-block;
  padding-top: 0.3125rem;
  padding-bottom: 0.3125rem;
  margin-right: 1rem;
  font-size: 1.25rem;
  line-height: inherit;
  white-space: nowrap;
} */

.navbar-expand-lg .navbar-nav .nav-link {
  padding-right: 0.5rem;
  padding-left: 0 !important;
}
</style>
