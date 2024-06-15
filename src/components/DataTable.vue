<template>
  <div class="table-container">

    <h1>Vue & Angular-First Test</h1>
    <div class="sub-title">
        <h2>Data table</h2>
        <input class="search" v-model="searchQuery" placeholder="Search..." @input="debouncedSearch" />
    </div>

    <table class="data-table">

      <thead class="data-table-header">
        <tr>
          <th @click="sortBy('id')">ID</th>
          <th>Name</th>
          <th @click="sortBy('email')">Email</th>
          <th>Body</th>
          <th>Actions</th>
        </tr>
      </thead>

      <tbody class="data-table-body">
        <tr class="table-data" v-for="comment in paginatedComments" :key="comment.id">
          <td class="id">{{ comment.id }}</td>
          <td>
            <span v-if="!editing || editId !== comment.id">{{comment.name}}</span>
            <input v-else v-model="editComment.name" />
          </td>
          <td>
            <span v-if="!editing || editId !== comment.id">{{comment.email }}</span>
            <input v-else v-model="editComment.email" />
          </td>
          <td>
            <div class="comment-body">
                <span v-if="!editing || editId !== comment.id">{{comment.body}}</span>
                <textarea v-else rows="6" v-model="editComment.body"></textarea>
            </div>
          </td>
          <td class="action-btn">
            <button class="edit" v-if="!editing || editId !== comment.id" @click="startEdit(comment)" ><ion-icon name="create-outline"></ion-icon></button>
            <button class="save" v-if="editing && editId === comment.id" @click="saveEdit"><ion-icon name="save-outline"></ion-icon></button>
            <button class="cancel" v-if="editing && editId === comment.id" @click="cancelEdit"><ion-icon name="close-outline"></ion-icon></button>
            <button class="delete" @click="deleteComment(comment.id)"><ion-icon name="trash-sharp"></ion-icon></button>
          </td>
        </tr>
      </tbody>

    </table>

    <div v-if="loading">Loading...</div>

    <div class="paginator">
        <div>        
            <label for="rowsPerPage">Rows per page:</label>
            <select id="rowsPerPage" v-model="rowsPerPage">
                <option v-for="option in rowsOptions" :value="option" :key="option">
                {{ option }}
                </option>
            </select>
        </div>

        <div>
            <button @click="prevPage" :disabled="page === 1">Previous</button>
            <span>Page {{ page }} of {{ totalPages }}</span>
            <button @click="nextPage" :disabled="page === totalPages">Next</button>
        </div>
    </div>
    

    
  </div>
</template>

<script>
import { ref, computed, onMounted } from "vue";
import axios from "axios";
import _ from "lodash";

export default {
  setup() {
    const comments = ref([]);
    const searchQuery = ref("");
    const page = ref(1);
    const rowsPerPage = ref(10);
    const sortColumn = ref("id");
    const sortOrder = ref("asc");
    const loading = ref(true);
    const rowsOptions = [10, 15, 20, "all"];

    const editing = ref(false);
    const editId = ref(null);
    const editComment = ref({ id: null, name: "", email: "", body: "" });

    const fetchComments = async () => {
      loading.value = true;
      const response = await axios.get(
        "https://jsonplaceholder.typicode.com/comments"
      );
      comments.value = response.data;
      loading.value = false;
    };

    onMounted(() => {
      fetchComments();
    });

    const filteredComments = computed(() => {
      return comments.value.filter(
        (comment) =>
          comment.name
            .toLowerCase()
            .includes(searchQuery.value.toLowerCase()) ||
          comment.email
            .toLowerCase()
            .includes(searchQuery.value.toLowerCase()) ||
          comment.body.toLowerCase().includes(searchQuery.value.toLowerCase())
      );
    });

    const sortedComments = computed(() => {
      return _.orderBy(
        filteredComments.value,
        [sortColumn.value],
        [sortOrder.value]
      );
    });

    const paginatedComments = computed(() => {
      const start = (page.value - 1) * rowsPerPage.value;
      return rowsPerPage.value === "all"
        ? sortedComments.value
        : sortedComments.value.slice(start, start + rowsPerPage.value);
    });

    const totalPages = computed(() => {
      return rowsPerPage.value === "all"
        ? 1
        : Math.ceil(filteredComments.value.length / rowsPerPage.value);
    });

    const sortBy = (column) => {
      if (sortColumn.value === column) {
        sortOrder.value = sortOrder.value === "asc" ? "desc" : "asc";
      } else {
        sortColumn.value = column;
        sortOrder.value = "asc";
      }
    };

    const nextPage = () => {
      if (page.value < totalPages.value) {
        page.value++;
      }
    };

    const prevPage = () => {
      if (page.value > 1) {
        page.value--;
      }
    };

    const debouncedSearch = _.debounce(() => {
      page.value = 1;
    }, 300);

    const startEdit = (comment) => {
      editing.value = true;
      editId.value = comment.id;
      editComment.value = { ...comment };
    };

    const saveEdit = () => {
      const index = comments.value.findIndex((c) => c.id === editId.value);
      if (index !== -1) {
        comments.value[index] = { ...editComment.value };
      }
      editing.value = false;
      editId.value = null;
    };

    const cancelEdit = () => {
      editing.value = false;
      editId.value = null;
    };

    const deleteComment = (id) => {
      comments.value = comments.value.filter((comment) => comment.id !== id);
    };

    return {
      comments,
      searchQuery,
      page,
      rowsPerPage,
      rowsOptions,
      sortColumn,
      sortOrder,
      loading,
      editing,
      editId,
      editComment,
      paginatedComments,
      totalPages,
      sortBy,
      nextPage,
      prevPage,
      debouncedSearch,
      startEdit,
      saveEdit,
      cancelEdit,
      deleteComment,
    };
  },
};
</script>
