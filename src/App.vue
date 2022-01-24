<template>
  <div id="app">
    <div class="container">

      <!-- Title -->
      <h1 class="title">
        Компонент випадаючого списка з функціоналом автокомпліта
      </h1>

      <!-- Subtitle -->
      <h2 class="subtitle">
        Список героїв фільму Зоряні Війни
      </h2>

      <!-- Custom component -->
      <custom-select
          class="custom-select"
          v-if="values.data.length"
          :values="values"
      />

    </div>
  </div>
</template>

<script>
import CustomSelect from "./components/CustomSelect";

export default {
  name: 'App',
  data() {
    return {
      dataValues: [],
      values: {
        data : [],
        numberOfVisibleValues: 20,
        placeholder: 'Введіть імя героя ...'
      }
    }
  },
  components: {
    CustomSelect
  },

  async created() {
    await this.loadHeroesNameData()
    await this.makeRepeated()
  },

  methods: {

    // Load data array
    async loadHeroesNameData() {
      try {
        let newDataArray =[]

        for (let num = 1; num <= 9; num++) {
          let response = await fetch(`https://swapi.dev/api/people/?page=${num}`)
          let chaptersData = await response.json()
          newDataArray.push(chaptersData.results.map(i => i.name))
        }

        this.dataValues = newDataArray.flat()
      } catch (error) {
        console.log(error)
      }
    },

    // Make repeated of values
    makeRepeated() {
      let makeRepeated = (arr, repeats) =>
          [].concat(...Array.from({length: repeats}, () => arr))

      this.values.data = makeRepeated(this.dataValues, 400)
    }

  }
}
</script>

<style>
.title,
.subtitle {
  text-align: center;
  font-weight: bold;
  line-height: 120%;
}
.title {
  padding-top: 40px;
  font-size: 40px;
  margin-bottom: 30px;
}

.subtitle {
  font-size: 32px;
  margin-bottom: 100px;
}

.custom-select {
  margin: 0 auto;
}
</style>
