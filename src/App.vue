<template>
  <div id="app">
    <Header
      :primary-nav="HeaderData.primaryItems"
      :secondary-nav="HeaderData.secondaryItems"
      :museum-info="HeaderData.museumInfo"
      :has-search="HeaderData.hasSearch"
      :force-on-dark="false"
      :baseUrl="'https://www.getty.edu'"
    />
    <Hero name="Content" :data="heroData" />
    <div
      :style="{ backgroundImage: `url('${heroImg}')` }"
      class="heroImage"
    ></div>

    <section>
      <div class="container">
        <router-view></router-view>
      </div>
    </section>
    <SearchOverlay />
    <Footer
      :baseUrl="'https://www.getty.edu'"
      :primary-nav="FooterData.primaryNavigation"
      :museum-info="FooterData.museumInfo"
      :nav-grid="FooterData.navGrid"
      :social="FooterData.social"
      :social-all-link="FooterData.socialAllLink"
    />
  </div>
</template>

<script>
import "@thegetty/getty-ui/dist/getty-ui.css";
import HeaderData from "@thegetty/getty-ui/src/components/globals/header/data.json";
import FooterData from "@thegetty/getty-ui/src/components/globals/footer/data.json";
import {
  Header,
  Footer,
  Hero,
  RichText,
  SearchOverlay,
} from "@thegetty/getty-ui";
import heroImg from "/assets/images/IMG_7794_edit.jpg";
export default {
  name: "App",
  components: {
    Header,
    Hero,
    Footer,
    SearchOverlay,
  },
  data: function () {
    return {
      HeaderData: HeaderData,
      FooterData: FooterData,
      heroImg,
      heroData: {
        title: "Animal Crossing Art Generator",
        backgroundColor: "white",
      },
    };
  },
};
</script>

<style lang="scss">
.container {
  width: 100%;
  max-width: 1800px;
  margin-left: auto;
  margin-right: auto;
  padding: 0 36px;
}
@media (max-width: 767px) {
  .container {
    padding: 0 16px;
  }
}
body {
  font-size: 15px;
  color: #1a1a1a;
}
p {
  font-size: 15px;
  margin-top: 12px !important;
}
li {
  font-size: 15px;
}
h1 {
  font-size: 1.867em;
  font-weight: 600;
  margin-top: 12px;
}
h2 {
  font-size: 1.4em;
  font-weight: 500px;
  marin-top: 24px;
}

h3 {
  font-size: 1.2 em;
  font-weight: 400;
  margin-top: 40px;
}
a {
  color: #1a47b8;
  cursor: pointer;
}
.heroImage {
  background-position: center;
  background-size: cover;
  height: 380px;
  width: 100%;
}
.o-hero--content .o-hero__inner {
  padding-top: inherit !important;
  height: inherit !important;
}

.g-header__inner a {
  color: black;
}
</style>
