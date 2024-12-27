<template>
  <div class="container">
    <Head>
      <title>Countries</title>
      <meta name="description" content="Countries list" />
      <link
        href="https://cdn.jsdelivr.net/npm/bootstrap@5.2.1/dist/css/bootstrap.min.css"
        rel="stylesheet"
        integrity="sha384-iYQeCzEYFbKjA/T2uDLTpkwGzCiq6soy8tYaI1GyVh/UjpbCx/TYkiZhlZB6+fzT"
        crossorigin="anonymous"
      />
      <link
        href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0-beta3/css/all.min.css"
        rel="stylesheet"
      />
    </Head>

    <h1 class="title">Countries</h1>

    <div>
      <!-- Search Bar -->
      <div class="search-bar">
        <div class="input-wrapper">
          <input
            v-model="searchTerm"
            @input="onSearch"
            type="text"
            class="form-control"
            placeholder="Search countries"
          />
          <i class="fas fa-search search-icon" @click="onSearch"> </i>
        </div>
      </div>

      <!-- Cards Section -->
      <div class="cards">
        <div
          v-for="(country, index) in data"
          :key="index"
          class="card custom-card"
          v-if="Array.isArray(data)"
        >
          <div class="row">
            <!-- Flag Section -->
            <div class="col-md-4 ms-md-2 ms-0 col-12">
              <img :src="country.flags.png" alt="Flag" class="flag-image" />
            </div>

            <!-- Country Details Section -->
            <div class="col-md-7 col-12">
              <div class="card-body">
                <b class="card-title">{{ country.name.common }}</b>
                <div class="card-details">
                  Currency: {{ getCurrency(country) }}
                </div>
                <div class="card-details">
                  Current date and time: {{ calcTime(country.timezones) }}
                </div>
                <div class="button-group">
                  <button
                    class="btn btn-primary"
                    @click="openMap(country.maps.googleMaps)"
                  >
                    Show Map
                  </button>
                  <NuxtLink
                    :to="`/details/${country.cca3}`"
                    class="btn btn-secondary"
                  >
                    Detail
                  </NuxtLink>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      data: null,
      searchTerm: "",
      allCountries: [],
    };
  },
  async mounted() {
    await this.fetchCountries();
  },
  methods: {
    async fetchCountries() {
      try {
        const response = await fetch("https://restcountries.com/v3.1/all");
        if (!response.ok) throw new Error("Failed to fetch countries");
        this.allCountries = await response.json();
        this.data = this.allCountries;
      } catch (error) {
        console.error(error);
        this.data = "Error fetching data.";
      }
    },
    calcTime(timezones) {
      if (!timezones || !timezones[0]) return "N/A";

      // Parse the offset from the timezone string
      const offset = parseInt(timezones[0].substring(3), 10);

      // Calculate UTC time in milliseconds
      const utc = new Date().getTime() + new Date().getTimezoneOffset() * 60000;

      // Adjust UTC time by the timezone offset
      const localTime = utc + offset * 3600000;

      // Format the date and time
      const options = {
        day: "numeric",
        month: "short",
        year: "numeric",
        hour: "numeric",
        minute: "2-digit",
        hour12: true, // Ensure AM/PM format
      };

      return new Date(localTime).toLocaleString("en-US", options);
    },
    getCurrency(country) {
      if (!country.currencies) return "N/A";
      const currencyKeys = Object.keys(country.currencies);
      return currencyKeys.length
        ? country.currencies[currencyKeys[0]].name
        : "N/A";
    },
    onSearch() {
      if (this.searchTerm) {
        this.data = this.allCountries.filter((country) =>
          country.name.common
            .toLowerCase()
            .includes(this.searchTerm.toLowerCase())
        );
        if (!this.data.length) this.data = "No countries found.";
      } else {
        this.data = this.allCountries;
      }
    },
    openMap(url) {
      window.open(url, "_blank");
    },
  },
};
</script>

<style scoped>
.container {
  padding: 20px;
  font-family: Arial, sans-serif;
}

.title {
  text-align: center;
  font-size: 2.5rem;
  margin-bottom: 20px;
  font-weight: bold;
}

.search-bar {
  display: flex;
  justify-content: center;
  margin-bottom: 30px;
}

.input-wrapper {
  position: relative;
  width: 50%;
}

.form-control {
  width: 100%;
  padding: 10px 40px 10px 15px;
  font-size: 1rem;
  border: 1px solid #ccc;
  border-radius: 5px;
}

.search-icon {
  position: absolute;
  right: 15px;
  top: 50%;
  transform: translateY(-50%);
  font-size: 1.2rem;
  color: #888;
  cursor: pointer;
}

.cards {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
}

.custom-card {
  width: 50%;
  /* max-width: 500px; */
  border: 1px solid #ddd;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  overflow: hidden;
  background-color: #fff;
  transition: transform 0.2s ease;
}

.custom-card:hover {
  transform: translateY(-5px);
}

.flag-image {
  width: 100%;
  height: 150px;
  object-fit: cover;
}

/* .card-body {
  padding: 15px;
} */

.card-title {
  font-size: 1rem;
}

.card-details {
  font-size: 14px;
}

.button-group {
  margin-top: 4px;
  display: flex;
  justify-content: space-between;
}

.button-group button,
.button-group a {
  font-size: 14px;
  font-weight: 800;
  padding: 7px 20px;
  /* margin: 5px; */
  border: 2px solid #0501ff;
  background-color: white;
  color: #0501ff;
  cursor: pointer;
  border-radius: 4px;
}

@media (max-width: 749.5px) {
  .button-group button,
  .button-group a {
    padding: 7px 12px !important;
  }
}

.no-data {
  text-align: center;
  font-size: 1.5rem;
  color: #555;
}
.custom-card .row {
  display: flex;
  align-items: center;
}

.flag-image {
  width: 100%;
  height: auto;
  object-fit: contain;
}

/* .card-body {
  padding: 15px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
} */
</style>
