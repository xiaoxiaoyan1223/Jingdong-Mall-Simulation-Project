<template>
  <div class="wrapper">
    <ProductList />
  </div>
  <Docker :currentIndex="1"/>
  </template>

  <script>
  import Docker from '../../components/Docker'
  import ProductList from '../orderConfirmation/ProductList.vue'
  import { useRoute } from 'vue-router'
  import { useCommonCartEffect } from '../../effects/cartEffects'
  export default {
    name: 'CarList',
    components:{Docker,ProductList},
    setup() {
      const route = useRoute()
      const shopId = route.params.id
      const { shopName, productList } = useCommonCartEffect(shopId)
      return { shopName, productList }
    }
  }
  </script>

  <style lang="scss" scoped>
  @import '../../style/viriables.scss';
  @import '../../style/mixins.scss';
  .products {
    margin: .16rem .18rem .1rem .18rem;
    background: red($color: #000000);
    &__title {
      padding: .16rem;
      font-size: .16rem;
      color: $content-fontcolor;
    }
    &__wrapper {
      overflow-y: scroll;
      margin: 0 .18rem;
      position: absolute;
      left: 0;
      right: 0;
      bottom: .6rem;
      top: 2.6rem;
    }
    &__list {
      background: $bgColor;
    }
    &__item {
      position: relative;
      display: flex;
      padding: 0 .16rem 0.16rem .16rem;
      &__img {
        width: .46rem;
        height: .46rem;
        margin-right: .16rem;
      }
      &__detail {
        flex: 1;
      }
      &__title {
        margin: 0;
        line-height: .2rem;
        font-size: .14rem;
        color: $content-fontcolor;
        @include ellipsis;
      }
      &__price {
        display: flex;
        margin: .06rem 0 0 0;
        line-height: .2rem;
        font-size: .14rem;
        color: $hightlight-fontColor;
      }
      &__total {
        flex: 1;
        text-align: right;
        color: $dark-fontColor;
      }
      &__yen {
        font-size: .12rem;
      }
    }
  }
  </style>
