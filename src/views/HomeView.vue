<template>
  <div>
    <header>
      <img
        src="https://t4.ftcdn.net/jpg/07/78/23/41/360_F_778234167_Wcm57Vq9GivIZLW5Qd3p2OUBQ0JGOXS1.jpg"
      />
      <h1>Welcome to BurgerHeaven Online</h1>
    </header>
    <main>
      <section id="burger-selection">
        <h2>Select burger</h2>
        <p>This is where you execute burger selection</p>
        <div class="burger-wrapper">
          <Burger
            v-for="burger in burgers"
            v-bind:burger="burger"
            v-bind:key="burger.name"
            v-on:orderedBurger="addToOrder($event)"
          />
        </div>
      </section>

      <section id="customer-information">
        <h2>Customer information</h2>
        <p>This is where you provide necessary information</p>

        <h3>Delivery information:</h3>

        <form>
          <p>
            <label for="name">Full name</label><br />
            <input
              type="text"
              id="name"
              v-model="name"
              required
              placeholder="First- and Last name"
            />
          </p>
          <p>
            <label for="email">E-mail</label><br />
            <input
              type="email"
              id="email"
              v-model="email"
              required
              placeholder="E-mail address"
            />
          </p>

          <p>
            <label>Gender:</label><br />
            <input
              type="radio"
              id="male"
              v-model="gender"
              value="male"
              checked
            />
            <label for="male">Male</label><br />

            <input type="radio" id="female" v-model="gender" value="female" />
            <label for="female">Female</label><br />

            <input
              type="radio"
              id="no-gender"
              v-model="gender"
              value="no-gender"
            />
            <label for="no-gender">Do not wish to provide</label>
          </p>
          <p>
            <label for="payment">Payment method</label><br />
            <select id="payment" v-model="payment">
              <option selected="selected">Card</option>
              <option>Klarna</option>
              <option>Pay at pickup</option>
              <option>Apple pay</option>
            </select>
          </p>
        </form>
      </section>
      <section id="map-section">
        <h2>Delivery Map</h2>
        <div class="map-container">
          <img
            id="map"
            src="/img/polacks.jpg"
            alt="Delivery Map"
            width="1920"
            height="1078"
            @click="setLocation"
          />
          <div
            class="target"
            :style="{ left: location.x + 'px', top: location.y + 'px' }"
          >
            T
          </div>
        </div>
      </section>

      <button type="button" v-on:click="submitOrder">
        <img
          src="https://img.myloview.se/fototapeter/check-icon-vector-check-mark-icon-check-list-button-icon-700-194930748.jpg"
          alt="Submit"
          width="20"
          height="20"
        />
        Place order
      </button>
    </main>

    <footer>
      <p>Copyright BurgerHeaven Online 2025</p>
    </footer>
  </div>
</template>

<script>
import menu from "../assets/menu.json";
import Burger from "../components/OneBurger.vue";
import io from "socket.io-client";

const socket = io("localhost:3000");

// 1. Create Menultem object constructor function
function Menultem(name, url, kCal, hasGluten, hasLactose) {
  this.name = name;
  this.url = url;
  this.kCal = kCal;
  this.gluten = hasGluten;
  this.lactose = hasLactose;
}

const burgers = [
  new Menultem(
    "Double burger",
    "https://images.arla.com/recordid/02F90FAA-0208-47E9-A76D3510DDFDA427/smashburger.jpg",
    300,
    false,
    true
  ),
  new Menultem(
    "Big burger",
    "https://images.themodernproper.com/production/posts/2016/ClassicCheeseBurger_9.jpg?w=1200&h=1200&q=60&fm=jpg&fit=crop&dm=1749310239&s=463b18fc3bb51dc5d96e866c848527c4",
    400,
    true,
    false
  ),
  new Menultem(
    "Chicken burger",
    "https://www.recipetineats.com/tachyon/2023/09/Crispy-fried-chicken-burgers_5.jpg",
    500,
    true,
    true
  ),
];

// 3. Print the array to console
console.log(burgers);

