<template>
  <div id="app">

    <!-- ========Inicio da Barra de navegação======== -->
    <div class="navbar-fixed">
      <nav>
        <div class="nav-wrapper red darken-3">
          <a href="#" class="brand-logo center">Pokédex</a>
          <form>
            <div class="input-field">
              <input id="search" type="search" v-model="busca" v-on:input="buscaPokemon();" />
              <label class="label-icon" for="search">
                <i class="material-icons">search</i>
              </label>
              <i class="material-icons">close</i>
            </div>
          </form>          
        </div>
      </nav>
    </div>
    <!-- ========Fim da Barra de navegação======== -->

    <!-- mensagem de erro -->
    <div class="row" v-if="erro">
      <div class="col s6 m6 l4 xl2 offset-s3 offset-m3 offset-l4 offset-xl5">
        <div class="card">
          <div class="card-image">
            <img src="pikachu_cry.jpg">
          </div>
          <div class="card-content center">
            <p><b>Serviço fora do ar!</b></p>
          </div>
        </div>
      </div>
    </div>      
    <!-- mensagem de erro -->


    <!-- ========Inicio Card Pokémons======== -->
    <div class="row">
      <div class="col s6 m6 l4 xl2" v-for="pokemons of listaDepokemons" v-bind:key="pokemons.id" tabindex="1">
        <div class="card modal-trigger" href="#modal1" v-on:click="infoPokemon(pokemons.name)" >
          <a class="float btn-floating btn halfway-fab waves-effect waves-light blue darken-2"><b>{{ pokemons.url | urlToId()}}</b></a>
          <div>
            <div class="card-image">
              <img
                class="activator"
                :src="imgPokemon(pokemons.url)"
                :title="pokemons.name"
                alt="Imagem frontal do Pokémon"
              />
            </div>
            <div class="card-content grey darken-2">
              <span
                class="card-title center-align white-text activator"
              >{{ pokemons.name | capitalizeFirstLetter() }}</span>
            </div>
          </div>
        </div>
      </div>
    </div>
    <!-- ========Fim Card Pokémons======== -->


    <!-- ========Inicio da Estrutura do Modal======== -->
    <div id="modal1" class="modal modal-fixed-footer" :class="pokemon.cor">
      <div id="modal-main" class="modal-content">
        <img class="responsive-img card-imagem" :src="pokemon.foto" />
        <div class="white box-interno">
          <h4 class="center-align">{{ pokemon.nome | capitalizeFirstLetter() }} <a class="right btn-floating btn-large waves-effect waves-light red"><b>{{ pokemon.id }}</b></a></h4>
          <p>
            <b>Peso:</b>
            <div class="chip">{{ pokemon.peso }} kg</div>
          </p>
          <p>
            <b>Altura:</b>
            <div class="chip">{{ pokemon.altura }} m</div>
          </p>
          <p>
            <b>Habilidades:</b>
          </p>
          <div
            class="chip"
            v-for="habilidade in pokemon.habilidades"
          >{{ habilidade.ability.name | capitalizeFirstLetter() }}</div>
          <p>
            <b>Tipo:</b>
          </p>
          <div
            class="chip"
            v-for="tipo in pokemon.tipo"
          >{{ tipo.type.name | capitalizeFirstLetter() }}</div>
          <p>
            <b>Eggs Group:</b>
          </p>
          <div
            class="chip"
            v-for="eggs in pokemon.grupo_ovo"
          >{{ eggs.name | capitalizeFirstLetter() }}</div>
          <p>
            <b>Movimentos:</b>
          </p>
          <div class="chip"
            v-for="movimentos in pokemon.movimentos"
          >{{ movimentos.move.name | capitalizeFirstLetter() }}</div>
        </div>
      </div>
      <div class="modal-footer darken-4" :class="pokemon.cor">
        <a href="#!" class="modal-close waves-effect waves-green btn-flat white-text">Fechar</a>
      </div>
    </div>
    <!-- =====Fim da Estrutura do Modal======== -->
  </div>
</template>

<script>
// import axios from 'axios'
import PokemonApi from "./services/pokeapi";

export default {
  data() {
    return {
      listaGeralPokemon: [],
      listaDepokemons: [],
      next: 20,
      busca: null,
      erro: false,
      pokemon: {
        id: null,
        habilidades: [],
        foto: null,
        nome: '',
        peso: null,
        altura: null,
        tipo: [],
        grupo_ovo: [],
        movimentos: [],
        cor: null
      }
    };
  },

  mounted() {
    this.listar();
  },

  filters: {
    capitalizeFirstLetter(string) {
      return string.replace(/\w\S*/g, function(txt) {
        return txt.charAt(0).toUpperCase() + txt.substr(1).toLowerCase();
      });
    },
    
    urlToId(url){
      return url.substring(34, url.length - 1)
    }
  },

  methods: {
    listar(next) {
      PokemonApi.listar(next).then(resposta => {
        this.listaGeralPokemon = resposta.data.results;
        this.listaDepokemons = this.listaGeralPokemon;
      })
      .catch(error => {
        console.log(error);
        this.erro = true;
      });
    },

    infoPokemon(name) {
      PokemonApi.pokemonGeral(name).then(resposta => {
        this.pokemon.habilidades = resposta.data.abilities;
        this.pokemon.foto = resposta.data.sprites.front_default;
        this.pokemon.nome = resposta.data.name;
        this.pokemon.id = resposta.data.id;
        this.pokemon.peso = resposta.data.weight / 10;
        this.pokemon.altura = resposta.data.height / 10;
        this.pokemon.tipo = resposta.data.types;
        this.pokemon.movimentos = resposta.data.moves;
        // console.log(this.pokemon.movimentos);
        // console.log(this.foto);
        // console.log(this.nome);
      }),
      PokemonApi.especie(name).then(resposta => {
        this.pokemon.grupo_ovo = resposta.data.egg_groups;
        this.pokemon.cor = resposta.data.color.name;
      });
    },

    imgPokemon(url) {
      return (
        "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/" +
        url.substring(34, url.length - 1) +
        ".png"
      );
    },

    // nextPokemons() {
    //   this.listaDepokemons.push(
    //     ...this.listaGeralPokemon.slice(this.next, this.next + 20)
    //   );
    //   this.next = this.next + 20;
    // },

    buscaPokemon() {
      this.listaDepokemons = this.listaGeralPokemon.filter((pokemon)=>{
        return pokemon.name.indexOf(this.busca) !== -1;
      });
      // console.table(this.listaDepokemons);
    }
  }
};
</script>

<style>
</style>
