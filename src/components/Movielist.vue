<template>
  <div>
    <!-- navigation -->
    <nav class="navbar navbar-expand-lg navbar-light bg-light p-2">
      <a class="navbar-brand" href="./index.html">Movie List</a>
      <button
        class="navbar-toggler"
        type="button"
        data-toggle="collapse"
        data-target="#navbarSupportedContent"
        aria-controls="navbarSupportedContent"
        aria-expanded="false"
        aria-label="Toggle navigation"
      >
        <span class="navbar-toggler-icon"></span>
      </button>
      <div class="collapse navbar-collapse" id="navbarSupportedContent">
        <ul class="navbar-nav mr-auto">
          <li class="nav-item">
            <a class="nav-link" href="#all" @click="setVisibility('all')"
              >Home
              <span class="sr-only">(current)</span>
            </a>
          </li>
          <li class="nav-item">
            <a
              class="nav-link"
              href="#favortie"
              @click="setVisibility('favorite')"
              >favorite</a
            >
          </li>
        </ul>
      </div>
    </nav>

    <div class="container mt-5">
      <!--search bar-->
      <div class="row" id="search-bar">
        <form id="search" class="form-inline">
          <label class="sr-only" for="inlineFormInputName2">Name</label>
          <input
            type="text"
            class="form-control mb-2 mr-sm-2"
            id="search-input"
            placeholder="search name ..."
            v-model="searchText"
          />
          <button
            type="submit"
            class="btn btn-primary mb-2"
            @click.prevent="searchMovie()"
          >
            Search
          </button>
        </form>
      </div>

      <!-- data-panel -->
      <div class="container mt-5">
        <div class="row" id="data-panel">
          <!-- print movie list here -->
          <div class="col-sm-3" v-for="movie in filterMovies" :key="movie.id">
            <div class="card mb-2">
              <img
                class="card-img-top"
                alt="Card image cap"
                :src="movie.image"
              />
              <div class="card-body movie-item-body">
                <h5 class="card-title">{{ movie.title }}</h5>
              </div>
              <!-- "More" button -->
              <div class="card-footer">
                <button
                  class="btn btn-primary btn-show-movie"
                  data-toggle="modal"
                  data-target="#show-movie-modal"
                  @click="showModal(movie.id)"
                >
                  More
                </button>
                <button
                  class="btn btn-info btn-add-favorite"
                  @click="addToFavorite(movie)"
                >
                  +
                </button>
                <button
                  class="btn btn-danger btn-remove-favorite"
                  @click="removeMovie(movie)"
                >
                  X
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Modal -->
      <div
        class="modal fade"
        id="show-movie-modal"
        tabindex="-1"
        role="dialog"
        aria-labelledby="exampleModalLongTitle"
        aria-hidden="true"
      >
        <div class="modal-dialog modal-lg" role="document">
          <div class="modal-content">
            <div class="modal-header">
              <h5 class="modal-title" id="show-movie-title">
                {{ showMovie.title }}
              </h5>
              <button
                type="button"
                class="close"
                data-dismiss="modal"
                aria-label="Close"
              >
                <span aria-hidden="true">×</span>
              </button>
            </div>
            <div class="modal-body" id="show-movie-body">
              <div class="row">
                <img
                  class="col-sm-8"
                  id="show-movie-image"
                  :src="showMovie.image"
                />
                <div class="col-sm-4">
                  <p>
                    <em id="show-movie-date"
                      >release at :{{ showMovie.release_date }}</em
                    >
                  </p>
                  <p id="show-movie-description">{{ showMovie.description }}</p>
                </div>
              </div>
            </div>
            <div class="modal-footer">
              <button
                type="button"
                class="btn btn-secondary"
                data-dismiss="modal"
              >
                Close
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
  <!-- #app -->
</template>

<script>
import axios from "axios";
import $ from "jquery";

const BASE_URL = "https://movie-list.alphacamp.io";
const INDEX_URL = BASE_URL + "/api/v1/movies/";
const POSTER_URL = BASE_URL + "/posters/";

export default {
  name: "Movielist",
  data: function () {
    return {
      movies: [],
      showMovie: "",
      searchText: "",
      favoriteMovies: [],
      visibility: "all",
    };
  },
  created() {
    axios
      .get(INDEX_URL)
      .then((response) => {
        // map() 運算原始資料，回傳新的陣列
        // 這裡在 {} 外加 ()，可以自動 return 物件
        this.movies = response.data.results.map((movie) => ({
          ...movie,
          image: POSTER_URL + movie.image,
        }));
      })
      .catch((error) => console.log(error));
    this.favoriteMovies =
      JSON.parse(localStorage.getItem("favoriteMovies")) || [];
  },
  methods: {
    showModal(id) {
      this.showMovie = this.movies.find((_movie) => _movie.id === id);
      $("#show-movie-modal").modal("show");
    },
    searchMovie() {
      const selection = this.movies.filter((_movie) =>
        _movie.title.toLowerCase().includes(this.searchText.toLowerCase())
      );
      console.log(selection);
    },
    addToFavorite(data) {
      const movie = this.movies.find((_movie) => _movie.id === data.id);
      if (this.favoriteMovies.some((_movie) => _movie.id === data.id)) {
        return alert("此電影已經在收藏清單中！");
      }
      this.favoriteMovies.push(movie);
      localStorage.setItem(
        "favoriteMovies",
        JSON.stringify(this.favoriteMovies)
      );
    },
    setVisibility(text) {
      this.visibility = text;
    },
    removeMovie(favoriteMovie) {
      var yes = confirm("你確定嗎？");
      if (yes) {
        this.favoriteMovies = this.favoriteMovies.filter(
          (_movie) => _movie.id !== favoriteMovie.id
        );
        localStorage.setItem(
          "favoriteMovies",
          JSON.stringify(this.favoriteMovies)
        );
      } else {
        alert("你按了取消按鈕");
      }
    },
  },
  computed: {
    filterMovies() {
      if (this.searchText) {
        return this.movies.filter((_movie) =>
          _movie.title.toLowerCase().includes(this.searchText.toLowerCase())
        );
      }
      if (this.visibility === "all") {
        return this.movies;
      }
      if (this.visibility === "favorite") {
        return this.favoriteMovies;
      }
      return false;
    },
  },
};
</script>

<style lang="scss" scoped>
.btn-remove-favorite {
  display: none;
}
.canremove {
  .btn-add-favorite {
    display: none;
  }
  .btn-remove-favorite {
    display: inline-block;
  }
}
</style>
