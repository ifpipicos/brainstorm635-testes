<template>
  <b-container fluid>
    <br />
    <b-row align-v="end">
      <b-col
        class="brain-card align-items-center justify-content-center ml-auto mr-auto"
        md="7">
        <b-card class="text-center pr-5 pl-5 pb-5 pt-0 mt-3">
          <b-row>
            <b-col>
              <h4 class="page-tittle mb-5 mt-4">Brainstorm</h4>
            </b-col>
          </b-row>
          <b-row class="text-center mt-4">
            <b-col md="6">
              <b-button
                @click="createNewBrainstorm()"
                class="new-brain-button mt-5"
                variant="outline-info">
                New Brainstorm
                <i class="fas fa-plus"></i>
              </b-button>
            </b-col>
            <b-col md="6">
              <b-form @submit.prevent="joinWithCode(coderoom)">
                <b-row class="mt-5">
                  <b-col md="12" class="d-flex">
                    <b-form-input
                      class="input-for-code"
                      v-model="coderoom"
                      placeholder="Join with code"
                      maxlength="10"
                    >
                    </b-form-input>
                    <b-button type="submit" variant="info">
                      Join
                    </b-button>
                  </b-col>
                </b-row>
              </b-form>
            </b-col>
          </b-row>
        </b-card>
        <!--         <b-modal
          centered
          id="modal"
          title="Informe seu nome">
          <template v-slot:modal-footer="{ hide }">
            <form  @submit.prevent="checkForm()">
              <b-form-group
                label="Nome"
                label-for="name-input"
                label-class="required"
                invalid-feedback="Name is required">
                <b-form-input
                  placeholder="Informe seu nome"
                  id="name-input"
                  v-model="name"
                  @change="$v.name.$touch()"
                  required>
                </b-form-input>
                <span
                  class="warning-input-forms"
                  v-if="$v.name.$error">
                  O campo nome é obrigatório ter no mínimo 4 caracteres.
                </span>
              </b-form-group>
              <br>
              <b-row>
                <b-col class="ml-auto mr-auto" sm="6">
                  <b-button size="sm" variant="outline-danger" @click="hide()">Cancelar</b-button>
                  <b-button class="ml-4 pl-4 pr-4" type="submit" size="sm" variant="outline-success">Ok</b-button>
                </b-col>
              </b-row>
             </form>
          </template>
        </b-modal> -->
      </b-col>
    </b-row>
  </b-container>
</template>

<script>
import firebase from 'firebase/app'
import Swal from 'sweetalert2'
/* import { EventBus } from '@/eventBus' */

export default {
  data () {
    return {
      idBrainstorm: '',
      coderoom: ''
    }
  },

  mounted () {
    const currentUser = this.$firebase.auth().currentUser
    console.log(currentUser.photoURL)
  },

  methods: {
    createNewBrainstorm () {
      /* EventBus.$emit('updateList') */
      const id = this.codeGenerator(6)
      const currentUser = this.$firebase.auth().currentUser
      const user = {
        displayName: currentUser.displayName,
        photoURL: currentUser.photoURL,
        uid: currentUser.uid
      }
      this.$firebase
        .firestore()
        .collection('brainstorms')
        .doc(id.toString())
        .set({
          running: false,
          leader: user.uid,
          description: 'Brainstorm description',
          listGuests: [user],
          currentRound: 0,
          timestamp: firebase.firestore.FieldValue.serverTimestamp()
        })
        .then(() => {
          this.$router.push({ name: 'brainstorm', params: { id: id } })
        })
        .catch(error => console.error(error))
    },

    codeGenerator (length) {
      let result = ''
      const characters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789'
      const charactersLength = characters.length
      for (let i = 0; i < length; i++) {
        result += characters.charAt(
          Math.floor(Math.random() * charactersLength)
        )
      }
      return result
    },

    async joinWithCode (coderoom) {
      coderoom = coderoom.trim()
      coderoom = coderoom.toUpperCase()
      if (coderoom) {
        const brainstorm = this.$firebase
          .firestore()
          .collection('brainstorms')
          .doc(coderoom)

        brainstorm
          .get()
          .then(async doc => {
            if (doc.data().listGuests.length < 6) {
              const userGuest = {}
              userGuest.uid = this.$firebase.auth().currentUser.uid
              userGuest.displayName = this.$firebase.auth().currentUser.displayName
              userGuest.photoURL = this.$firebase.auth().currentUser.photoURL
              await brainstorm.update({
                timestamp: firebase.firestore.FieldValue.serverTimestamp(),
                listGuests: firebase.firestore.FieldValue.arrayUnion(userGuest)
              })
              console.log('Você entrou!!!')
              this.$router.push({
                name: 'brainstorm',
                params: { id: coderoom }
              })
            } else {
              this.fullBrainstorm()
              console.log('Já está lotado!')
            }
          })
          .catch((error) => {
            console.log('Error getting cached document:', error)
            this.nonExistentBrainstorm()
          })
      }
    },

    fullBrainstorm () {
      Swal.fire({
        title: 'The Brainstorm is full!',
        text:
          'You are trying to access the Brainstorm, but it´s reached the number maximum of guests!',
        icon: 'error',
        confirmButtonText: 'OK',
        timer: 4000
      })
    },

    nonExistentBrainstorm () {
      Swal.fire({
        title: 'Brainstorm not existent!',
        text:
          'You are trying to access a non-existent Brainstorm, try another code!',
        icon: 'error',
        confirmButtonText: 'OK',
        timer: 4000
      })
    }
  }
}
</script>

<style lang="css">
* {
  box-sizing: border-box;
}

.input-for-code {
  padding: 1.2rem 0.5rem !important;
}

/* Changes for Modal */
.modal-title {
  font-weight: bold;
  text-transform: uppercase;
}

.modal-body {
  padding: 0 !important;
}

.modal-footer {
  border-top: none !important;
  display: inline-block !important;
  padding: 0 0 0.75rem 0 !important;
}

.modal-footer > * {
  margin: 1rem 1.5rem !important;
}

.modal-content {
  box-shadow: 0px 5px 10px 0px rgba(230, 230, 230, 0.2) !important;
}

/* Chancge colors for class button .btn-secondary */
/* .btn-outline-danger {
  background-color: transparent !important;
  border-color: #ce242a !important;
  color: #bd2130 !important;
}

.btn-outline-danger:hover {
  color: #fff !important;
  background-color: #bd2130 !important;
  border-color: #a94442 !important;
}

.btn-outline-danger:focus {
  color: #fff !important;
  background-color: #a94442 !important;
  border-color: #a94442 !important;
  box-shadow: 0 0 0 0.2rem rgba(225,83,97,.5) !important;
} */
</style>
