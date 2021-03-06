<template>
  <Layout>
    <nav class="breadcrumb" aria-label="breadcrumbs">
      <ul>
        <li><g-link to="/">Accueil</g-link></li>
        <li class="is-active"><a href="#">{{ exhibitor.name }}</a></li>
      </ul>
    </nav>

    <h2 class="title is-2">{{ exhibitor.name }}</h2>

    <div class="tile is-ancestor">
      <div class="tile is-parent is-8">
        <div class="tile is-child">
          <div class="presentation content" v-html="exhibitor.presentation"></div>

          <div class="infobox">
            <div v-if="exhibitor.payment && exhibitor.payment.length">
              <b-tooltip :label="$t('payment.means')" position="is-left"><b-icon icon="credit-card-outline" size="is-small" /></b-tooltip>&nbsp;
              <span v-for="(mean, i) in exhibitor.payment" :key="mean">{{ $t('payment.' + mean) }}<span v-if="i < exhibitor.payment.length - 1">, </span></span>
            </div>
            <div v-if="exhibitor.delivery && exhibitor.delivery.length">
              <b-tooltip :label="$t('delivery.means')" position="is-left"><b-icon icon="truck-delivery" size="is-small" /></b-tooltip>&nbsp;
              <span v-for="(mean, i) in exhibitor.delivery" :key="mean">
                <b-tooltip v-if="mean === 'pickup'" position="is-bottom" size="is-large" dashed multilined>
                  {{ $t('delivery.' + mean) }}
                  <template v-slot:content><div v-html="pickupLocations" class="content"></div></template>
                </b-tooltip>
                <b-tooltip v-else-if="mean === 'store'" position="is-bottom" size="is-large" dashed multilined>
                  {{ $t('delivery.' + mean) }}
                  <template v-slot:content><div v-html="storeLocations" class="content"></div></template>
                </b-tooltip>
                <span v-else>{{ $t('delivery.' + mean) }}</span>
              <span v-if="i < exhibitor.delivery.length - 1">, </span></span>
            </div>
            <div v-if="exhibitor.email">
              <b-tooltip :label="$t('exhibitor.email')" position="is-left"><b-icon icon="email" size="is-small" /></b-tooltip>&nbsp;
              <g-link :to="`mailto:${exhibitor.email}`">{{ exhibitor.email }}</g-link>
            </div>
            <div v-if="exhibitor.phone">
              <b-tooltip :label="$t('exhibitor.phone')" position="is-left"><b-icon icon="phone" size="is-small" /></b-tooltip>&nbsp;
              <span>{{ exhibitor.phone }}</span>
            </div>
            <div v-if="exhibitor.website">
              <b-tooltip :label="$t('exhibitor.website')" position="is-left"><b-icon icon="web" size="is-small" /></b-tooltip>&nbsp;
              <g-link :to="exhibitor.website">{{ exhibitor.website }}</g-link>
            </div>
            <div v-if="exhibitor.facebook">
              <b-tooltip :label="$t('exhibitor.facebook')" position="is-left"><b-icon icon="facebook" size="is-small" /></b-tooltip>&nbsp;
              <g-link :to="`https://www.facebook.com/${exhibitor.facebook}`">{{ exhibitor.facebook }}</g-link>
            </div>
            <div v-if="exhibitor.instagram">
              <b-tooltip :label="$t('exhibitor.instagram')" position="is-left"><b-icon icon="instagram" size="is-small" /></b-tooltip>&nbsp;
              <g-link :to="`https://www.instagram.com/${exhibitor.instagram}`">{{ exhibitor.instagram }}</g-link>
            </div>
          </div>
        </div>
      </div>
      <div class="tile is-parent is-vertical">
        <div class="tile is-child">
          <g-image v-if="exhibitor.image" :src="exhibitor.image.url" />
        </div>
        <div class="tile is-child videoBtn">
          <b-tooltip v-if="exhibitor.story" :label="$t('exhibitor.discover.story')" position="is-bottom">
            <b-button type="is-info" size="is-small" icon-left="book-open-variant" @click="isStoryModalActive = true">{{ $t('exhibitor.details') }}</b-button>
          </b-tooltip>
          &nbsp;
          <b-tooltip v-if="exhibitor.pictures && exhibitor.pictures.length" :label="$t('exhibitor.discover.pictures')" position="is-bottom">
            <b-button type="is-info" size="is-small" icon-left="image-multiple" @click="isPicturesModalActive = true">{{ $t('exhibitor.pictures') }}</b-button>
          </b-tooltip>
          &nbsp;
          <b-tooltip v-if="exhibitor.assetVideo" :label="$t('exhibitor.discover.video')" position="is-bottom">
            <b-button type="is-info" size="is-small" icon-left="movie" @click="isVideoModalActive = true">{{ $t('exhibitor.video') }}</b-button>
          </b-tooltip>
        </div>
      </div>
    </div>

    <div v-if="exhibitor.promotion" class="content promotion" v-html="exhibitor.promotion"></div>

    <b-modal v-if="exhibitor.story" v-model="isStoryModalActive">
      <b-message>
        <div class="content" v-html="exhibitor.story"></div>
      </b-message>
    </b-modal>

    <b-modal v-if="exhibitor.pictures && exhibitor.pictures.length" v-model="isPicturesModalActive">
      <b-carousel :indicator-inside="false" class="picturesCarousel">
        <b-carousel-item v-for="(item, i) in exhibitor.pictures.length" :key="i" class="bigPicture">
          <span class="image">
            <img :src="exhibitor.pictures[i].url">
          </span>
        </b-carousel-item>
        <template slot="indicators" slot-scope="props">
          <span class="al image">
            <img class="thumbPicture" :src="exhibitor.pictures[props.i].url" :title="props.i">
          </span>
        </template>
      </b-carousel>
    </b-modal>

    <b-modal v-if="exhibitor.assetVideo" v-model="isVideoModalActive">
      <div class="video">
        <div v-if="exhibitor.assetVideo.__typename === 'data_Asset'">
          <video controls autoplay>
            <source :src="exhibitor.assetVideo.url">
          </video>
        </div>
        <div v-else-if="exhibitor.assetVideo.__typename === 'data_Video'" v-html="exhibitor.assetVideo.html"></div>
      </div>
    </b-modal>

    <h3 class="title is-3">{{ $tc('product._', nbProducts) }}</h3>

    <card-list v-if="exhibitor.products.length">
      <product-card v-for="product in products" :key="product.id" :product="product" />
    </card-list>
    <div v-else>
      <p>{{ $t('product.no') }}</p>
    </div>
  </Layout>
