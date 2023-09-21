<template>
    <div class="row">
        <div class="col-12 mb-2 text-end">
            <b-button @click="openAddCategoryModal"  variant="primary">Add Category</b-button>
        </div>
            <div class="col-12">
                <div class="card">
                    <div class="card-header">
                        <h4>List Categories</h4>
                    </div>
                    <div class="card-body">
                        <div class="table-responsive">
                            <table class="table table-bordered">
                                <thead>
                                    <tr>
                                        <th>ID</th>
                                        <th>Title</th>
                                        <th>Description</th>
                                        <th>Actions</th>
                                    </tr>
                                </thead>
                                <tbody v-if="categories.length > 0">
                                    <tr v-for="(category,key) in categories" :key="key">
                                        <td>{{ category.id }}</td>
                                        <td>{{ category.title }}</td>
                                        <td>{{ category.description }}</td>
                                        <td>
                                            <button type="button" @click="openEditCategoryModal(category)" class="btn btn-success">Edit</button>
                                            <button type="button" @click="openDeleteCategoryModal(category.id)" class="btn btn-danger">Delete</button>
                                        </td>
                                    </tr>
                                </tbody>
                                <tbody v-else>
                                    <tr>
                                        <td colspan="4" align="center">No Categories Found.</td>
                                    </tr>
                                </tbody>
                            </table>
                        </div>
                    </div>
                </div>
            </div>

            <div>
                <b-modal
                    id="category-modal"
                    ref="modal"
                    :title="editMode ? 'Edit Category' : 'Add Category'"
                    @show="editMode ? true : resetModal"
                    @hidden="editMode ? true : resetModal"
                    @ok="editMode ? updateCategory() : createCategory()"
                >
                    <form ref="form" @submit.prevent="editMode ? updateCategory() : createCategory()">
                    <b-form-group
                        label="Title"
                        label-for="title-input"
                        invalid-feedback="Title is required"
                    >
                        <b-form-input
                        id="title-input"
                        v-model="category.title"
                        required
                        ></b-form-input>
                    </b-form-group>

                    <b-form-group
                        label="Description"
                        label-for="description-input"
                        invalid-feedback="Description is required"
                    >
                        <b-form-input
                        id="description-input"
                        v-model="category.description"
                        required
                        ></b-form-input>
                    </b-form-group>
                    </form>
                </b-modal>
            </div>
            <div>
                <b-modal
                    id="delete-category-modal"
                    ref="deleteModal"
                    title="Confirmar Eliminación"
                    @ok="confirmDeleteCategory"
                    @cancel="cancelDeleteCategory"
                    v-model="deleteModalShow"
                    >
                    <p>¿Estás seguro de que deseas eliminar esta categoría?</p>
                    <p>ID: {{ categoryIdToDelete }}</p>
                </b-modal>
            </div>
    </div>
</template>

<script>
export default {
  name: "categories",
  data() {
    return {
      categories: [],
      modalShow: false,
      category: {
        id: null,
        title: "",
        description: ""
      },
      editMode: false,
      deleteModalShow: false,
      categoryIdToDelete: null,
    };
  },
  mounted() {
    this.getCategories();
  },
  methods: {
    openAddCategoryModal() {
      this.editMode = false;
      this.category.id = null;
      this.category.title = "";
      this.category.description = "";
      this.$refs.modal.show();
    },

    openEditCategoryModal(category) {
        this.editMode = true;
        this.category.id = category.id;
        this.category.title = category.title;
        this.category.description = category.description;
        this.$refs.modal.show();
    },

    openDeleteCategoryModal(id) {
      this.categoryIdToDelete = id;
      this.deleteModalShow = true;
    },

    confirmDeleteCategory() {
      if (this.categoryIdToDelete) {
        this.deleteCategory(this.categoryIdToDelete);
        this.deleteModalShow = false;
      }
    },

    cancelDeleteCategory() {
      this.categoryIdToDelete = null;
      this.deleteModalShow = false;
    },

    resetModal() {
      this.category.id = null;
      this.category.title = "";
      this.category.description = "";
    },

    async createCategory() {
        await this.axios.post("/api/category", this.category)
          .then((response) => {
            console.log("Created successful");
            this.$refs.modal.hide();
            this.getCategories();
        })
          .catch((error) => {
            console.log(error);
        });
    },

    async updateCategory() {
        await this.axios.put(`/api/category/${this.category.id}`, this.category)
            .then((response) => {
                console.log("Update successful");
                this.$refs.modal.hide();
                this.getCategories();
            })
            .catch((error) => {
                console.error("Update error:", error);
            });
    },

    async getCategories(){
        await this.axios.get('/api/category').then(response=>{
            this.categories = response.data
        }).catch(error=>{
            console.log(error)
            this.categories = []
        })
    },

    deleteCategory(id){
        this.axios.delete(`/api/category/${id}`).then(response=>{
            this.getCategories()
        }).catch(error=>{
            console.log(error)
        })
    },
  },
};
</script>
