<template>
  <div class="container">
    <Head>
      <title v-if="country">Detail of {{ slug }}</title>
      <meta name="description" content="Country details page" />
    </Head>

    <!-- Main Details -->
    <div class="details-card" v-if="country">
      <h1 class="text-center">{{ country.name?.common || "N/A" }}</h1>
      <div class="details-content ">
        <div class="flag-wrapper">
          <img
            v-if="country.flags"
            :src="country.flags.png"
            class="flag-image"
            alt="flag"
          />
        </div>
        <div class="details-text">
          <div><b>Native Name:</b> {{ nativeName || "N/A" }}</div>
          <div><b>Capital:</b> {{ country.capital?.[0] || "N/A" }}</div>
          <div><b>Population:</b> {{ country.population || "N/A" }}</div>
          <div><b>Region:</b> {{ country.region || "N/A" }}</div>
          <div><b>Sub-region:</b> {{ country.subregion || "N/A" }}</div>
          <div><b>Area:</b> {{ country.area || "N/A" }} km²</div>
          <div>
            <b>Country Code:</b>
            {{ country.idd?.root + country.idd?.suffixes?.[0] || "N/A" }}
          </div>
          <div><b>Languages:</b> {{ languages || "N/A" }}</div>
          <div><b>Currencies:</b> {{ currencyName || "N/A" }}</div>
          <div><b>Timezones:</b> {{ country.timezones?.join(", ") || "N/A" }}</div>
        </div>
      </div>
    </div>

    <!-- Neighboring Countries -->
    <div class="neighbor-countries">
      <h2>Neighboring Countries</h2>
      <div v-if="borderCountries.length" class="neighbor-grid">
        <div
          v-for="border in borderCountries"
          :key="border.cca3"
          class="neighbor-card"
        >
          <img :src="border.flags.png" class="neighbor-flag" alt="flag" />
          <p class="neighbor-name">{{ border.name?.common || "N/A" }}</p>
        </div>
      </div>
      <p v-else class="no-neighbors">No neighboring countries found.</p>
    </div>

    <!-- Loading State -->
    <p v-if="loading" class="loading-text">Loading...</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      slug: null,
      country: null,
      borderCountries: [],
      nativeName: null,
      languages: null,
      currencyName: null,
      loading: true, // Track loading state
    };
  },
  async mounted() {
    this.slug = this.$route.params.slug;
    await this.fetchCountryDetails();
  },
  methods: {
    async fetchCountryDetails() {
      try {
        this.loading = true; // Show loading state

        // Fetch the country details
        const countryRes = await fetch(
          `https://restcountries.com/v3.1/alpha/${this.slug}`
        );
        const countryData = await countryRes.json();
        this.country = countryData[0] || null;

        // Fetch border countries if available
        if (this.country?.borders?.length) {
          const borderRes = await fetch(
            `https://restcountries.com/v3.1/alpha?codes=${this.country.borders.join(
              ","
            )}`
          );
          this.borderCountries = await borderRes.json();
        }

        // Process additional details
        this.nativeName =
          this.country?.name?.nativeName &&
          Object.values(this.country.name.nativeName)[0]?.common;
        this.languages =
          this.country?.languages &&
          Object.values(this.country.languages).join(", ");
        const currency =
          this.country?.currencies && Object.values(this.country.currencies)[0];
        this.currencyName = currency ? currency.name : null;
      } catch (error) {
        console.error("Error fetching country data:", error);
      } finally {
        this.loading = false; // Hide loading state
      }
    },
  },
};
</script>

<style>
/* Container Styling */
.container {
  margin: 50px auto;
  max-width: 900px;
}

/* Main Details Styling */
.details-card {
  border: 1px solid #ddd;
  border-radius: 10px;
  padding: 20px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  background-color: #f9f9f9;
}

.details-content {
  display: flex;
  align-items: flex-start;
  gap: 30px;
  margin-bottom: 2rem;
}

.flag-wrapper {
  flex: 1;
  max-width: 40%;
}

.flag-image {
  width: 100%;
  height: auto;
  border-radius: 5px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.details-text {
  /* flex: 2; */
  text-align: left;
  line-height: 1.4;
}

/* Neighboring Countries Styling */
.neighbor-countries {
  margin-top: 40px;
  text-align: center;
}

.neighbor-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 20px;
  justify-items: center;
}

.neighbor-card {
  border: 1px solid #ddd;
  border-radius: 10px;
  padding: 10px;
  text-align: center;
  background-color: #fff;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.neighbor-flag {
  max-width: 100%;
  height: auto;
  border-radius: 5px;
}

.neighbor-name {
  margin-top: 10px;
  font-size: 14px;
  font-weight: bold;
}

/* Loading Text */
.loading-text {
  font-size: 18px;
  color: #555;
}
/* No Neighboring Countries Message */
.no-neighbors {
  font-size: 16px;
  color: #777;
  margin-top: 20px;
  font-style: italic;
  text-align: center;
}
</style>