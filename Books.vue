<template>
  <div class="container">
    <div class="row">
      <div class="col-sm-10">
        <h1>Books</h1>
        <hr><br><br>
        <!-- Props in action -->
        <alert message="Hard coded message"></alert>
        <alert :message=message v-if="showMessage"></alert>
        <button type="button" class="btn btn-success btn-sm" v-b-modal.book-modal>Add Book</button>
        <br><br>
        <table class="table table-hover">
          <thead>
            <tr>
              <th scope="col">Title</th>
              <th scope="col">Author</th>
              <th scope="col">Read?</th>
              <th></th>
            </tr>
          </thead>
          <tbody>
              <!-- V for loops and passing data -->
            <tr v-for="(book, index) in books" :key="index">
              <td>{{ book.title }}</td>
              <td>{{ book.author }}</td>
              <td>
                <span v-if="book.read">Yes</span>
                <span v-else>No</span>
              </td>
              <td>
                  <!-- Button to edit book -->
                <div class="btn-group" role="group">
                  <button type="button" 
                  class="btn btn-warning btn-sm"
                  v-b-modal.book-update-modal
                  @click="editBook(book)">
                  Update
                  </button>
                  <button type="button" 
                  class="btn btn-danger btn-sm"
                  @click="onDeleteBook(book)">Delete</button>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
    <b-modal ref="addBookModal"
            id="book-modal"
            title="Add a new book"
            hide-footer>
      <b-form @submit="onSubmit" @reset="onReset" class="w-100">
      <b-form-group id="form-title-group"
                    label="Title:"
                    label-for="form-title-input">
          <b-form-input id="form-title-input"
                        type="text"
                        v-model="addBookForm.title"
                        required
                        placeholder="Enter title">
          </b-form-input>
        </b-form-group>
        <b-form-group id="form-author-group"
                      label="Author:"
                      label-for="form-author-input">
            <b-form-input id="form-author-input"
                          type="text"
                          v-model="addBookForm.author"
                          required
                          placeholder="Enter author">
            </b-form-input>
          </b-form-group>
        <b-form-group id="form-read-group">
          <b-form-checkbox-group v-model="addBookForm.read" id="form-checks">
            <b-form-checkbox value="true">Read?</b-form-checkbox>
          </b-form-checkbox-group>
        </b-form-group>
        <b-button-group>
          <b-button type="submit" variant="primary">Submit</b-button>
          <b-button type="reset" variant="danger">Reset</b-button>
        </b-button-group>
      </b-form>
    </b-modal>
    <b-modal ref="editBookModal"
         id="book-update-modal"
         title="Update"
         hide-footer>
  <b-form @submit="onSubmitUpdate" @reset="onResetUpdate" class="w-100">
  <b-form-group id="form-title-edit-group"
                label="Title:"
                label-for="form-title-edit-input">
      <b-form-input id="form-title-edit-input"
                    type="text"
                    v-model="editForm.title"
                    required
                    placeholder="Enter title">
      </b-form-input>
    </b-form-group>
    <b-form-group id="form-author-edit-group"
                  label="Author:"
                  label-for="form-author-edit-input">
        <b-form-input id="form-author-edit-input"
                      type="text"
                      v-model="editForm.author"
                      required
                      placeholder="Enter author">
        </b-form-input>
      </b-form-group>
    <b-form-group id="form-read-edit-group">
      <b-form-checkbox-group v-model="editForm.read" id="form-checks">
        <b-form-checkbox value="true">Read?</b-form-checkbox>
      </b-form-checkbox-group>
    </b-form-group>
    <b-button-group>
      <b-button type="submit" variant="primary">Update</b-button>
      <b-button type="reset" variant="danger">Cancel</b-button>
    </b-button-group>
  </b-form>
</b-modal>
  </div>
</template>

<script>
import axios from 'axios';
import Alert from './Alert.vue';

export default {
  data() {
    return {
      books: [],
      addBookForm: {
        title: '',
        author: '',
        read: [],
      },
      message: '', // alert user
      showMessage: true,
      editForm: {
          id: '',
          title: '',
          author: '',
          read: [],
      }
    };
  },
  components: {
      alert: Alert,
  },
  methods: {
    getBooks() { //GET REQEST is called to server
      const path = 'http://localhost:5000/books';
      axios.get(path)
        .then((res) => { //retrieve the database 
          this.books = res.data.books;
        })
        .catch((error) => {
          // eslint-disable-next-line
          console.error(error);
        });
    },
    addBook(payload) { // POST request to server
      const path = 'http://localhost:5000/books';
      axios.post(path, payload) // payload is push
        .then(() => {
          this.getBooks(); // GET is called to retrieve the new expanded database 
          this.message = 'Book added!';
        })
        .catch((error) => {
          // eslint-disable-next-line
          console.log(error);
          this.getBooks();
        });
    },
    updateBook(payload, bookID) { //signal a PUT request to server
        const path = `http://localhost:5000/books/${bookID}`;
        axios.put(path, payload) //payload is push to server
        .then(() => { //sever updates the database
            this.getBooks();  //request data from server 
            this.message = 'Book updated!'; // alert user
            this.showMessage = true;
        })
        .catch((error) => {
            console.error(error);
            this.getBooks();
        });
    },
    initForm() {
      this.addBookForm.title = '';
      this.addBookForm.author = '';
      this.addBookForm.read = [];
      this.editForm.id = '';        //collects form input and pushes it to the data object
      this.editForm.title = '';
      this.editForm.author = '';
      this.editForm.read = [];
    },
    onSubmit(evt) {
      evt.preventDefault();
      this.$refs.addBookModal.hide();
      let read = false;
      if (this.addBookForm.read[0]) read = true;
      const payload = {
        title: this.addBookForm.title,
        author: this.addBookForm.author,
        read, // property shorthand
      };
      this.addBook(payload);
      this.initForm();
    },
    onReset(evt) {
      evt.preventDefault();
      this.$refs.addBookModal.hide();
      this.initForm();
    },
    editBook(book) {
        this.editForm = book;
    },
    onSubmitUpdate(evt) {
        evt.preventDefault();
        this.$refs.editBookModal.hide(); //hide the form
        let read = false;
        if (this.editForm.read[0]) read = true; //pass in data from the form
        const payload = {
            title: this.editForm.title,
            author: this.editForm.author,
            read,
        };
        this.updateBook(payload, this.editForm.id); //takes form input and push it to PUT
    },
    onResetUpdate(evt) {            //clears the submit form when you submit 
        evt.preventDefault();
        this.$refs.editBookModal.hide();
        this.initForm();
        this.getBooks();
    },
       removeBook(bookID) { //signal a DELETE request to server
        const path = `http://localhost:5000/books/${bookID}`;
        axios.delete(path) // i
        .then(() => { //sever updates the database and removes the book
            this.getBooks();  //request data from server 
            this.message = 'Book removed!'; // alert user
            this.showMessage = true;
        })
        .catch((error) => {
            console.error(error);
            this.getBooks();
        });
    },
    onDeleteBook(book) {
        this.removeBook(book.id);
    },
  },
  created() {
    this.getBooks();
  },
};
</script>