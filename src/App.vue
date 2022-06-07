<template>
  <v-app>
    <v-main class="mt-12">
        <v-container>
          <v-row justify="space-between">
            <v-col>
              <v-card-header-text>Total comments: {{ comments.length }}</v-card-header-text>
            </v-col>
            <v-col align="end">
              <v-btn
                  @click="TEST"
                  class="mr-4"
                  size="small"
                  icon="mdi-plus"
                  color="primary"
              ></v-btn>
            </v-col>
          </v-row>
        </v-container>
        <v-container>
          <v-row justify="center">
            <v-col cols="12">
        <v-table>
          <thead>
          <tr>
            <th class="text-center">
              Name
            </th>
            <th class="text-center">
              Email
            </th>
            <th class="text-center">
              Comment
            </th>
            <th class="text-center">
              Delete
            </th>
          </tr>
          </thead>
          <draggable
              v-model="paginate"
              tag="tbody"
              item-key="id"
          >
            <template #item="{ element }">
              <tr style="cursor: pointer" @dblclick="editComment(element)">
                <td>{{ element.name }}</td>
                <td>{{ element.email }}</td>
                <td>{{ element.body }}</td>
                <td>
                  <v-btn
                      @click="removeComment(element.id)"
                      size="small"
                      icon="mdi-delete"
                      color="error"
                  ></v-btn>
                </td>
              </tr>
            </template>
          </draggable>
        </v-table>
              <v-container class="max-width">
                <v-pagination
                    :start="start"
                    v-model="page"
                    class="my-4"
                    :length="counter"
                ></v-pagination>
              </v-container>
            </v-col>
          </v-row>
        </v-container>
      <v-row justify="center">
        <v-dialog
            width="420"
            min-width="320"
            v-model="dialog"
            persistent
        >
          <v-card class="v-col-12">
            <v-form
                v-model="valid"
                lazy-validation
                ref="form"
                style="width: 420px">
              <v-card-title>
                <span class="text-h5">Add some data</span>
              </v-card-title>
              <v-card-text>
                <v-container>
                  <v-row>
                    <v-col cols="12">
                      <v-text-field
                          :rules="nameRules"
                          v-model="name"
                          label="Name"
                          :counter="50"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12">
                      <v-text-field
                          :rules="emailRules"
                          v-model="email"
                          label="Email"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12">
                      <v-textarea
                          :rules="commentRules"
                          v-model="comment"
                          label="Comment"
                          :counter="200"
                      ></v-textarea>
                    </v-col>
                  </v-row>
                </v-container>
              </v-card-text>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn
                    color="blue-darken-1"
                    text
                    @click="closeModal"
                >
                  Close
                </v-btn>
                <v-btn
                    :disabled="!valid"
                    :color="!valid ? 'disabled' : 'blue-darken-1'"
                    text
                    @click="saveData"
                >
                  Save
                </v-btn>
              </v-card-actions>
            </v-form>
          </v-card>
        </v-dialog>
      </v-row>
    </v-main>
  </v-app>
</template>

<script>
import draggable from 'vuedraggable';

export default {
  name: 'App',
  data: () => ({
    start: 1,
    page: 1,
    step: 10,
    id: null,
    valid: true,
    name: '',
    email: '',
    comment: '',
    counter: '',
    dialog: false,
    comments: [],
    editElement: '',
    sortedComments: [],
    nameRules: [v => !!v || 'Name is required', v => v.length >= 3 || 'Name must be less than 3 characters',
      v => v.length <= 50 || 'Name must be less than 50 characters'],
    emailRules: [v => !!v || 'E-mail is required', v => /.+@.+/.test(v) || 'E-mail must be valid'],
    commentRules: [v => !!v || 'Comment is required', v => v.length <= 200 || 'Name must be less than 200 characters']
  }),
  components: {
    draggable,
  },
  computed: {
    startIndex() {
      return (this.page - 1) * this.step;
    },

    endIndex() {
      return this.page * this.step;
    },
    paginate: {
      get() {
        return this.comments.slice(this.startIndex, this.endIndex);
      },
      set(newValue) {
        this.comments.splice(this.startIndex, this.step, ...newValue)
        localStorage.setItem('comments', JSON.stringify(this.comments))
      }
    }
  },
  methods: {
    closeModal() {
      this.name = '';
      this.email = '';
      this.comment = '';
      this.dialog = false;
    },
    editComment(element) {
      this.id = element.id;
      this.name = element.name;
      this.email = element.email;
      this.comment = element.body;
      this.dialog = true;
      this.$nextTick(() => this.$refs.form.validate())
    },
    saveData() {
      const comment = {
        id: this.id || Date.now(),
        name: this.name,
        email: this.email,
        body: this.comment
      }
      if (this.id) {
        const index = this.comments.findIndex((item) => item.id === this.id)
        this.comments.splice(index, 1, comment)
      } else {
        this.comments.unshift(comment)
      }
      console.log(this.comments.length)
      localStorage.setItem('comments', JSON.stringify(this.comments))
      this.name = '';
      this.email = '';
      this.comment = '';
      this.dialog = false;
    },
    removeComment(id) {
      this.comments = this.comments.filter(item => item.id !== id);
      localStorage.setItem('comments', JSON.stringify(this.comments))
    },
    TEST() {
      this.name = 'ASDASDAS';
      this.email = 'ASDASDA@CVSDVS';
      this.comment = 'CSDCSCZZXCZXC';
      this.dialog = true;
      this.$nextTick(() => this.$refs.form.validate())
    }
  },
  mounted() {
    const comments = localStorage.getItem('comments');
    if (!comments) {
      fetch('https://jsonplaceholder.typicode.com/comments')
        .then(response => response.json())
        .then(comments => {
          localStorage.setItem('comments', JSON.stringify(comments))
          this.comments = comments;
          this.counter = this.comments.length / this.step;
        })
        .catch(e => console.log(e))
    } else {
      this.comments = JSON.parse(comments);
      this.counter = (this.comments.length / this.step).toFixed();
      this.page = +localStorage.getItem('page') || 1
    }
  },
  watch: {
    'comments.length'(newValue, oldValue) {
      if(oldValue === 0) return;
      if (newValue > oldValue && (newValue % this.step) === 1) {
        ++this.counter;
      }
      if (newValue < oldValue && (newValue % this.step) === 0) {
        --this.counter;
        this.page = this.page - 1;
      }
    },
    page(newValue) {
      localStorage.setItem('page', newValue)
    }
  }
}
</script>
