<template>
    <div class="container">
      <h1>Joke's API</h1>
        <div class="logo-animation">
        <LogoAnimation />
      </div>
  
      <input type="text" v-model="searchQuery" placeholder="Search jokes..." class="search-bar" />
  
      <label for="sortBy">Sort by: </label>
      <select v-model="sortBy" @change="sortJokes" class="styled-dropdown">
        <option value="alphabetical">Alphabetically</option>
        <option value="length">By length</option>
      </select>
  
      <div v-if="!error" class="card-container">
        <transition-group name="fade" tag="div" class="cards-list">
          <div v-for="(joke, index) in paginatedJokes" :key="index" class="card">

            <button class="delete-button" @click="deleteJoke(index)">✕</button>
  
            <div @click="togglePunchline(index)" class="joke-setup">
              <strong>{{ joke.setup }}</strong>
            </div>
  
            <div v-if="showPunchline[index]" class="joke-punchline">
              - {{ joke.punchline }}
            </div>
  
            <div class="rating">
              <span 
                v-for="star in 5" 
                :key="star" 
                @click="rateJoke(index, star)"
                class="star"
                :class="{ filled: star <= (ratings[index] || 0) }"
              >
                ★
              </span>
            </div>
          </div>
        </transition-group>
      </div>
  
      <div class="pagination-controls">
        <button :disabled="currentPage === 1" @click="prevPage" class="pagination-button">Preview</button>
        <span class="page-number">Page {{ currentPage }} </span>
        <button :disabled="currentPage === totalPages" @click="nextPage" class="pagination-button">Next</button>
      </div>
  
      <div v-if="error">{{ error }}</div>
  
      <div class="add-joke-form">
        <h2>Add a new joke</h2>
        <input type="text" v-model="newJokeSetup" placeholder="Enter joke setup" class="form-input" />
        <input type="text" v-model="newJokePunchline" placeholder="Enter joke punchline" class="form-input" />
        <button @click="addJoke" class="add-joke-button">Add Joke</button>
      </div>
    </div>
  </template>
  
  <script>
    import LogoAnimation from './LogoAnimation.vue';
    export default {
      components: {
        LogoAnimation,
      },
      data() {
        return {
          jokes: [],
          ratings: [],
          showPunchline: [],
          sortBy: "alphabetical",
          searchQuery: "",
          currentPage: 1,
          jokesPerPage: 5,
          newJokeSetup: "", 
          newJokePunchline: "", 
          error: null
        };
      },
      computed: {
        filteredJokes() {
          return this.jokes.filter(joke =>
            joke.setup.toLowerCase().includes(this.searchQuery.toLowerCase())
          );
        },
        sortedJokes() {
          const sorted = [...this.filteredJokes];
          if (this.sortBy === 'alphabetical') {
            return sorted.sort((a, b) => a.setup.localeCompare(b.setup));
          } else if (this.sortBy === 'length') {
            return sorted.sort((a, b) => a.setup.length - b.setup.length);
          }
          return sorted;
        },
        paginatedJokes() {
          const start = (this.currentPage - 1) * this.jokesPerPage;
          const end = start + this.jokesPerPage;
          return this.sortedJokes.slice(start, end);
        },
        totalPages() {
          return Math.ceil(this.sortedJokes.length / this.jokesPerPage);
        }
      },
      methods: {
        prevPage() {
          if (this.currentPage > 1) {
            this.currentPage--;
          }
        },
        nextPage() {
          if (this.currentPage < this.totalPages) {
            this.currentPage++;
          }
        },
        rateJoke(index, rating) {
          this.ratings[index] = rating;
        },
        deleteJoke(index) {
          this.jokes.splice(index, 1);
          this.ratings.splice(index, 1);
          this.showPunchline.splice(index, 1);
        },
        togglePunchline(index) {
          this.showPunchline[index] = !this.showPunchline[index];
        },
        addJoke() {
          if (this.newJokeSetup && this.newJokePunchline) {
            // new joke
            this.jokes.push({
              setup: this.newJokeSetup,
              punchline: this.newJokePunchline,
            });
            this.showPunchline.push(false);
            this.ratings.push(0);
  
            this.newJokeSetup = "";
            this.newJokePunchline = "";
          } else {
            alert("Both fields are required!");
          }
        }
      },
      mounted() {
        fetch('https://official-joke-api.appspot.com/jokes/ten')
          .then(response => {
            if (!response.ok) throw new Error('Error loading jokes');
            return response.json();
          })
          .then(data => {
            this.jokes = data;
            this.showPunchline = new Array(this.jokes.length).fill(false);
            this.ratings = new Array(this.jokes.length).fill(0);
            // console.log('data:', data);
          })
          .catch(error => {
            this.error = error.message || "Unknown error";
          });
      }
    };
  </script>
  
  <style scoped>
  .container {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center; 
    background-color: #d8b2d9;
    min-height: 100vh;
    padding: 20px;
    font-family: 'Poppins', sans-serif;
  }
  
  .search-bar {
    padding: 10px;
    border-radius: 8px;
    border: 1px solid #ccc;
    font-size: 16px;
    margin-bottom: 20px;
    width: 300px;
    font-family: 'Poppins', sans-serif;
  }
  
  .card-container {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    gap: 16px;
    max-width: 800px;
    margin: 0 auto;
  }
  
  .card {
    width: 300px;
    border: 1px solid #ccc;
    border-radius: 8px;
    padding: 16px;
    margin: 16px 0;
    position: relative;
    background-color: #f9f9f9;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  }
  
  .delete-button {
    position: absolute;
    top: 4px;
    right: 4px;
    background-color: transparent;
    color: red;
    border: none;
    font-size: 12px;
    cursor: pointer;
  }
  
  .joke-setup {
    font-size: 18px;
    cursor: pointer;
    margin-bottom: 8px;
  }
  
  .joke-punchline {
    font-style: italic;
    color: #555;
  }
  
  .rating {
    display: flex;
    align-items: center;
    justify-content: center;
  }
  
  .star {
    font-size: 24px;
    color: gray;
    cursor: pointer;
    margin-left: 4px;
    transition: color 0.2s;
  }
  
  .star:hover,
  .star.filled {
    color: gold;
  }
  
  .styled-dropdown {
    padding: 10px;
    border-radius: 8px;
    border: 1px solid #ccc;
    background-color: white;
    font-family: 'Poppins', sans-serif;
    margin-bottom: 20px;
    font-size: 16px;
    transition: border-color 0.3s ease;
    outline: none;
  }
  
  .styled-dropdown:focus {
    border-color: #7b42f6;
  }
  
  .pagination-controls {
    margin-top: 16px;
  }
  
  .pagination-button {
    padding: 10px 20px;
    border-radius: 8px;
    background-color: #7b42f6;
    color: white;
    border: none;
    cursor: pointer;
    font-family: 'Poppins', sans-serif;
    transition: background-color 0.3s ease;
  }
  
  .pagination-button:hover {
    background-color: #4f2cb0;
  }
  
  .pagination-button:disabled {
    background-color: #ccc;
    cursor: not-allowed;
  }
  
  .logo-animation {
    margin-top: -60px;
  }
  
  .page-number {
    padding: 10px;
  }
  
  .add-joke-form {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    margin-top: 40px;
  }
  
  .add-joke-form input {
    padding: 10px;
    border-radius: 8px;
    border: 1px solid #ccc;
    margin: 5px;
    width: 300px;
    font-family: 'Poppins', sans-serif;
  }
  
  .add-joke-button {
    padding: 10px 20px;
    border-radius: 8px;
    background-color: #7b42f6;
    color: white;
    border: none;
    cursor: pointer;
    font-family: 'Poppins', sans-serif;
    transition: background-color 0.3s ease;
    margin-top: 10px;
  }
  
  .add-joke-button:hover {
    background-color: #4f2cb0;
  }
  
  .fade-enter-active, .fade-leave-active {
    transition: opacity 0.5s;
  }
  .fade-enter, .fade-leave-to {
    opacity: 0;
  }
  
  @media (max-width: 600px) {
    .card {
      width: 300px;
      margin: 8px 0;
    }
  
    .pagination-button {
      padding: 8px 16px;
    }
  
    .search-bar, .styled-dropdown {
      width: 300px;
    }
  
    .add-joke-form input {
      width: 300px;
    }
  }
  </style>
  