<template>
  <div>
    <div class="content">
      <!-- Introduction -->
      <section class="section">
        <h3 class="f-heading-3">
          Add art to your game using the Animal Crossing Art Generator
        </h3>
        <div class="l-thirds">
          <div class="l-thirds__two-thirds">
            <RichText :content="introText" contentType="markdown" />
          </div>
        </div>
      </section>
      <hr />

      <!-- Step 1 -->
      <section class="section">
        <h1 id="step1">
          Step 1: Select an artwork
        </h1>
        <div class="l-thirds">
          <div class="l-thirds__two-thirds">
            <RichText :content="step1Text" contentType="markdown" />
          </div>
        </div>

        <Search @input="onSearchSelect" ref="search" />

        <h3 class="f-heading-3">Our Favorites</h3>
        <Gallery @selectedExample="loadFromExample" ref="gallery" />
      </section>

      <hr />

      <section class="section">
        <div class="l-halves">
          <!-- Step 2 -->
          <div class="l-halves__half">
            <h1 id="step2" ref="step2">
              Step 2: Select the crop for your artwork
            </h1>
            <div class="column-content">
              <ImageLoader
                :pattern-type="patType"
                :iiif-url="iiif.url"
                @converted="onConvert"
                ref="imageloader"
              />
              <MetadataFields
                :title="searchResult.full_name"
                :artist="searchResult.artist"
                :collection-link="searchResult.webpage"
                :attribution="searchResult.attribution"
                :license="searchResult.license"
              />
            </div>
          </div>
          <!-- Step 3 -->
          <div class="l-halves__half leftborder">
            <h1>
              Step 3: Import your artwork into Animal Crossing
            </h1>
            <div class="column-content">
              <div class="generated-image">
                <ACNLQRGenerator :pattern="qrCode" />
                <span class="save-button" @click="downPNG">
                  <a>Download artwork and QR code </a
                  ><object class="save-icon" :data="saveIcon"></object>
                </span>
              </div>
              <RichText :content="qrInstructions" contentType="markdown" />
            </div>
          </div>
        </div>
      </section>
      <hr />

      <!-- Step 4: Share -->
      <section class="section">
        <h1>Step 4: Share with us</h1>
        <Share />
      </section>
      <hr />

      <!-- IIIF section -->
      <section class="section">
        <div id="iiifloader">
          <h1>Use the art generator with other open-access IIIF images</h1>
          <IIIFInput @updateIiif="updateIiifData" :iiif-error="iiif_error" />
        </div>
      </section>
      <hr class="hr-dark" />

      <!-- Credits -->
      <section class="section">
        <Credits />
        <Disclaimer />
      </section>

      <!-- hidden image used for QR code image -->
      <img
        id="gettylogo"
        class="hidden"
        :src="gettyLogo"
        style="border: 1px solid blue;"
      />
    </div>
  </div>
</template>

<script>
import Credits from "/components/Credits.vue";
import MetadataFields from "/components/MetadataFields.vue";
import Disclaimer from "/components/Disclaimer.vue";
import gettyLogo from "/assets/images/getty-logo.png";
import saveIcon from "/assets/images/save-icon.svg";
import introText from "../data/intro_text.md";
import step1Text from "../data/step1_text.md";
import Share from "/components/Share.vue";
import qrInstructions from "../data/qr_instructions.md";
import examples from "../data/example_images.json";
import { RichText, RecordMetadataField } from "@thegetty/getty-ui";
import IIIFInput from "/components/IIIFInput.vue";
import ImageLoader from "/components/ImageLoader.vue";
import Gallery from "/components/Gallery.vue";
import ACNLQRGenerator from "/components/ACNLQRGenerator.vue";
import Search from "/components/Search.vue";
import DrawingTool from "/libs/DrawingTool";
import origin from "/libs/origin";
import logger from "/utils/logger";
import lzString from "lz-string";
import { saveAs } from "file-saver";
import { getIIIFData } from "../libs/ExtractData";
import generateACNLQR from "/libs/ACNLQRGenerator";
if (typeof window !== "undefined") {
  let smoothscroll = require("smoothscroll-polyfill");
  smoothscroll.polyfill();
}

