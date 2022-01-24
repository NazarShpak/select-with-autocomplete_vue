<template>
  <div class="custom-select">

    <label class="custom-select__label">

      <!-- Input value -->
      <input
          class="custom-select__input"
          v-model.trim="selectValue"
          type="text"
          name="value"
          :placeholder="placeholder"

          @input="openBlock = true; filterValues = true; resetValue()"
          @keyup.up.prevent="prevValue"
          @keydown.down.prevent="nextValue"
          @keyup.enter="changeSelectValue()"
          @click="openBlock = !openBlock"

          autocomplete="off"
          :class="!openBlock ? 'open-block' : 'close-block'"
      >

      <!-- Clean value -->
      <img
          alt
          class="custom-select__img"
          src="../assets/icons/clean-value.svg"
          v-if="selectValue"
          @click="selectValue = ''"
      >
    </label>

    <!-- All values list -->
    <div
        class="custom-select__block"
        v-if="openBlock"
    >

      <ul class="custom-select__list"
          ref="list"
          v-if="generalDataValues"
          @wheel="scrollList"
      >

        <li
            class="custom-select__item"
            v-for="(name, index) of generalDataValues"
            :key="index"
            :ref="getRef(index)"
            :class="{
              'select' : index === indexOfActiveValue - indexOfFirstElement,
              'active' : name === selectValue
            }"
            @click="changeSelectValue(name)"
        >
          {{ name }}
        </li>
      </ul>

    </div>

  </div>
</template>

<script>
export default {
  name: "CustomSelect",

  data() {
    return {
      placeholder: '',
      generalData: [],
      selectValue: '',
      filterValues: false,
      indexOfActiveValue: 0,
      numberOfVisibleValues: 0,
      openBlock: false,
      scrollActive: false,
    }
  },

  props: {
    values: {
      type: Object
    }
  },

  created() {
    this.placeholder = this.values.placeholder
    this.generalData = this.values.data
    this.numberOfVisibleValues = this.values.numberOfVisibleValues

    // Closed all values list when click outside the component
    window.addEventListener('click', (e) => {
      if (!this.$el.contains(e.target)) {
        this.openBlock = false
      }
    })
  },

  computed: {

    // The average value of the number of visible values
    averageValue() {
      return this.numberOfVisibleValues / 2
    },


    // Index of the first element of the array
    indexOfFirstElement() {

      // For scroll by buttons
      if (!this.scrollActive) {
        let index = this.indexOfActiveValue - this.averageValue

        if (this.indexOfActiveValue <= this.averageValue - 1) {
          return 0
        } else {
          return index
        }

        // For scroll by mouse
      } else {

        if (this.indexOfActiveValue > 0) {
          return this.indexOfActiveValue - 1
        } else {
          return this.indexOfActiveValue
        }

      }
    },

    // Index of the last element of the array
    indexOfLastElement() {
      let index = this.indexOfActiveValue + this.averageValue

      if (index < this.generalData.length - 1) {
        return index
      } else {
        return this.generalData.length - 2
      }
    },

    // Filtered data array for autocomplete
    filterGeneral() {
      let newArray = []
      let value = this.selectValue.toLowerCase()

      for (let val of this.generalData) {
        if (val.toLowerCase().includes(value)) {
          if (!newArray.includes(val)) {
            newArray.push(val)
          }
        }
      }

      return newArray
    },

    // Returns the final data array depending on the usage condition
    generalDataValues() {
      if (this.filterValues) {
        return this.filterGeneral.slice(this.indexOfFirstElement, this.indexOfLastElement)
      } else {
        return this.generalData.slice(this.indexOfFirstElement, this.indexOfLastElement)
      }
    },

  },

  watch: {
    openBlock: {
      handler() {
        this.scroll()
      }
    },
  },

  methods: {

    // Scroll list by mouse
    scrollList() {
      const list = this.$refs.list
      this.scrollActive = true

      if (list.scrollTop + list.clientHeight >= list.scrollHeight - 1 && this.indexOfLastElement < this.generalData.length - 1) {
        this.nextPrevValue('next')
      } else if (list.scrollTop === 0 && this.indexOfActiveValue - this.numberOfVisibleValues > 0) {
        this.nextPrevValue('prev')
      }
    },

    // Load next or prev values
    nextPrevValue(value) {
      const val = this.numberOfVisibleValues + 1

      if (value === 'next') {
        this.indexOfActiveValue += val
      } else {
        this.indexOfActiveValue -= val
      }
      this.scroll(20)
    },

    // Change selected value
    changeSelectValue(value) {
      if (value) {
        this.selectValue = value
        this.filterValues = false
      } else {

        if (this.filterValues) {
          this.selectValue = this.filterGeneral[this.indexOfActiveValue]
        } else {
          this.selectValue = this.generalData[this.indexOfActiveValue]
        }

        this.indexOfActiveValue = this.generalData.indexOf(this.selectValue)
        this.filterValues = false
      }

      this.openBlock = false
    },

    // Reset value
    resetValue() {
      this.indexOfActiveValue = 0
    },

    // Prev value
    prevValue() {
      this.scrollActive = false
      this.openBlock = true

      if (this.indexOfActiveValue > 0) {
        this.indexOfActiveValue--
        this.scroll(120)
      }
    },

    // Next value
    nextValue() {
      this.scrollActive = false
      this.openBlock = true

      if (this.filterValues && this.indexOfActiveValue < this.filterGeneral.length - 1) {
        this.indexOfActiveValue++
      } else if (!this.filterValues && this.indexOfLastElement < this.generalData.length - 1) {
        this.indexOfActiveValue++
      }

      this.scroll(120)
    },

    // Get ref for active element
    getRef(value) {
      if (value === this.indexOfActiveValue - this.indexOfFirstElement) {
        return 'activeItem'
      }
    },

    // Scroll list
    scroll(value) {
      if (this.openBlock) {
        setTimeout(() => {
          let heightFromEl = this.$refs.activeItem[0].offsetTop
          this.$refs.list.scroll({top: heightFromEl - value})
        }, 0)
      }
    },
  }
}
</script>

