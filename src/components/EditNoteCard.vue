<template>
  <transition name="zoom">
    <div
      v-if="dataReady"
      class="row justify-content-center"
    >
      <div class="col col-12 col-md-6">
        <ul
          v-if="validationErrors.length"
          class="alert alert-danger form-errors"
        >
          <li
            v-for="(validationError, index) in validationErrors"
            v-bind:key="index"
          >{{validationError}}</li>
        </ul>
        <ul
          class="nav nav-pills mb-3"
          id="pills-tab"
          role="tablist"
        >
          <li class="nav-item">
            <a
              class="nav-link active"
              id="pills-form-tab"
              data-toggle="pill"
              href="#pills-form"
              role="tab"
              aria-controls="pills-form"
              aria-selected="true"
            >Edit note</a>
          </li>
          <li class="nav-item">
            <a
              class="nav-link"
              id="pills-preview-tab"
              data-toggle="pill"
              href="#pills-preview"
              role="tab"
              aria-controls="pills-preview"
              aria-selected="false"
            >Note preview</a>
          </li>
        </ul>

        <!-- <div v-if="dataReady" class="row"> -->
        <div
          class="tab-content"
          id="pills-tabContent"
        >
          <div
            class="tab-pane fade show active"
            id="pills-form"
            role="tabpanel"
            aria-labelledby="pills-form-tab"
          >
            <div class="card shadow-sm mt-4">
              <div class="card-body">
                <form>
                  <div class="form-group">
                    <input
                      class="form-control"
                      v-model="note.title"
                      type="text"
                      placeholder="Note Title"
                    >
                  </div>
                  <div class="form-group">
                    <input
                      class="form-control"
                      v-model="note.body"
                      type="text"
                      placeholder="Note Body"
                    >
                  </div>
                  <div class="form-group clearfix">
                    <button
                      type="button"
                      class="btn btn-sm float-right ml-1"
                      v-bind:class="note.theme"
                    >Current Theme</button>
                    <div class="dropdown float-right">
                      <button
                        type="button"
                        class="btn btn-sm btn-light dropdown-toggle"
                        data-toggle="dropdown"
                      >
                        Select Theme
                      </button>
                      <div class="dropdown-menu dropdown-menu-right fade">
                        <theme-picker
                          v-bind:themes="themes"
                          v-bind:currentTheme="note.theme"
                          v-on:applytheme="applyTheme"
                        />
                        <input
                          class="form-control"
                          v-model="note.theme"
                          type="hidden"
                        >
                      </div>
                    </div>
                  </div>
                  <div class="form-group">
                    <p class="card-text text-right">
                      <button
                        type="button"
                        class="btn btn-primary"
                        v-on:click="updateNote"
                      >Save</button>&nbsp;
                      <button
                        type="button"
                        class="btn btn-dark"
                        v-on:click="cancelEditNote"
                      >Cancel</button>
                    </p>
                  </div>

                </form>
              </div>
            </div>
          </div>
          <div
            class="tab-pane fade"
            id="pills-preview"
            role="tabpanel"
            aria-labelledby="pills-preview-tab"
          >
            <div
              class="card shadow-sm mt-4"
              v-bind:class="note.theme"
            >
              <div class="card-body">
                <h5 class="card-title">
                  {{note.title}}
                </h5>
                <p class="card-text">{{note.body}}</p>
                <p><small>Created: {{note.created_at}} | Last Updated: {{note.updated_at}}</small></p>
              </div>
            </div>
          </div>
          <div class="col mt-4">
            <home-link></home-link>
          </div>
        </div>
      </div>
    </div>

  </transition>
</template>

<script>
import { mapState } from "vuex";
import ThemePicker from "./ThemePicker.vue";
import HomeLink from "./HomeLink.vue";
import formErrorHandler from '../mixins/formErrorHandler'
export default {
  mixins: [formErrorHandler],
  components: {
    "theme-picker": ThemePicker,
    "home-link": HomeLink
  },
  computed: {
    ...mapState({
      dataReady: "dataReady",
      themes: "themes"
    }),
    loggedIn: function () {
      return this.$store.getters.loggedIn
    },
    getUserId: function () {
      return this.$store.getters.getUserId
    }
  },
  data () {
    return {
      id: this.$route.params.id,
      note: {},
      validationErrors: [

      ]
    };
  },
  methods: {
    fetchNote: function (id) {
      this.$store.dispatch("fetchNote", id).then(response => {
        this.note = response.data.data;
        const noteId = parseInt(this.note.user_id)
        if (noteId !== this.getUserId) {
          this.$router.push({ name: 'viewNote', params: this.id })
        } else {
          this.$store.dispatch('dataReady', true)
        }
      })
    },
    applyTheme: function (theme) {
      this.note.theme = theme;
    },
    cancelEditNote: function () {
      this.$store.dispatch('dataReady', false)
      this.$store.dispatch("cancelEditNote");
      this.$router.push({ name: "viewNote", params: this.id });
    },
    updateNote: function () {
      this.validationErrors = []

      if (!this.note.title) {
        this.validationErrors.push("Title is required.")
      }

      if (!this.note.body) {
        this.validationErrors.push("Body is required.")
      }

      if (!this.note.theme) {
        this.validationErrors.push("Theme is required.")
      }


      if (!this.validationErrors.length) {
        this.$store.dispatch("updateNote", this.note)
          .then(() => {
            this.$store.dispatch("notify", {
              type: "success",
              message: "Note changed successfully!"
            });
          })
          .catch((error) => {

            this.errorHandler(error, this.validationErrors)

          });
      }

    }
  },
  created () {
    this.$store.dispatch("clearSelectedNote");
  },
  mounted () {
    if (this.loggedIn) {
      this.fetchNote(this.id);
    } else {
      this.$router.push({ name: 'login' })
    }

  }
};
</script>

<style scoped>
.form-errors li {
  margin-left: 10px;
}
</style>
