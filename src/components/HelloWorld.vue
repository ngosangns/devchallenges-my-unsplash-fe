<template>
  <div class="hello">
    <!-- Navbar -->
    <vs-navbar id="top" center-collapsed>
      <template #left>
        <!-- Home button -->
        <vs-button
          gradient
          :active="false"
          @click="resetHome"
        >
          My Unsplash
        </vs-button>

        <!-- Search button -->
        <vs-navbar-item :active="false">
          <vs-input
            id="search-input"
            state="dark"
            type="search"
            placeholder="Search"
            icon-before
            v-on:keyup.enter="search"
            v-model="inSearch"
          >
            <!-- Icon -->
            <template #icon>
              <box-icon name='search'></box-icon>
            </template>
          </vs-input>
        </vs-navbar-item>
      </template>
      <template #right>
        <!-- Add photo button -->
        <vs-button id="add-photo-button" flat :active="true"
          v-on:click="addPhoto()">Add a photo</vs-button>
      </template>
    </vs-navbar>

    <!-- Main -->
    <div id="main">
      <div id="search-results-title" v-if="enableSearchTitle.value">
        <h5>Results for "{{searchText}}"</h5>
      </div>
      <Masonry id="masonry"/>
    </div>

    <AddingPhotoDialog id="dialog" :enableDialog="enableAddingPhotoDialog"/>

    <footer id="footer">
      <span>©2021 ngosangns -
        <a style="color: blue; text-decoration: none;" href="https://devchallenges.io/">devChallenges</a></span>
    </footer>
  </div>
</template>

<script>
import Masonry from "./Masonry/Masonry.vue"
import AddingPhotoDialog from "./AddingPhotoDialog/AddingPhotoDialog.vue"

import axios from "axios"

export default {
  name: 'HelloWorld',
  data: () => ({
    state: "off",
    enableAddingPhotoDialog: {
      value: false
    },
    enableDeletingPhotoDialog: {
      value: false
    },
    // Using for state
    enableSearchTitle: {
      value: false,
    },
    inSearch: '', // Using for search input
    searchText: '', // Using for binding
  }),
  components: {Masonry, AddingPhotoDialog},
  props: {
    msg: String
  },
  methods: {
    clearState: function() {
      let states = [
        this.enableAddingPhotoDialog,
        this.enableDeletingPhotoDialog,
        this.enableSearchTitle,
      ]
      for(let state of states) {
        state.value = false
      }
    },
    clearInput: function() {
      this.inSearch = ''
      this.searchText = ''
    },
    resetHome: function() {
      this.clearState()
      this.clearInput()
      this.getData()
    },
    getData: function() {
      // Open loading
      this.states = "loading"
      const loading = this.$vs.loading({
        target: document.getElementById("masonry"),
        background: "transparent",
      })
      // Get data
      axios.get(process.env.VUE_APP_GET)
        .then(res => {
          if(res.data.status) {
            this.$store.commit("setImages", res.data.message.reverse())
          } else {
            this.$vs.notification({
              color: "danger",
              duration: 3000,
              title: "",
              text: "Error while requesting",
            })
          }
        })
        .catch((err) => {
          console.log(err)
          this.$vs.notification({
            color: "danger",
            duration: 3000,
            title: "",
            text: "Error while requesting",
          })
        })
        .finally(() => {
          loading.close()
          this.state = "off"
        })
    },
    addPhoto: function() {
      if(this.state == "loading") return
      this.enableAddingPhotoDialog.value = true
    },
    search(e) {
      let searchText = e.target.value

      // Validate
      if(!searchText.length) return

      // Open loading
      this.state = "loading"
      const loading = this.$vs.loading({
        target: document.getElementById("masonry"),
        background: "transparent",
      })
      // Search request
      axios.get(process.env.VUE_APP_SEARCH+'?query='+searchText)
        .then(res => {
          if(res.data.status) {
            this.$store.commit("setImages", res.data.message.reverse())
            this.enableSearchTitle.value = true
            this.searchText = searchText
          } else {
            this.$vs.notification({
                color: "danger",
                duration: 3000,
                title: '',
                text: res.data.message
              })
          }
        })
        .catch((err) => {
          this.$vs.notification({
            color: "danger",
            duration: 3000,
            title: "",
            text: "Error while requesting",
          })
          throw err
        })
        .finally(() => {
          loading.close()
          this.state = "off"
        })
    }
  },
  mounted: function() {
    this.getData()
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
#top {
  position: relative;
}

#main {
  position: relative;
  margin: auto;
  margin-top: 1rem;
  width: 96vw;
  min-height: 80vh;
  #search-results-title {

    h5 {
      display: block;
      text-align: left;
      margin: 1rem 0;
    }
  }
}

#footer {
  margin-top: 2rem;
  margin-left: .5rem;
  span {
    display: block;
    text-align: left;
  }
}
</style>
