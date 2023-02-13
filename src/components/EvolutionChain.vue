<!-- eslint-disable vue/no-use-v-if-with-v-for -->
<template>
  <div>
    <v-row class="evolution">
      <template v-for="(chain, index) in evolutions()">
        <v-col cols="12" md="3" :key="`evolution-${index}`" v-if="typeof chain == 'object'">
          <img :src="`https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${getId(chain)}.png`"
            width="96" height="96" alt="">
          <h3>{{ chain.name }}</h3>
        </v-col>
        <v-col md="1" :key="`${index}`" v-else>
          <h3>L {{ chain }}</h3>
        </v-col>
      </template>
    </v-row>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      evolution: null
    };
  },
  props: {
    pokemon: Object,
  },
  mounted() {
    this.fetchEvolution();
  },
  methods: {
    fetchEvolution() {
      axios
        .get(`https://pokeapi.co/api/v2/pokemon-species/${this.pokemon.id}`)
        .then((response) => {
          axios.get(response.data.evolution_chain.url).then((response) => {
            this.evolution = response.data.chain;
          });
        });
    },
    evolutions() {
      let chain = [];
      if (this.evolution) {
        let evolution = this.evolution;
        if (evolution.species) {
          chain.push(evolution.species);
        }

        while (evolution.species) {
          if (evolution.evolves_to.length > 0) {
            evolution = evolution.evolves_to[0];

            if (
              evolution.evolution_details.length > 0 &&
              evolution.evolution_details[0].min_level
            ) {
              chain.push(evolution.evolution_details[0].min_level);
            }

            if (evolution.species) {
              chain.push(evolution.species);
            }
          } else {
            break;
          }
        }

        return chain;
      }
    },
    getId(pokemon) {
      return Number(pokemon.url.split("/")[6])
    },
  },
  watch: {
    pokemon() {
      this.fetchEvolution();
    }
  }
}
</script>

<style lang="scss" scoped>
.evolution {
  display: flex;
  margin: 0;
  justify-content: space-between;

  v-col {
    align-items: center;
  }
}
</style>