export default {
  name: "Editor",
  components: {
    Credits,
    MetadataFields,
    Disclaimer,
    IIIFInput,
    Search,
    Share,
    ImageLoader,
    Gallery,
    ACNLQRGenerator,
    RichText,
    RecordMetadataField,
  },
  beforeRouteUpdate: function(to, from, next) {
    if (to.hash.length > 1) {
      if (to.hash.startsWith("#H:")) {
        origin.view(to.hash.substring(3)).then((r) => {
          this.drawingTool.load(r);
        });
        next();
        return;
      }
      let newHash = lzString.compressToEncodedURIComponent(
        this.drawingTool.toString()
      );
      if (to.hash !== "#" + newHash) {
        this.drawingTool.load(
          lzString.decompressFromEncodedURIComponent(to.hash.substring(1))
        );
      }
    }
    next();
  },

  data: function() {
    return {
      gettyLogo,
      saveIcon,
      qrInstructions: qrInstructions,
      step1Text,
      introText,
      iiif: {},
      searchResult: {},
      drawingTool: new DrawingTool(),
      qrCode: false,
      allTypes: [],
      storedAuthorHuman: false,
      patInfoModal: false,
      fragment: "",
      iiif_error: undefined,
      patType: 9,
      patTypeName: "",
      multiName: "Local storage",
      allowMoveToLocal: true,
      // convertImage: false,
      mainMenu: false,
      origin,
    };
  },
  methods: {
    scrollTo(el) {
      const scroll = el.offsetTop - 110;
      window.scrollTo({
        top: scroll,
        behavior: "smooth",
      });
    },

    async downPNG() {
      let img = undefined;
      try {
        let logo = document.getElementById("gettylogo");
        if (document.getElementById("otherlogo")) {
          logo = document.getElementById("otherlogo");
        }
        img = await generateACNLQR(this.drawingTool, logo);
      } catch (e) {
        img = await generateACNLQR(this.drawingTool, undefined);
      }

      saveAs(img, this.drawingTool.title + ".png");
    },
    // patInfoSave(publish = false) {
    //   const patTitle = this.patTitle.trim();
    //   const patTown = this.patTown.trim();
    //   const patAuthor = this.patAuthor.trim();
    //   const titleCheck = patTitle && patTitle !== "Empty";
    //   const townCheck = patTown && patTown !== "Unknown";
    //   const nameCheck = patAuthor && patAuthor !== "Unknown";
    //   if (titleCheck && townCheck && nameCheck) {
    //     this.drawingTool.title = this.patTitle;
    //     if (this.drawingTool.creator[0] !== this.patAuthor)
    //       this.drawingTool.creator = this.patAuthor;
    //     if (this.drawingTool.town[0] !== this.patTown)
    //       this.drawingTool.town = this.patTown;
    //     if (this.drawingTool.patternType !== this.patType) {
    //       this.drawingTool.patternType = this.patType;
    //       this.patTypeName = this.drawingTool.typeInfo.name;
    //     }
    //     this.patInfoModal = false;
    //     if (publish) this.onPublish();
    //     return;
    //   }
    //   alert(
    //     "Please provide a valid pattern name, town name, and player name for this pattern."
    //   );
    // },
    // onLocalSave() {
    //   localStorage.setItem(
    //     "acnl_" + this.drawingTool.fullHash,
    //     lzString.compressToUTF16(this.drawingTool.toString())
    //   );
    // },
    onLoad: async function(t) {
      let patStr = this.drawingTool.toString();
      this.patType = this.drawingTool.patternType;
      this.patTypeName = this.drawingTool.typeInfo.name;

      // need to wait 2 ticks before access ref in portal
      // AFTER setting isOpenModal to true
      // https://portal-vue.linusb.org/guide/caveats.html#provide-inject
      await this.$nextTick();
      await this.$nextTick();

      /*
      const newHash = lzString.compressToEncodedURIComponent(patStr);
      const newPixHash = "#H:"+this.drawingTool.pixelHash;
      if (this.$router.currentRoute.hash !== "#" + newHash && this.$router.currentRoute.hash !== newPixHash) {
        this.$router.replace({ hash: newHash });
      }
      */
      return;
    },
    extLoad: function(data) {
      this.drawingTool.load(data);
    },
    onSearchSelect: function(data, scroll = true, preserveManifest = false) {
      if (!data) {
        console.log("handle this case");
        return null;
      }
      this.searchResult = data;

      this.clearOtherLogo();
      if (data.logo) {
        let logoImg = new Image();
        logoImg.src = data.logo;
        logoImg.id = "otherlogo";
        logoImg.setAttribute("crossorigin", "anonymous");
        logoImg.setAttribute("class", "hidden");
        document.body.appendChild(logoImg);
      }

      if (!preserveManifest && Object.keys(this.$route.query).length > 0) {
        this.$router.replace({ query: {} });
      }

      this.$set(this.iiif, "url", data.large_iiif_url);
      if (scroll) {
        this.scrollTo(this.$refs["step2"]);
      }
      // make sure gallery thumbs are visually unselected
      this.$refs["gallery"].selectedImageIndex = -1;
    },
    clearOtherLogo: function() {
      let logoImg = document.getElementById("otherlogo");
      if (logoImg) {
        logoImg.parentNode.removeChild(logoImg);
      }
    },
    onConvert: function(patterns) {
      // this.convertImage = false;
      let title = "untitled";
      if (patterns.length == 1) {
        title = this.iiif.short_name;
        this.extLoad(patterns[0]);
      } else {
        this.multiName = "Conversion result";
        this.allowMoveToLocal = true;
      }
      if (this.searchResult && this.searchResult.short_name) {
        title = this.searchResult.short_name;
      }

      this.drawingTool.title = title;
      const patStr = this.drawingTool.toString();
      this.qrCode = patStr;
    },
    onQROpen: function() {
      const patStr = this.drawingTool.toString();
      this.qrCode = patStr;
    },
    loadFromExample(exampleNumber) {
      let currentExample = examples[exampleNumber];
      this.searchResult = currentExample;
      if (Object.keys(this.$route.query).length > 0) {
        this.$router.replace({ query: {} });
      }
      this.$set(this.iiif, "url", currentExample.large_iiif_url);
      this.clearOtherLogo();
      this.$refs["imageloader"].setCropData(currentExample.crop);
      let self = this;
      // wait a tiny big before loading the cropper
      self.$refs["imageloader"].setCropData(currentExample.crop);
      // make sure search thumbs are visually unselected
      this.$refs["search"].selected = undefined;
    },
    updateIiifData(manifestUrl) {
      this.iiif_error = undefined;
      if (manifestUrl == undefined) {
        return;
      }

      if (!manifestUrl.startsWith("http")) {
        this.iiif_error = "Please enter a valid IIIF URL";
      }

      if (
        Object.keys(this.$route.query).length == 0 ||
        this.$route.query["iiif-content"] != manifestUrl
      ) {
        this.$router.replace({ query: { "iiif-content": manifestUrl } });
      }
      getIIIFData(manifestUrl)
        .then((data) => {
          if (data == undefined) {
            this.iiif_error =
              "There was an error processing your IIIF Manifest";
          } else {
            this.onSearchSelect(data, false, true);
          }
        })
        .catch((e) => {
          this.iiif_error = e.message;
        });
    },
  },
  mounted: function() {
    if (localStorage && localStorage.getItem("author_acnl")) {
      this.drawingTool.authorStrict = localStorage.getItem("author_acnl");
      this.storedAuthorHuman =
        this.drawingTool.creator[0] + " / " + this.drawingTool.town[0];
    }
    this.allTypes = this.drawingTool.allTypes;
    this.drawingTool.onLoad(this.onLoad);
    if (this.$router.currentRoute.hash.length > 1) {
      const hash = this.$router.currentRoute.hash.substring(1);
      if (hash.startsWith("H:")) {
        origin.view(hash.substring(2)).then((r) => {
          this.drawingTool.load(r);
        });
      } else {
        this.drawingTool.load(lzString.decompressFromEncodedURIComponent(hash));
      }
    } else {
      this.onLoad();
      this.drawingTool.render();
    }

    document.addEventListener("keydown", (e) => {
      if (e.ctrlKey && e.key === "Z") {
        this.drawingTool.redo();
        e.preventDefault();
        return;
      }
      if (e.ctrlKey && e.key === "z") {
        this.drawingTool.undo();
        e.preventDefault();
        return;
      }
    });
    if (this.$route.query["iiif-content"]) {
      let manifest = this.$route.query["iiif-content"];
      this.updateIiifData(manifest);
    } else {
      this.loadFromExample(0);
    }
  },
};
</script>