</template>

<page-query>
query ($code: String!) {
  data {
    exhibitor(where: {code: $code}) {
      assetVideo {
        __typename
        ...on data_Asset {
          url
        }
        ...on data_Video {
          html
        }
      }
      code
      delivery
      email
      facebook
      image {
        url
      }
      instagram
      name
      payment
      phone
      pickUpRelay
      pictures {
        url
      }
      presentation
      products {
        exhibitor {
          code
          name
        }
        id
        image {
          url
        }
        name
        price
      }
      promotion
      store
      story
      website
    }
  }
}
</page-query>

<script>
import CardList from '~/components/CardList.vue'
import ProductCard from '~/components/ProductCard.vue'

export default {
  metaInfo(vm) {
    return {
      title: vm.$context.title
    }
  },
  components: {
    CardList,
    ProductCard
  },
  computed: {
    exhibitor() {
      return this.$page.data.exhibitor;
    },
    products() {
      return this.exhibitor.products.sort((a, b) => (a.name > b.name) ? 1 : -1);
    },
    nbProducts() {
      return this.exhibitor.products ? this.exhibitor.products.length : 0;
    },
    pickupLocations() {
      let pickupLocations = '';
      if (this.exhibitor.pickUpRelay && this.exhibitor.pickUpRelay.length) {
        pickupLocations += '<ul>';
        for (const location of this.exhibitor.pickUpRelay) {
          pickupLocations += '<li>' + location + '</li>';
        }
        pickupLocations += '</ul>';
      }
      return pickupLocations;
    },
    storeLocations() {
      let storeLocations = '';
      if (this.exhibitor.store && this.exhibitor.store.length) {
        storeLocations += '<ul>';
        for (const location of this.exhibitor.store) {
          storeLocations += '<li>' + location + '</li>';
        }
        storeLocations += '</ul>';
      }
      return storeLocations;
    }
  },
  data() {
    return {
      isStoryModalActive: false,
      isPicturesModalActive: false,
      isVideoModalActive: false
    }
  }
}
</script>

<style>
.presentation {
  margin-bottom: 20px;
  text-align: justify;
}

.infobox {
  border-left: 3px solid #999;
  padding-left: 10px;
}

.picturesCarousel {
  background-color: #111;
}
.bigPicture {
  background-color: black;
  padding: 15px;
}
.bigPicture img {
  max-height: 500px;
  object-fit: contain;
}
.thumbPicture {
  max-height: 50px;
  object-fit: contain;
}

.videoBtn {
  text-align: right;
}
.video {
  margin: auto;
  text-align: center;
}

.promotion {
  border: 1px dashed #c2a609;
  border-radius: 10px;
  background-color: #fefcf3;
  padding: 10px;
}
</style>
