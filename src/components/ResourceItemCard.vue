<template>
  <div class="resource-item-card">
    <div class="card is-shady" :style="{ 'box-shadow': boxShadow }">
      <div class="card-image">
        <b-carousel
          v-if="resourceItem.covers && resourceItem.covers.length > 0"
          :indicator="resourceItem.covers.length > 1"
          :arrow="resourceItem.covers.length > 1"
          :autoplay="false"
          :pause-info="false"
        >
          <b-carousel-item
            class="carousel-image"
            v-for="cover in resourceItem.covers"
            :key="cover"
          >
            <img
              loading="lazy"
              :src="cover"
              :alt="resourceItem.name"
              class="cover-image"
            />
          </b-carousel-item>
        </b-carousel>
        <img
          v-else
          loading="lazy"
          style="background-color: black;width: 100%;height:160px;"
          class="cover-image"
        />
      </div>
      <div class="card-content">
        <div class="content">
          <h4
            class="resource-item-title truncated"
            @click="showResourceItemInfo"
          >
            <img
              v-if="icon.type === 'img'"
              style="border-radius: 4px; background: #ffffffd0;"
              class="item-icon"
              :src="icon.src"
            />
            <img
              v-else-if="icon.type === 'animal'"
              class="item-icon"
              style="border-radius: 50%;background: #167cf0b8;"
              :src="'/static/anonymousAnimals/' + icon.src + '.png'"
            />
            <b-icon v-else class="item-icon" :icon="icon.src" />
            {{ resourceItem.name }}
          </h4>
          <div class="buttons floating-buttons">
            <app-icons
              :apps="resourceItem.apps"
              :enableHover="!isTouchDevice"
            ></app-icons>
          </div>
          <span class="authors" :title="affil(resourceItem.authors)">
            {{
              resourceItem.authors && resourceItem.authors.length > 0
                ? "by " + etAl(resourceItem.authors)
                : ""
            }}
          </span>

          <p class="resource-item-description" v-if="resourceItem.description">
            {{
              resourceItem.description.slice(0, 60) +
                (resourceItem.description.length > 60 ? "..." : "")
            }}
          </p>
          <span style="margin-top:3px;display: block;">
            <span v-for="t in resourceItem.tags.slice(0, 4)" :key="t">
              <b-tag
                style="cursor: pointer;"
                rounded
                @click.native="selectTag(t)"
                >{{ t }}</b-tag
              >
            </span>
            <span v-if="resourceItem.tags.length > 4">
              <b-tag
                style="cursor: pointer;"
                rounded
                @click.native="showResourceItemInfo"
                >...</b-tag
              >
            </span>
          </span>
          <badges class="badges" :badges="resourceItem.badges"></badges>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Badges from "./Badges";
import AppIcons from "./AppIcons";
import siteConfig from "../../site.config.json";

const colorMap = {};
for (let it of siteConfig.resource_categories) {
  colorMap[it.type] = it.outline_color;
}

const isTouchDevice = (function() {
  try {
    document.createEvent("TouchEvent");
    return true;
  } catch (e) {
    return false;
  }
})();

export default {
  name: "ModelCard",
  props: {
    resourceItem: {
      type: Object,
      default: null
    }
  },
  data() {
    return {
      isTouchDevice: isTouchDevice
    };
  },
  components: {
    badges: Badges,
    "app-icons": AppIcons
  },
  computed: {
    boxShadow: function() {
      const color = colorMap[this.resourceItem.type] || "rgba(0,0,0,.2)";
      return `0 3px 1px -2px ${color}, 0 2px 2px 0 ${color}, 0 1px 5px 0 rgba(0,0,0,.12)`;
    },
    icon: function() {
      if (this.resourceItem.icon) {
        return { type: "material", src: this.resourceItem.icon };
      } else {
        return {};
      }
    }
  },
  methods: {
    etAl(authors) {
      authors = authors.map(author => {
        return author.name.split(";")[0];
      });
      if (authors.length < 3) {
        return authors.join(", ");
      } else {
        return authors.slice(0, 3).join(", ") + " et al.";
      }
    },
    affil(authors) {
      const affliations = authors.map(author => {
        return author.affiliation;
      });
      return Array.from(new Set(affliations)).join("; ");
    },
    showResourceItemInfo() {
      this.$emit("show-info", this.resourceItem);
    },
    selectTag(tag) {
      this.$emit("select-tag", tag);
    }
  }
};
</script>
<style scoped>
.resource-item-card {
  max-width: 500px;
}
.card {
  height: 360px;
}
.card-image {
  height: 160px;
}
.card-content {
  padding-left: 1rem;
  padding-right: 1rem;
  padding-top: 4px;
}
.resource-item-title {
  margin-bottom: 2px;
  margin-top: 6px;
  font-size: 1.2em;
  font-weight: 400;
  cursor: pointer;
  color: #002e52;
}
.authors {
  font-size: 0.9em;
  font-weight: 600;
}
.resource-item-description {
  font-size: 0.9em;
}

.floating-buttons {
  position: absolute;
  top: 0px;
  left: 5px;
}

.cover-image {
  height: 160px;
  max-height: 180px;
  object-fit: contain;
}
.carousel-image {
  max-height: 180px;
  background: black;
  text-align: center;
}
.item-icon {
  position: absolute;
  top: 125px;
  display: inline-block;
  margin-top: auto;
  margin-bottom: auto;
  border: 3px solid #00000000;
  margin-right: 4px;
  width: 32px;
  max-width: 100px;
}

.badges {
  position: absolute;
  left: 5px;
  bottom: 5px;
}

.truncated {
  width: 100%;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
</style>
