<style scoped>
/* @import "./css/normalize.css"; */
@import "./css/webflow.css";
@import "./css/mydentist-apps.css";
</style>

<template>
  <div v-if="loadedReviews" class="review-score-wrapper badge">
    <div class="text-review-score">
      {{ getRatingFromCity() ? getRatingFromCity() : getRatingAllClinics() }}
    </div>

    <div class="star-wrapper">
      <star class="review-star" />
      <star class="review-star" />
      <star class="review-star" />
      <star class="review-star" />
      <star class="review-star" />
      <div class="draperi" :style="{ left: starClipping }"></div>
    </div>
  </div>
</template>

<script>
import star from "./images/star.vue";

export default {
  name: "App",
  components: { star },

  data() {
    return {
      apiBaseUrl: "https://api.ngine.se/webhook/mydentist/",
      getClinics: "get-clinics",
      userName: "XkehuCfMZ!hU%8h=",
      userPass: "QH5EV=2hNc*LFjJd",
      listReviews: [],
      loadedReviews: false,
    };
  },

  async created() {
    this.listClinics = await this.getApiData(this.apiBaseUrl + this.getClinics);
    this.loadedReviews = true;
  },

  computed: {
    starClipping() {
      return (
        ((this.getRatingFromCity()
          ? this.getRatingFromCity()
          : this.getRatingAllClinics()) /
          5) *
        100
      ).toFixed(3);
    },
  },

  methods: {
    getApiData(urlEndpoint) {
      return new Promise((resolve, reject) => {
        var requestOptions = {
          method: "GET",
          headers: {
            Authorization: "Basic " + btoa(this.userName + ":" + this.userPass),
          },
          redirect: "follow",
        };

        fetch(urlEndpoint, requestOptions)
          .then((response) => {
            if (!response.ok) throw new Error();
            return response.json();
          })
          .then((result) => {
            resolve(result);
          })
          .catch((error) => {
            console.log(error);

            reject(error);
          });
      });
    },

    getCityFromUrl() {
      const url = decodeURI(window.location.pathname);

      for (const clinic of this.listClinics.data) {
        const cityFormatted = clinic.attributes.clinic_city
          .normalize("NFD")
          .replace(/[^a-z\s]/gi, "")
          .toLowerCase();

        for (const pathname of url.split("/")) {
          if (pathname === cityFormatted) {
            return clinic.attributes.clinic_city;
          }
        }
      }

      return null;
    },

    getRatingFromCity() {
      const city = this.getCityFromUrl();

      if (city) {
        for (const clinic of this.listClinics.data) {
          if (clinic.attributes.clinic_city === city) {
            return clinic.attributes.overall_rating.toFixed(1);
          }
        }
      }

      return null;
    },

    getRatingAllClinics() {
      const ratings = [];
      let sum = 0;

      for (const clinic of this.listClinics.data) {
        ratings.push(parseFloat(clinic.attributes.overall_rating));
      }

      for (const rating of ratings) {
        sum += rating;
      }

      return (sum / this.listClinics.data.length).toFixed(1);
    },
  },
};
</script>