export default {
  name: "HomeView",
  components: {
    Burger,
  },
  data: function () {
    return {
      burgers: menu,
      name: "",
      email: "",
      gender: "male",
      payment: "Card",
      orderedBurgers: {},
      location: { x: 0, y: 0 },
    };
  },
  methods: {
    getOrderNumber: function () {
      return Math.floor(Math.random() * 100000);
    },
    addOrder: function (event) {
      var offset = {
        x: event.currentTarget.getBoundingClientRect().left,
        y: event.currentTarget.getBoundingClientRect().top,
      };
      this.location.x = event.clientX - 10 - offset.x;
      this.location.y = event.clientY - 10 - offset.y;
    },
    submitOrder: function () {
      console.log("Order submitted:");
      console.log("Name:", this.name);
      console.log("Email:", this.email);
      console.log("Gender:", this.gender);
      console.log("Payment method:", this.payment);
      console.log("Delivery location:", this.location);
      console.log("Ordered burgers (Proxy):", this.orderedBurgers);

      // Konvertera orderedBurgers objekt till array för orderItems
      const orderItems = [];
      for (const [burgerName, amount] of Object.entries(this.orderedBurgers)) {
        for (let i = 0; i < amount; i++) {
          orderItems.push(burgerName);
        }
      }

      socket.emit("addOrder", {
        orderId: this.getOrderNumber(),
        details: {
          x: this.location.x,
          y: this.location.y,
          customerName: this.name,
          customerEmail: this.email,
          customerGender: this.gender,
          paymentMethod: this.payment,
        },
        orderItems: orderItems,
      });
    },
    addToOrder: function (event) {
      if (event.amount > 0) {
        this.orderedBurgers[event.name] = event.amount;
      } else {
        delete this.orderedBurgers[event.name];
      }
      this.orderedBurgers = { ...this.orderedBurgers };
    },
    setLocation: function (event) {
      var offset = {
        x: event.currentTarget.getBoundingClientRect().left,
        y: event.currentTarget.getBoundingClientRect().top,
      };
      this.location.x = event.clientX - 10 - offset.x;
      this.location.y = event.clientY - 10 - offset.y;

      console.log("Location set to:", this.location.x, this.location.y);
    },
  },
};
</script>

<style>
#map-section {
  margin: 20px;
  border: 2px dashed;
  padding: 20px;
}

.map-container {
  width: 500px;
  height: 300px;
  overflow: auto;
  border: 1px solid #ccc;
  position: relative;
}

#map {
  cursor: pointer;
}
.target {
  position: absolute;
  background: red;
  border-radius: 50%;
  width: 20px;
  height: 20px;
  text-align: center;
  line-height: 20px;
  color: white;
  font-weight: bold;
}

@import url("https://fonts.googleapis.com/css2?family=Agbalumo&family=Cormorant:wght@700&display=swap");
body {
  font-size: 12pt;
  font-family: "comic sans", sans-serif;
}

h1 {
  font-family: "Times New Roman", Times, serif;
  font-size: 36pt;
}
main,
header,
footer,
nav ul {
  max-width: 80rem;
  margin: 0 auto 0 auto;
}
main {
  background-color: bisque;
}

header h1 {
  width: 40rem;
  margin: 0 auto;
  text-align: center;
}

nav ul {
  display: grid;
  grid-template-columns: repeat(auto-fill, 9.25em);
  gap: 1em;
  padding: 0;
}

nav li {
  display: block;
  background-color: grey;
  padding: 1em;
}

.Very-good {
  color: green;
}

.Master {
  color: green;
  font-weight: bold;
}
.allergen {
  font-weight: bold;
  text-transform: uppercase;
}
#burger-selection {
  background-color: black;
  color: white;
  border-color: white;
}
#customer-information {
  border-color: black;
  color: black;
}
button:hover {
  background-color: #1287b6;
  cursor: pointer;
}
section {
  margin: 20px;
  border: 2px dashed;
  padding: 20px;
}
button {
  margin: 20px;
  padding: 10px 20px;
}
.burger {
  padding: 10px;
}
header {
  position: relative;
  height: 300px;
  overflow: hidden;
  margin: 0 20px;
  max-width: 80rem; /* SAMMA som dina andra element */
  margin-left: auto; /* Centrera */
  margin-right: auto; /* Centrera */
}

header img {
  width: 100%;
  height: auto;
  opacity: 0.8; /* Justera opaciteten för att göra bilden lite genomskinlig */
}

header h1 {
  position: absolute;
  top: 50%; 
  left: 50%; 
  transform: translate(-50%, -50%);
  margin: 0; /* Ta bort negativ margin */
  padding: 20px;
  color: white; 
  text-align: center;
  width: 100%; 
}
.burger-wrapper {
  display: grid;
  grid-gap: 5px;
  grid-template-columns: 1fr 1fr 1fr; 
}
</style>