<style lang="scss" scoped>
.custom-select {
  position: relative;
  width: 500px;

  &__label {
    position: relative;
    width: 100%;
  }

  &__input,
  &__input::placeholder {
    width: 100%;
    font-weight: 300;
    font-size: 24px;
    line-height: 160%;
    letter-spacing: 0.02em;
    color: rgba(0, 0, 0, 0.4);
  }

  &__input {
    height: 60px;
    font-weight: 500;
    color: #000000;
    border: 1px solid rgba(0, 0, 0, 0.4);
    padding: 5px 10px;
    box-shadow: 0 24px 32px rgba(0, 0, 0, 0.04), 0 16px 24px rgba(0, 0, 0, 0.04), 0 4px 8px rgba(0, 0, 0, 0.04), 0 0 1px rgba(0, 0, 0, 0.04);
  }

  .open-block,
  .close-block {
    background-repeat: no-repeat;
    background-position: 460px center;
    background-size: 20px 10px;
  }

  .open-block {
    background-image: url("../assets/icons/arrow-down.svg");
  }

  .close-block {
    background-image: url("../assets/icons/arrow-up.svg");
  }

  &__img {
    position: absolute;
    top: -4px;
    right: 50px;
    cursor: pointer;
  }

  &__datalist {
    display: none;
  }

  &__block {
    height: 300px;
    width: 500px;
    background: #F8F8FA;
    box-shadow: 0 24px 32px rgba(0, 0, 0, 0.04), 0 16px 24px rgba(0, 0, 0, 0.04), 0 4px 8px rgba(0, 0, 0, 0.04), 0 0 1px rgba(0, 0, 0, 0.04);
    border-radius: 10px;
    position: absolute;
    top: 90px;
    left: 0;
    z-index: 200;
    display: flex;
    flex-direction: column;
    padding: 20px 10px 20px 21px;
  }

  &__list {
    overflow-y: auto;
  }

  &__list::-webkit-scrollbar {
    width: 3px;
    border-radius: 5px;
    margin-right: 0;
    background-color: #E4E4E4;
  }

  &__list::-webkit-scrollbar-thumb {
    margin-right: 20px;
    height: 50px;
    background: #C6C6C6;
    border-radius: 5px;
  }

  &__list::-webkit-scrollbar-button {
    display: none;
  }

  &__item {
    font-weight: 500;
    font-size: 24px;
    line-height: 34px;
    cursor: pointer;
  }

  &__item:hover {
    color: #f17a7a;
  }

  &__item + &__item {
    margin-top: 20px;
  }

  .active {
    color: #FF0000;
  }

  .select {
    color: #f17a7a;
  }
}
</style>