<template>
    <vs-dialog :not-close="true" v-model="enableDialog.value" prevent-close>
      <template #header>
        <h4>Delete photo</h4>
      </template>

      <div id="dlg-form" v-if="photo.value">
        <vs-alert danger>
          Type the password for deleting the photo <b>#{{photo.value.data.label}}</b><br>
          (password is <b>123</b>)
        </vs-alert>

        <!-- Input -->
        <vs-input class="dlg-input"
          state="dark"
          type="password"
          label="Password"
          v-model="password"
        >
          <!-- Message -->
          <template #message-danger>
            {{message}}
          </template>
        </vs-input>
      </div>
      <div>
        
      </div>

      <template #footer>
        <div id="dlg-footer">
          <vs-button dark flat :active="false" v-on:click="hideDialog">
            Cancel
          </vs-button>
          <vs-button danger flat :active="true" v-on:click="onSubmit">
            Submit
          </vs-button>
        </div>
      </template>
    </vs-dialog>
</template>
<script>
import axios from "axios"

export default {
    name: "DeletingPhotoDialog",
    props: ["photo", "enableDialog"],
    data: () => ({
        password: '',
        message: '',
    }),
    methods: {
      clearForm: function() {
        this.password = ""
        this.message = ""
      },
      hideDialog: function() {
        this.clearForm()
        this.enableDialog.value = false
        this.photo.value = null
      },
      onSubmit: function() {
        // Validate
        if(!this.password.length) {
          this.message = "The field mustn't be null"
          return
        } else this.message = ""

        let loading = this.createLoading()
        // Send
        axios.get(process.env.VUE_APP_DELETE + "?id=" + this.photo.value._id, {
            headers: {'Password': this.password}
          })
          .then(res => {
            if(res.data.status) {
              // Notification
              this.$vs.notification({
                duration: 3000,
                color: "danger",
                title: "",
                text: "Deleted!"
              })
              this.$store.commit("removeImage", this.photo.value)
            } else {
              this.$vs.notification({
                color: "danger",
                duration: 3000,
                title: "",
                text: res.data.message,
              })
            }
          })
          .catch(err => {
            this.$vs.notification({
              color: "danger",
              duration: 3000,
              title: "",
              text: "Error while requesting",
            })
            throw err
          })
          .finally(() => {
            this.clearForm()
            this.enableDialog.value = false
            loading.close()
          })
      },
      createLoading: function() {
        const loading = this.$vs.loading()
        // loading.close()
        return loading
      },
    }
}
</script>
<style lang="scss">
// Form css is inheritance from AddingPhotoDialog.vue
</style>