<template>
  <v-container grid-list-md>
    <vue-headful title="Public Ideas | Idea Re-Vue" />

    <v-flex row class="item align-center justify-center">
      <v-tooltip bottom>
        <template v-if="Ideas" v-slot:activator="{ on }">
          <v-btn @click="SortByComp" small v-on="on" fab depressed>
            <v-icon>filter_list</v-icon>
          </v-btn>
        </template>
        <span>Sort by Completed</span>
      </v-tooltip>

      <v-tooltip bottom>
        <template v-slot:activator="{ on }">
          <v-btn v-if="Ideas" @click="SortByDate" small v-on="on" fab depressed>
            <v-icon>calendar_today</v-icon>
          </v-btn>
        </template>
        <span>Sort by Date</span>
      </v-tooltip>

      <v-tooltip v-if="Ideas" bottom>
        <template v-slot:activator="{ on }">
          <v-btn :loading="Loading" @click="GetIdeas" small v-on="on" fab depressed>
            <v-icon>refresh</v-icon>
          </v-btn>
        </template>
        <span>Refresh</span>
      </v-tooltip>
    </v-flex>

    <v-layout v-if="!Ideas" class="align-center justify-center" justify-center row align-center>
      <v-flex
        align="center"
        justify="center"
        class="align-center justify-center"
        v-for="x in 3"
        :key="x"
        xs12
        md3
      >
        <v-card hover width="320" class="mx-auto">
          <vue-content-loading primary="#f3f3f3" secondary="#ecebeb" :width="300" :height="130">
            <rect x="6.72" y="8" rx="4" ry="4" width="243.73" height="13.02" />
            <rect x="361" y="95" rx="3" ry="3" width="141.95" height="10.02" />
            <rect x="10" y="36" rx="3" ry="3" width="270" height="6" />
            <rect x="10" y="53" rx="3" ry="3" width="270" height="6" />
            <rect x="5" y="99" rx="3" ry="3" width="230" height="6.84" />
            <rect x="10" y="70" rx="3" ry="3" width="270" height="6" />
            <rect x="5" y="115" rx="3" ry="3" width="180" height="6" />
            <circle cx="284.48" cy="110.92" r="12.25" />
            <circle cx="283.48" cy="14.92" r="12.25" />
          </vue-content-loading>
        </v-card>
      </v-flex>
    </v-layout>

    <v-layout
      v-masonry
      transition-duration="1s"
      item-selector=".item"
      class="masonry-container"
      justify-center
      row
      align-center
    >
      <v-flex
        align="center"
        justify="center"
        class="item align-center justify-center"
        v-masonry-tile
        v-for="Idea in Ideas"
        :key="Idea.DocID"
        xs12
        md3
      >
        <v-card hover width="320" class="mx-auto">
          <v-card-Title class="title">
            {{ Idea.Title }}
            <v-spacer></v-spacer>
            <v-tooltip right>
              <template v-slot:activator="{ on }">
                <v-icon v-on="on" v-if="Idea.Completed" class="green--text" small>done_all</v-icon>
                <v-icon v-on="on" v-else class="orange--text text--lighten-2" small>update</v-icon>
              </template>
              <span>{{ Idea.Completed ? "Completed" : "Pending" }}</span>
            </v-tooltip>
          </v-card-Title>
          <v-card-text>
            <div class="grey--text text--darken-3 mb-1 subtitle-1">By {{Idea.UserName}}</div>
            {{ Idea.Description }}
          </v-card-text>

          <v-card-actions class="caption grey--text text--darken-3">
            Added on: {{ Idea.Added }}
            <br />
            Visibility: {{ Idea.Visibility }}
          </v-card-actions>
        </v-card>
      </v-flex>
    </v-layout>
  </v-container>
</template>
<script>
import firebase from '../firebase'
import VueContentLoading from 'vue-content-loading'
import moment from 'moment'

export default {
  components: {
    VueContentLoading
  },

  data() {
    return {
      Ideas: null,
      Loading: false
    }
  },
  methods: {
    GetIdeas() {
      let self = this
      self.Loading = true
      firebase.firebase
        .firestore()
        .collection('Ideas')
        .where('Visibility', '==', 'Public')
        .get()
        .then(function (querySnapshot) {


          var IdeasWithUsersName = []


          querySnapshot.forEach(function (doc) {
            var DocData = doc.data()
            DocData.DocID = doc.id

            let UserID = doc.data().AddedBy

            const User = firebase.firebase.firestore().collection('Users').doc(UserID)
            User.get().then(UserDetails => {
              DocData.UserName = UserDetails.data().Name
              IdeasWithUsersName.push(DocData)
              self.Ideas = IdeasWithUsersName
            })
          })
          self.Loading = false
        })
        .catch(function (error) {
          console.log('Error getting documents: ', error)
        })
    },

    SortByComp() {
      let UpdatedIdeas = this.Ideas.sort((a, b) => {
        if (a.Completed > b.Completed) {
          return -1
        }
      })

      this.Ideas = []

      setTimeout(() => {
        this.Ideas = UpdatedIdeas
      }, 0.1)
    },
    SortByDate() {

      let UpdatedIdeas = this.Ideas.sort((A, B) => {
        var C = new Date(moment(A.Added, 'Do MMMM YYYY').format('YYYY-MM-DD'))
        var D = new Date(moment(B.Added, 'Do MMMM YYYY').format('YYYY-MM-DD'))
        return D - C
      })

      this.Ideas = []

      setTimeout(() => {
        this.Ideas = UpdatedIdeas
      }, 0.1)
    },

  },
  created() {
    this.GetIdeas()
  }
}
</script>

<style>
v-card {
  color: aqua;
}
</style>
