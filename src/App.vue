<template>
  <v-app>
    <v-main class="mt-12">
        <v-container>
          <v-row justify="end">
            <v-col align="right">
              <v-btn
                  @click="addComment()"
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
              <tr style="cursor: pointer">
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
                    :length="(comments.length / step).toFixed()"
                ></v-pagination>
              </v-container>
            </v-col>
          </v-row>
        </v-container>
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
    comments: [],
    sortedComments: []
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
    addComment() {
      this.comments.push({
        id: this.comments.length + 1,
        name: 'Den',
        email: 'bar@mail.com',
        body: 'Some text',
        userId: 1
      })
      localStorage.setItem('comments', JSON.stringify(this.comments));
    },
    removeComment(id) {
      console.log('Remove')
      this.comments = this.comments.filter(item => item.id !== id);
      localStorage.setItem('comments', JSON.stringify(this.comments))
    }
  },
  mounted() {
    const comments = localStorage.getItem('comments');
    if (!comments) {
      fetch('https://jsonplaceholder.typicode.com/comments')
        .then(response => response.json())
        .then(comments => {
          localStorage.setItem('comments', JSON.stringify(comments))
          this.comments = comments
        })
        .catch(e => console.log(e))
    } else {
      this.comments = JSON.parse(comments);
    }
  },
}
</script>
