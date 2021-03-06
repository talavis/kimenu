<template>
<q-page class="justify-center">
  <q-toolbar v-if="tmpState === undefined"
             class="justify-center text-primary">
    <q-toggle v-model="onlyFavourites"
              label="Favourites" />

    <q-toggle v-if="!onlyFavourites"
              v-model="showSolna"
	      label="Solna" />
    <q-toggle v-else
              color="accent"
	      label="Solna"
              v-model="constTrue"
              disable />

    <q-toggle v-if="!onlyFavourites"
              v-model="showUppsala"
	      label="Uppsala" />
    <q-toggle v-else
              color="accent"
              v-model="constTrue"
              label="Uppsala"
              disable />
  </q-toolbar>

  <div class="flex justify-center">
    <q-list class="flex-center">
      <res-entry v-for="restaurant in visibleRestaurants"
                 :key="restaurant.identifier"
                 :restaurantBase="restaurant" />
    </q-list>

    <q-inner-loading :showing="loading">
      <q-spinner-dots color="primary"
                      size="2em" />
    </q-inner-loading>
  </div>
</q-page>
</template>

<script>
import axios from 'axios';

import RestaurantEntry from 'components/RestaurantEntry.vue'

export default {
  name: 'PageIndex',

  props: ["tmpState"],
  
  components: {
    'res-entry': RestaurantEntry,
  },
  
  computed: {
    restaurants: {
      get () {
        return this.$store.state.main.restaurants;
      },
    },

    showSolna: {
      get () {
        return this.$store.state.main.showSolna;
      },
      set (val) {
        this.$store.dispatch('main/setShowSolna', val)
      }

    },

    showUppsala: {
      get () {
        return this.$store.state.main.showUppsala;
      },
      set (val) {
        this.$store.dispatch('main/setShowUppsala', val)
      }

    },

    favourites: {
      get () {
        return this.$store.state.main.favourites;
      },
    },

    onlyFavourites: {
      get () {
        return this.$store.state.main.onlyFavourites;
      },
      set (val) {
        this.$store.dispatch('main/setOnlyFavourites', val)
      }

    },

    visibleRestaurants: {
      get () {
        let current = this.restaurants;

        if (this.tmpState !== undefined) {
          if (this.tmpState !== 'solna') {
            current = current.filter((value) => value.campus !== 'Solna');
          }
          if (this.tmpState !== 'bmc') {
            current = current.filter((value) => value.campus !== 'Uppsala');
          }
        }
        else {
          if (this.onlyFavourites) {
            current = current.filter((value) => this.favourites.includes(value.identifier));
          }
          else {
            if (!this.showSolna) {
              current = current.filter((value) => value.campus !== 'Solna');
            }
            if (!this.showUppsala) {
              current = current.filter((value) => value.campus !== 'Uppsala');
            }
          }
        }
        current = current.sort((a,b) => {
          if (a.campus > b.campus)
            return 1;
          if (a.campus < b.campus)
            return -1;
          return (a.name > b.name) ? 1 : ((b.name > a.name) ? -1 : 0);
        });

        return current;
      }
    },
  },
  
  data () {
    return {
      error: false,
      loading: true,
      constTrue: true,
    }
  },

  created () {
    this.$store.dispatch('main/getRestaurants')
      .then(() => this.loading = false);
  }
}
</script>
