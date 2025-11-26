<template>
  <div class="burger">
    <h3>{{ burger.name }}</h3>
    <img v-bind:src="burger.url" v-bind:alt="burger.name" width="300" height="300">
    <p>Ordered: {{ amountOrdered }}</p>
    <div>
      <button v-on:click="increaseOrder">+</button>
      <button v-on:click="decreaseOrder">-</button>
    </div>
    <ul>
      <li v-if="burger.gluten">Contains <span class="allergen">gluten</span></li>
      <li v-if="burger.lactose">Contains <span class="allergen">dairy</span></li>
    </ul>
  </div>
</template>

<script>
export default {
  name: 'OneBurger',
  props: {
    burger: Object
  },
  data: function () {
    return {
      amountOrdered: 0,
    }
  },
  methods: {
    increaseOrder: function () {
      this.amountOrdered += 1;
      this.$emit('orderedBurger', { 
        name: this.burger.name,
        amount: this.amountOrdered 
      });
    },
    decreaseOrder: function () {
      if (this.amountOrdered > 0) {
        this.amountOrdered -= 1;
        this.$emit('orderedBurger', { 
          name: this.burger.name,
          amount: this.amountOrdered 
        });
      }
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.burger {
  padding: 10px;
}
.allergen {
  font-weight: bold;
  text-transform: uppercase;
}
.burger p {color: white;}
</style>