<style lang="scss">
.content a {
  color: #1a47b8;
  text-decoration: none;
}
@media screen and (min-width: 1024px) {
  .l-halves .leftborder {
    margin-left: 12px;
    padding-left: 12px;
    border-left: 1px solid #e6e6e6;
  }
}

@media screen and (max-width: 1023px) and (min-width: 768px) {
  .l-halves .leftborder {
    margin-left: 8px;
    padding-left: 8px;
    border-left: 1px solid #e6e6e6;
  }
}

.column-content {
  margin-top: 36px;
}
.generated-image {
  margin-bottom: 40px;
}
.hidden {
  display: none;
}
.o-hero__title {
  font-size: 28px !important;
}
.save-button {
  width: auto;
  display: flex;
  align-items: center;
}
.save-icon {
  margin-left: 12px;
  height: 18px;
  width: 18px;
  margin-top: -3px;
  cursor: pointer;
}
.section {
  margin-bottom: 80px;
}
.hero {
  width: 100%;
  max-height: 10em;
  overflow: hidden;
}

.top-padding {
  padding-top: 2em;
}
.view-in-collection {
  font-size: 15px;
  margin-top: 8px;
}
.top-margin4 {
  margin-top: 4em;
}
.m-record-metadata-field--horizontal {
  margin-bottom: 0;
}
</style>
