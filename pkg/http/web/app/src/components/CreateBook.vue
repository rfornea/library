<template>
  <v-app id="inspire">
    <v-content>
      <v-container fluid fill-height>
        <v-layout align-center justify-center>
          <v-flex xs12 sm8 md4>
            <v-card class="elevation-12">
              <v-toolbar dark color="teal">
                <v-toolbar-title justify-center>Add New Book</v-toolbar-title>
              </v-toolbar>
              <v-card-text>
                <label>Title:</label>
                <input
                  :class="validTitleStyle(book.title, 'title')"
                  v-model="book.title"
                  placeholder="* Enter Title"
                  @blur="
                    () => {
                      dirty.title = true;
                    }
                  "
                />
                <label>Author's First Name:</label>
                <input
                  :class="
                    validAuthorNameStyle(
                      book.authorFirstName,
                      'authorFirstName'
                    )
                  "
                  v-model="book.authorFirstName"
                  placeholder="* Enter Author's First Name"
                  @blur="
                    () => {
                      dirty.authorFirstName = true;
                    }
                  "
                />
                <label>Author's Last Name:</label>
                <input
                  :class="
                    validAuthorNameStyle(book.authorLastName, 'authorLastName')
                  "
                  v-model="book.authorLastName"
                  placeholder="* Enter Author's Last Name"
                  @blur="
                    () => {
                      dirty.authorLastName = true;
                    }
                  "
                />
                <label>ISBN:</label>
                <button
                  type="button"
                  class="btn btn-sm btn-outline-info book-generate-btn"
                  @click="generateRandomISBN()"
                >
                  Generate
                </button>
                <input
                  :class="validIsbnStyle(book.isbn)"
                  v-model="book.isbn"
                  placeholder="* Enter ISBN"
                  @blur="
                    () => {
                      dirty.isbn = true;
                    }
                  "
                />
                <label>Book Description:</label>
                <textarea
                  :class="validDescriptionStyle(book.description)"
                  class="book-description-input"
                  v-model="book.description"
                  placeholder="* Enter Book Description"
                  @blur="
                    () => {
                      dirty.description = true;
                    }
                  "
                ></textarea>
                <div class="container">
                  <button
                    type="button"
                    class="btn btn-sm btn-outline-info book-clear-btn"
                    @click="clearBook()"
                  >
                    Clear
                  </button>
                  <button
                    :disabled="!isEnabled()"
                    type="button"
                    class="btn btn-sm btn-outline-primary book-update-btn"
                    @click="createBook(book)"
                  >
                    Create
                  </button>
                </div>
                <br />
              </v-card-text>
            </v-card>
          </v-flex>
        </v-layout>
      </v-container>
    </v-content>
  </v-app>
</template>

<script>
import { mapActions } from "vuex";
import Vue from "vue";
import Toasted from "vue-toasted";

Vue.use(Toasted, {
  duration: 4000
});

export default {
  name: "CreateBook",
  data() {
    return {
      dirty: {
        description: false,
        title: false,
        isbn: false,
        authorLastName: false,
        authorFirstName: false
      },
      book: {
        title: "",
        authorLastName: "",
        authorFirstName: "",
        description: "",
        available: true,
        isbn: ""
      }
    };
  },
  computed: {},
  methods: {
    ...mapActions(["addBook"]),
    createBook: function(book) {
      this.$store
        .dispatch("addBook", book)
        .then(() => {
          this.$toasted.show(book.title + " has been created.", {
            type: "success",
            theme: "bubble"
          });
          this.$router.push({ name: "view-books", location: "books" });
        })
        .catch(() => {
          this.$toasted.show("There was an error.", {
            type: "danger",
            theme: "bubble"
          });
        });
    },
    clearBook: function() {
      this.book = {
        title: "",
        authorLastName: "",
        authorFirstName: "",
        description: "",
        available: true,
        isbn: ""
      };
      this.dirty = {
        description: false,
        title: false,
        isbn: false,
        authorLastName: false,
        authorFirstName: false
      };
    },
    generateRandomISBN() {
      let isbn = "",
        total = 0;
      for (let i = 0; i < 9; i++) {
        isbn += "" + Math.floor(Math.random() * 9);
        total += parseInt(isbn[i]) * (10 - i);
      }
      let modDiff = 11 - (total % 11);
      if (modDiff === 10) isbn += "X";
      else isbn += "" + modDiff;
      this.book.isbn =
        isbn[0] +
        "-" +
        isbn.slice(1, isbn.length - 1) +
        "-" +
        isbn[isbn.length - 1];
    },
    isEnabled: function() {
      return (
        this.validTitle(this.book.title) &&
        this.validAuthorName(this.book.authorFirstName) &&
        this.validAuthorName(this.book.authorLastName) &&
        this.isValidIsbn(this.book.isbn) &&
        this.book.description.length > 0
      );
    },
    validAuthorName(str) {
      let letters = /^[A-Za-z .]+$/;
      return (
        str.length >= 1 &&
        str.match(letters) !== undefined &&
        str.match(letters) !== null &&
        str.length === str.trim().length
      );
    },
    validAuthorNameStyle: function(input, prop) {
      return [
        this.validAuthorName(input) || !this.dirty[prop] ? "" : "input-error"
      ];
    },
    validTitle(str) {
      let letters = /^[A-Za-z0-9 '!,-]+$/;
      return (
        str.length >= 1 &&
        str.match(letters) !== undefined &&
        str.match(letters) !== null &&
        str.length === str.trim().length
      );
    },
    validTitleStyle: function(input, prop) {
      return [this.validTitle(input) || !this.dirty[prop] ? "" : "input-error"];
    },
    isValidIsbn(isbn) {
      let isbnRe = /\d-?\d{4}-?\d{4}-?[\dX]$/;
      if (!isbnRe.test(isbn)) return false;

      let testId = isbn.split("-").join("");
      let result = 0;
      for (let i = 0; i < testId.length; i++) {
        let cur = testId[i] === "X" ? 10 : parseInt(testId[i]);
        result += cur * (10 - i);
      }
      return result % 11 === 0;
    },
    validIsbnStyle: function(input) {
      return [this.isValidIsbn(input) || !this.dirty.isbn ? "" : "input-error"];
    },
    isValidDescription: function(description) {
      return (
        description.length > 0 &&
        description.length === description.trim().length
      );
    },
    validDescriptionStyle: function(input) {
      return [
        this.isValidDescription(input) || !this.dirty.description
          ? ""
          : "input-error"
      ];
    }
  }
};
</script>

<style>
label {
  margin-left: 5px;
  font-size: 1.25em;
}
textarea {
  border: 1px solid #cac6c6;
  padding: 1px;
  margin: 0 0 5px 5px;
  width: 100%;
}
input {
  border: 1px solid #cac6c6;
  padding: 1px;
  margin: 0 0 5px 5px;
  width: 100%;
}
/*this class needed for error styles, do not delete*/
.input-error {
  border: 1px solid red;
}
.book-generate-btn {
  float: right;
  margin-bottom: 3px;
}
.book-clear-btn {
  float: right;
}
.book-update-btn {
  float: right;
  margin-right: 5px;
}
.book-description-input {
  height: 150px;
}
</style>
