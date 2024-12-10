<template>
  <div class="container mt-4">
    <!-- Create Post Card -->
    <div class="card shadow mb-4">
      <div class="card-header">
        <h3>Create a Post</h3>
      </div>
      <div class="card-body">
        <!-- Title and Content Inputs -->
        <div class="mb-3">
          <input
              type="text"
              v-model="title"
              class="form-control"
              placeholder="Enter title"
          />
        </div>
        <div class="mb-3">
          <input
              type="text"
              v-model="content"
              class="form-control"
              placeholder="Enter content"
          />
        </div>

        <!-- Category Dropdown -->
        <div class="mb-3">
          <label for="category" class="form-label">Select Category</label>
          <select
              v-model="category"
              class="form-select"
              id="category"
          >
            <option v-for="category in categories" :key="category.id" :value="category.id">
              {{ category.name }}
            </option>
          </select>
        </div>

        <!-- Tags Dropdown -->
        <div class="mb-3">
          <label for="tags" class="form-label">Select Tags</label>
          <select
              v-model="selected_tags"
              class="form-select"
              id="tags"
              multiple
          >
            <option v-for="tag in tags" :key="tag.id" :value="tag.id">
              {{ tag.name }}
            </option>
          </select>
        </div>

        <!-- Create Button -->
        <button
            class="btn btn-primary w-100"
            @click="addPost()"
        >
          Create Post
        </button>
      </div>
    </div>

    <!-- Posts List -->
    <div>
      <h3>Posts</h3>
      <div
          v-for="el in posts"
          :key="el.id"
          class="card mb-3 shadow-sm"
      >
        <div class="card-body">
          <h5 class="card-title">{{ el.title }}</h5>
          <p class="card-text">Category: {{ getCategoryName(el.category_id) }}</p>

          <!-- Edit and Delete Buttons -->
          <button
              class="btn btn-outline-secondary me-2"
              @click="edit_click(el)"
          >
            Edit
          </button>
          <button
              class="btn btn-outline-danger"
              @click="deletePost(el)"
          >
            Delete
          </button>
        </div>

        <!-- Edit Post Form -->
        <div v-if="el.isEdit" class="card-body bg-light">
          <h6>Edit Post</h6>
          <div class="mb-2">
            <input
                type="text"
                v-model="edited_title"
                class="form-control"
                placeholder="Enter new title"
            />
          </div>
          <div class="mb-2">
            <input
                type="text"
                v-model="edited_content"
                class="form-control"
                placeholder="Enter new content"
            />
          </div>

          <!-- Category and Tags for Edit -->
          <div class="mb-2">
            <select
                v-model="category"
                class="form-select"
            >
              <option v-for="category in categories" :key="category.id" :value="category.id">
                {{ category.name }}
              </option>
            </select>
          </div>
          <div class="mb-2">
            <select
                v-model="selected_tags"
                class="form-select"
                multiple
            >
              <option v-for="tag in tags" :key="tag.id" :value="tag.id">
                {{ tag.name }}
              </option>
            </select>
          </div>

          <!-- Save Button -->
          <button
              class="btn btn-success w-100"
              @click="postUpdate(el)"
          >
            Save
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      posts: [], // Posts List
      title: '',
      content: '',
      user_id: '',
      edited_title: '',
      edited_content: '',
      isEdit: false,
      categories: [], // Categories List
      selectedCategory: null,
      category: '',
      tag: '',
      tags: [], // Tags List
      selected_tags: []
    };
  },
  methods: {
    // Add new post
    addPost() {
      axios.post('http://127.0.0.1:8000/api/posts', {
        'title': this.title,
        'content': this.content,
        'category_id': this.category,
        'tags': this.selected_tags
      }).then(() => {
        this.all_posts();
        this.title = '';
        this.content = '';
        this.category = '';
        this.selected_tags = [];
      });
    },

    // Delete post
    deletePost(post) {
      axios.delete(`http://127.0.0.1:8000/api/posts/${post.id}`).then(() => {
        this.all_posts();
      });
    },

    // Get category name by ID
    getCategoryName(category_id) {
      const category = this.categories.find(cat => cat.id === category_id);
      return category ? category.name : "Unknown Category";
    },

    // Enable edit mode for a post
    edit_click(el) {
      el.isEdit = true;
      this.edited_title = el.title;
      this.edited_content = el.content;
      this.category = el.category_id;

      // Безопасная проверка перед вызовом map
      this.selected_tags = Array.isArray(el.tags) ? el.tags.map(tag => tag.id) : [];
    },

    // Update post
    postUpdate(el) {
      axios.put(`http://127.0.0.1:8000/api/posts/${el.id}`, {
        'title': this.edited_title,
        'content': this.edited_content,
        'category_id': this.category,
        'tags': this.selected_tags
      }).then(() => {
        el.isEdit = false;
        this.edited_title = '';
        this.edited_content = '';
        this.all_posts();
      });
    },

    // Get all posts
    all_posts() {
      axios.get('http://127.0.0.1:8000/api/posts')
          .then((response) => {
            this.posts = response.data;
          });
    },

    // Get all categories
    all_categories() {
      axios.get('http://127.0.0.1:8000/api/categories').then((response) => {
        this.categories = response.data;
      });
    },

    // Get all tags
    all_tags() {
      axios.get('http://127.0.0.1:8000/api/tags').then((response) => {
        this.tags = response.data;
      });
    }
  },
  mounted() {
    this.all_posts();
    this.all_categories();
    this.all_tags();
  }
}
</script>

<style scoped>
/* You can customize the styling here if needed */
</style>
