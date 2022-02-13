<template>
  <div class="home">
    <div class="home__table-container">
      <table class="table has-background-dark is-narrow is-hoverable">
        <thead>
          <tr>
            <th
              v-for="category in categories"
              :key="category.id"
              @click="updateFilters(category.id)"
              class="has-text-white item-hover"
            >
              {{ category.title }}
            </th>
            <th>
              <div :class="dropdownClasses" class="dropdown" @click="openDropdown">
                <div class="dropdown-trigger">
                  <button class="button" aria-haspopup="true" aria-controls="dropdown-menu">
                    <span>Elements affichés: {{ limit === 0 ? 'Tous les films' : limit }}</span>
                    <span class="icon is-small">
                      <i class="fas fa-angle-down" aria-hidden="true"></i>
                    </span>
                  </button>
                </div>
                <div class="dropdown-menu" role="menu">
                  <div class="dropdown-content">
                    <p
                      v-for="limit in limitList"
                      :key="limit.title"
                      @click="updateLimitValue(limit.value)"
                      class="dropdown-item"
                    >
                      {{ limit.title }}
                    </p>
                  </div>
                </div>
              </div>
            </th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="film in films"
            :key="film.title"
            class="has-text-white"
          >
            <td>{{ film.title }}</td>
            <td>{{ film.amount || 'Prix non renseigné' }}</td>
            <td>{{ film.rating }}</td>
            <td>{{ film.category }}</td>
            <td>{{ film.rentNb }}</td>
          </tr>
        </tbody>
      </table>
    </div>
    <nav class="pagination is-centered ${}" role="navigation" aria-label="pagination">
      <ul class="pagination-list">
        <li v-if="nbPages > 1 && limit !== 0" @click="navigate(1)" class="item-hover">
          <p :class="`pagination-link ${currentPage === 1 ? 'currentPage' : ''}`">1</p>
        </li>
        <li v-if="limit !== 0"><span class="pagination-ellipsis">&hellip;</span></li>
        <li
          v-for="page in pagination"
          :key="`page-${page}`"
          @click="navigate(page)"
          class="item-hover"
        >
          <p :class="`pagination-link ${page === currentPage ? 'currentPage' : ''}`">{{ page }}</p>
        </li>
        <li v-if="limit !== 0"><span class="pagination-ellipsis">&hellip;</span></li>
        <li v-if="limit !== 0" class="item-hover" @click="navigate(nbPages)">
          <p :class="`pagination-link ${currentPage === nbPages ? 'currentPage' : ''}`">{{ nbPages }}</p>
        </li>
      </ul>
    </nav>
    <!-- Loader -->
    <div v-if="isLoading" class="home__loader">
      <img :src="require('@/assets/img/loader.gif')">
    </div>
  </div>
</template>

<script>
// utils
import axios from 'axios';

export default {

  name: 'Home',

  data() {
    return {
      categories: [
        { id: 1, title: 'Nom' },
        { id: 2, title: 'Prix' },
        { id: 3, title: 'Classement' },
        { id: 4, title: 'Genre' },
        { id: 5, title: 'Nombre de locations' },
      ],
      limitList: [
        { value: 10, title: '10' },
        { value: 25, title: '25' },
        { value: 50, title: '50' },
        { value: 100, title: '100' },
        { value: 0, title: 'Tout afficher' },
      ],
      films: [],
      nbPages: 0,
      limit: 10,
      currentPage: 1,
      orderType: 1,
      orderByDesc: 0,
      dropDownIsOpened: false,
      isLoading: false,
    }
  },

  computed: {
    dropdownClasses() {
      return {
        'is-active': this.dropDownIsOpened
      }
    },
    pagination() {
      const pages = [];
      if (this.limit === 0) return [1];
      if (this.currentPage - 1 > 1) {
        pages.push(this.currentPage - 1);
      }
      if (this.currentPage !== 1 && this.currentPage !== this.nbPages) {
        pages.push(this.currentPage);
      }
      if (this.currentPage + 1 <= this.nbPages - 1 && this.currentPage !== this.nbPages) {
        pages.push(this.currentPage + 1);
      }
      return pages;
    }
  },

  mounted() {
    this.refresh();
  },

  methods: {
    refresh() {
      const { limit, currentPage, orderType, orderByDesc } = this;
      this.isLoading = true;
      axios
        .get(`https://film-api-lgbdr.herokuapp.com/api/film?limit=${limit}&page=${currentPage}&orderType=${orderType}&orderByDesc=${orderByDesc}`)
        .then(({ data }) => {
          this.films = data.films;
          this.nbPages = data.nbPages;
          setTimeout(() => {
            this.isLoading = false;
          }, 250);
        }).catch(() => {
          this.isLoading = false;
        });
    },
    updateFilters(categoryId) {
      this.orderType = categoryId;
      this.orderByDesc = Number(!this.orderByDesc);
      this.refresh();
    },
    openDropdown() {
      this.dropDownIsOpened = !this.dropDownIsOpened;
    },
    updateLimitValue(value) {
      if (value === this.limit) return;
      this.limit = value;
      this.currentPage = 1;
      this.refresh();
    },
    navigate(page) {
      if (page === this.currentPage) return;
      this.currentPage = page;
      this.refresh();
    }
  },

}
</script>

<style lang="scss" scoped>
  .home {
    height: calc(100vh - 60px);
    width: 100%;

    &__table-container {
      margin-top: 40px;
      height: 85%;
      overflow-y: scroll;
      display: flex;
      justify-content: center;
      margin-bottom: 20px;

      table {
        width: 60%;
      }

      thead {
        position: relative;

        th {
          position: sticky;
          background-color: rgb(41, 44, 46);
          top: 0
        }
      }
    }

    .pagination-link {
      color: #fafafa;
    }

    .table.is-hoverable tbody tr:not(.is-selected):hover {
      background-color: rgb(41, 40, 40);
      cursor: pointer;
    }

    .item-hover {

      &:hover {
        cursor: pointer;
        transform: scale(1.02);
        background-color: rgb(29, 31, 32);
        border-radius: 5px;
      }
    }

    .currentPage {
      background-color: rgb(48, 48, 200);
    }

    .dropdown {

      .dropdown-item {

        &:hover {
          background-color: #7c7a7a;
          cursor: pointer;
        }
      }
    }

    &__loader {
      height: 100%;
      width: 100%;
      position: absolute;
      top: 0;
      left: 0;
      background-color: rgba(14, 13, 13, 0.74);
      display: flex;
      align-items: center;
      justify-content: center;
    }
  }
</style>
