<template>

  <div class="container-fluid">
    <!-- BEGIN SIDEBAR & CONTENT -->
    <div class="app-sidebar-container row margin-bottom-55">
      <!-- BEGIN SIDEBAR -->

      <div
        class="infinite-loader"
        v-cloak
        v-if="cartLoading"
      >
        <i class="tm tm-spinner anim-spin"></i>
      </div>
      <div
        class="sidebar-next col-md-12 col-sm-12 col-xs-12"
        :style="[ OrderPartpayRestricted || (shippingTotal + subTotal < 20) ? { 'min-height': '650px' } : { 'min-height': '450px' }]"
        v-else
      >

        <alert-box
          v-if="UnpaidOrders.length > 0 && IsUnpaidOrdersLoaded"
          :title="UnpaidAlertOptions.title"
          :body="UnpaidAlertOptions.body"
          v-bind:options="UnpaidAlertOptions.options"
          @click="navigateUnpaidOrder"
        />

        <div
          class="infinite-loader"
          v-cloak
          v-if="!$rootData.CartLoaded || (CalculatingPromos && CombinedCartPromoItemsList.length == 0)"
        >
          <i class="tm tm-spinner anim-spin"></i>
        </div>

        <div
          v-if="!(!$rootData.CartLoaded || !IsUnpaidOrdersLoaded || (CalculatingPromos && CombinedCartPromoItemsList.length == 0))"
          class="cart-banner-cont"
        >
          <div
            class="cart-banner-info"
            :class="{'margin-top-15' : $mq == 'xsmall'}"
          >
            <div
              class="d-flex-center row"
              :class="{'padding-y-15' : $mq == 'xsmall'}"
            >

              <router-link
                v-if="$mq == 'xsmall'"
                to="/c/christmas-and-gifting/6950"
                class="col-xs-4 col-sm-4 cart-banner-heading-link no-padding"
              >
                <h2 class="cart-banner-heading no-margin">Need more Christmas?</h2>
              </router-link>
              <h2
                v-else
                class="cart-banner-heading col-sm-4 no-margin"
                :class="{'padding-left-30' : !($mq == 'small') }"
              >Need more Christmas?</h2>

              <div
                class="col-xs-8 col-sm-8 d-flex-center"
                :class="{'no-padding justify-end margin-left-5' : $mq == 'xsmall'}"
              >
                <div class="cart-banner-img">
                  <router-link to="/c/christmas-and-gifting/stocking-fillers/6956">
                    <div
                      :style="{'background-image': 'url('+ $tmconfig.imageHost +'cart-square-stocking-stuffers.jpg)' }"
                      class="c-pointer cart-banner-img-bg background-image"
                    />
                  </router-link>
                </div>
                <div class="cart-banner-img">
                  <router-link to="/c/christmas-and-gifting/christmas/6959">
                    <div
                      :style="{'background-image': 'url('+ $tmconfig.imageHost +'cart-square-festive-decorations.jpg)' }"
                      class="c-pointer cart-banner-img-bg background-image"
                    />
                  </router-link>
                </div>
                <div
                  v-if="$mq !== 'xsmall'"
                  class="cart-banner-img"
                >
                  <router-link to="/list?cid=6054,6080,6135,6107,6288,6130">
                    <div
                      :style="{'background-image': 'url('+ $tmconfig.imageHost +'cart-square-beverages.jpg)' }"
                      class="c-pointer cart-banner-img-bg background-image"
                    />
                  </router-link>
                </div>
                <div
                  v-if="$mq !== 'xsmall'"
                  class="cart-banner-img"
                >
                  <router-link to="/list?cid=6907,6169,6244,6269,6262,6154">
                    <div
                      :style="{'background-image': 'url('+ $tmconfig.imageHost +'cart-square-delicious-treats.jpg)' }"
                      class="c-pointer cart-banner-img-bg background-image"
                    />
                  </router-link>
                </div>
              </div>
            </div>
          </div>
        </div>

        <alert-box
          v-if="CombinedCartPromoItemsList.length == 0 && !CalculatingPromos && $rootData.CartLoaded && !$rootData.CartError && UnpaidOrders.length == 0 && IsUnpaidOrdersLoaded"
          title="Your cart is empty"
          body="We've got lots of great stuff to find. Get inspired & explore our range."
        />

        <div class="cart-info-container">
          <content-image
            v-if="!subscriptionNotApplicable && eligibleForTrial && TrialBanner && $mq == 'xsmall' && CombinedCartPromoItemsList.length > 0"
            v-show="!applySubscriptionShipping"
            @onclick="onClickSubscriptionTrialBanner"
            :imgWidth="600"
            style="clear: both"
            class="c-pointer margin-bottom-10"
            :class="{ 'margin-top-10' : !(UnpaidOrders.length > 0 && IsUnpaidOrdersLoaded) }"
            className="img-responsive border-radius-5"
            imgBucket="bannerimages"
            :altValue="TrialBanner.BannerName"
            :imgKey="TrialBanner.BannerImage"
            useImage
          />

          <ui-feedback
            v-if="showSpendMore"
            type="info"
            :class="{ 'margin-top-10': $mq == 'xsmall' }"
          >
            <span>TheMarket Club members get free standard shipping on eligible orders over {{ $root.ReferenceData.SubscriptionConfig.freeShippingMinimumTotal | price }}.&nbsp;</span>
            <a
              class="hyperlink-color c-pointer"
              @click.prevent="onClickShowClubModal"
            >Learn More</a>
          </ui-feedback>
        </div>

        <div
          class="row"
          v-if="($mq == 'xsmall') && CombinedCartPromoItemsList.length > 0"
        >
          <div
            class="goods-page"
            style="min-height:calc(100vh - 160px);"
          >
            <div
              class="cart goods-data clearfix"
              style="padding-top: 3px;"
            >
              <div class="table-wrapper-responsive">
                <table summary="Shopping cart">
                  <tbody
                  >
                    <template v-if="item.DisplayMerchant">
                      <tr style="border: none;">
                        <td
                          colspan="6"
                          style="border:none; padding-bottom: 0"
                        >
                          <div class="row d-flex-center">
                            <template v-if="item.IsSubscription">
                              <div
                                class="col-xs-1"
                                style="width:65px;"
                              >
                                <div
                                  :style="{'background-image': 'url('+ $tmconfig.imageHost +'the-market-logo-arrow.png)' }"
                                  class="c-pointer background-image coupon-card-info__bg"
                                />
                              </div>
                              <div
                                class="col-xs-1 cart-merchant-name padding-top-0"
                                style="width:calc(100% - 65px)"
                              >
                                <span style="font-weight: bold;">{{ item.DisplayMerchant.MerchantName }}</span>
                              </div>
                            </template>
                            <template v-else>
                              <div
                                class="col-xs-1"
                                style="width:65px;"
                              >
                                <router-link :to="$links.merchant(item.DisplayMerchant)">
                                  <div
                                    :style="'background-image:url(' + item.DisplayMerchant.LogoImg + ');'"
                                    class="cart-merchant-logo"
                                  ></div>
                                </router-link>
                              </div>
                              <div
                                class="col-xs-1 cart-merchant-name padding-top-0"
                                style="width:calc(100% - 65px)"
                              >
                                <router-link :to="$links.merchant(item.DisplayMerchant)">
                                  <span style="font-weight: bold;">{{ item.DisplayMerchant.MerchantName }}</span>
                                  <span style="display: block;font-size: 12px;color: #8d8d8d;margin-top: 3px;">Store</span>
                                </router-link>
                              </div>
                            </template>
                          </div>
                        </td>
                      </tr>
                      <tr v-if="item.MerchantStatusId != 2 && !item.IsSubscription">
                        <td
                          style="border:none;padding-bottom:0px;padding-right:0px;"
                          colspan="5"
                        >
                          <ui-feedback
                            class="no-margin"
                            type="warning"
                          >
                            This Merchant is no longer available
                          </ui-feedback>
                        </td>
                      </tr>
                    </template>

                    <template v-if="item.MerchantStatusId == 2">
                      <tr v-if="item.IsOutOfStockOrNotAvailableMsg">
                        <td
                          style="border:none;padding-bottom:0px;padding-right:0px;"
                          colspan="5"
                        >
                          <div
                            style="margin:0px;"
                            class="alert alert-warning cart-validation-error"
                          >{{ item.IsOutOfStockOrNotAvailableMsg }}<span v-if="OrderError"> Please remove this item before continuing.</span></div>
                        </td>
                      </tr>

                      <tr v-if="item.Qty > item.QtyAts && !item.IsOutOfStockOrNotAvailableMsg">
                        <td
                          style="border:none;padding-bottom:0px;padding-right:0px;"
                          colspan="5"
                        >
                          <div
                            style="margin:0px;"
                            class="alert alert-warning cart-validation-error"
                          >This quantity not currently available for this product</div>
                        </td>
                      </tr>
                    </template>

                    <tr :class="{'blurred-item' : item.IsOutOfStockOrNotAvailableMsg, 'no-border' : (item.DisplaySubTotal && item.MerchantStatusId != 2) }">
                      <td class="goods-page-image blurred">
                        <a
                          v-if="!item.IsSubscription"
                          :href="$links.product(item, false, true)"
                          @click.prevent="handleViewProduct(item)"
                          rel="noopener"
                        >
                          <div
                            :style="'background-image:url(' + item.ProductImageUrl + ')'"
                            class="image-display-contain-size-square"
                          />
                        </a>
                        <div
                          v-else
                          :style="{'background-image': 'url('+ $tmconfig.imageHost +'TheMarketClub-Square.png)' }"
                          class="image-display-cover-size-square"
                        />
                      </td>

                      <td style="padding-right:0px;">
                        <a
                          v-if="!item.IsSubscription"
                          :href="$links.product(item, false, true)"
                          @click.prevent="handleViewProduct(item)"
                          rel="noopener"
                          class="cart-mobile-item-name blurred"
                        >
                          {{ item.SkuName }}
                        </a>
                        <div
                          class="font-medium font-size-14"
                          v-else-if="subscriptionInfo"
                        >
                          <strong class="margin-bottom-5 d-block">{{ subscriptionInfo.Title }}</strong>
                          <p class="color-dark-grey margin-bottom-5">{{ subscriptionInfo.Cost }}</p>
                        </div>

                        <template v-if="!item.IsSubscription">
                          <ul class="blurred product-item-meta-data margin-top-10 margin-bottom-5">
                            <template>
                              <li
                                class="size-color-data"
                                v-if="item.ColorId > 1"
                              >{{ IsSkuDeliveryItem(item) ? 'Delivery Region' : 'Colour' }}: {{ item.SkuColor }}</li>
                              <li
                                class="size-color-data"
                                v-if="showItemSizeLabel(item)"
                              >{{ IsSkuDeliveryItem(item) ? 'Delivery Date' : 'Size' }}: {{ item.SizeLabel }}</li>
                              <li
                                class="margin-top-10 font-size-13 d-flex-center"
                                v-if="item.Category && item.Category.IsPartpayRestricted"
                              >
                                <img
                                  :src="$tmconfig.imageHost+'zip-logo-small.svg'"
                                  class="img-responsive partpay-icon"
                                  alt="ZipLogo"
                                /> Not eligible for Zip
                              </li>
                              <li
                                class="deal-color"
                                v-if="item.PromotionId > 0"
                              >Promo: <span class="d-inline-block">{{ item.PromotionName }}</span></li>
                            </template>

                            <li class="d-flex">
                              <div
                                class="margin-right-5"
                                style="flex: 0 0 auto"
                              >
                                Unit price:
                              </div>
                              <div>
                                <span v-if="!item.IsDisplaySale && item.IsSale">{{ item.EffectivePrice | price }}</span>
                                <template v-else>
                                  <span :class="{ 'text-line-through' : (item.PromotionId && item.PromotionAmount > 0) || item.IsSale }">{{ item.PriceRetail | price }}</span>
                                  <span
                                    class="color-red"
                                    v-if="(item.PromotionId && item.PromotionAmount > 0) || item.IsSale"
                                  >{{ item.EffectivePrice | price }}</span>
                                </template>
                              </div>
                            </li>
                          </ul>

                          <div class="club-logo-container">
                            <club-logo v-if="!item.IsSubscriptionExclude && !item.IsMerchantSubscriptionExclude && subscriptionEnabled" />
                            <p
                              class="club-logo-container__bulky-text"
                              v-if="item.IsBulky"
                            >Bulky surcharge applies</p>
                          </div>
                        </template>

                        <div
                          v-if="!item.IsSubscription"
                          style="font-size: 14px;margin-top: 5px;float:left;"
                        >

                          <a
                            v-if="item.IsOutOfStockOrNotAvailableMsg"
                            href="javascript:;"
                            class="wishlist-buttons-left remove-item-unavailable"
                            v-on:click="removeToCart(item)"
                          >&#10060; Remove this item</a>
                          <a
                            v-else
                            href="javascript:;"
                            class="wishlist-buttons-left"
                            v-on:click="removeToCart(item)"
                          ><i
                              style="color:rgb(153, 153, 153);"
                              class="tm tm-trash"
                            ></i>&nbsp;Remove</a>

                        </div>

                        <template v-if="item.MerchantStatusId == 2">
                          <div
                            v-if="!item.IsSubscription"
                            class="blurred pull-right cart-wishlist-new-price-selection"
                          >

                            <div class="d-flex">
                              <div
                                class="infinite-loader margin-right-10 padding-0"
                                v-visible="CalculatingPromos && ChangedCartItemId == item.CartItemId"
                                style="min-height:0;"
                              >
                                <i class="tm tm-spinner anim-spin"></i>
                              </div>

                              <ui-select
                                small
                                v-model="item.QtyUpdated"
                                class="cart-quantity-list"
                                @change="itemQtyChange($event, item)"
                                :disabled="disableActions"
                                :options="getQtyOptions(item)"
                              />
                            </div>

                          </div>

                          <div
                            v-else
                            class="width-100 margin-top-10"
                          >

                            <div class="d-flex-center justify-space-between">
                              <a
                                v-if="!item.IsOutOfStockOrNotAvailableMsg"
                                href="javascript:;"
                                class="wishlist-buttons-left"
                                v-on:click="removeToCart(item)"
                              ><i
                                  style="color:rgb(153, 153, 153);"
                                  class="tm tm-trash"
                                ></i>&nbsp;Remove</a>

                              <ui-select
                                small
                                v-model="item.QtyUpdated"
                                class="cart-quantity-list"
                                @change="itemQtyChange($event, item)"
                                :disabled="disableActions"
                                :options="getQtyOptions(item)"
                              />
                            </div>

                          </div>
                        </template>

                      </td>
                    </tr>

                    <template v-if="item.DisplaySubTotal">
                      <template v-if="item.MerchantStatusId == 2">
                        <tr>
                          <td
                            style="padding-bottom:5px; padding-right:0; padding-top:10px;"
                            colspan="2"
                          >
                            <div
                              class="shopping-total"
                              :class="[(item.IsSubscription) ? 'padding-bottom-30' : 'padding-bottom-0']"
                            >
                              <div class="row">
                                <div class="col-xs-1 cart-price-mobile-left"><em class="strong font-size-14">Subtotal</em></div>
                                <div class="col-xs-1 cart-price-mobile-right"><strong class="price">{{ MerchantSubTotal(item.MerchantId) | price }}</strong></div>
                              </div>
                              <div
                                class="row"
                                v-if="GetOrderFieldValue(item.MerchantId, 'PromotionAmount') > 0"
                              >
                                <div class="col-xs-1 cart-price-mobile-left"><em class="strong font-size-14">Store Promo Savings</em></div>
                                <div class="col-xs-1 cart-price-mobile-right">
                                  <div
                                    class="price"
                                    v-show="GetOrderFieldValue(item.MerchantId, 'PromotionAmount') > 0"
                                  ><strong :class="{ 'price-deduct' : promotionTotal > 0 }">-{{ GetOrderFieldValue(item.MerchantId, 'PromotionAmount') | price }}</strong></div>
                                </div>
                              </div>
                            </div>
                          </td>
                        </tr>

                        <tr v-if="MerchantOrderInfo[item.MerchantId]">
                          <td
                            style="border: none; padding-right:0;"
                            colspan="2"
                            :class="[(index == CombinedCartPromoItemsList.length - 1) ? 'padding-bottom-30' : 'padding-bottom-20']"
                          >
                            <div class="shopping-total">
                              <div class="row">
                                <div class="cart-shipping-details col-xs-12">
                                  <h3 class="font-size-16 font-bold">Shipping / Pick up</h3>

                                  <div
                                    class="cart-shipping-details__sub"
                                    v-if="(applySubscriptionShipping || !subscriptionNotApplicable) && MerchantOrderInfo[item.MerchantId].SubscriptionApplicable"
                                  >
                                    <ui-radio
                                      id="SUBSCRIPTION"
                                      value="SUBSCRIPTION"
                                      :checked="MerchantOrderInfo[item.MerchantId].SubscriptionApplicable && applySubscriptionShipping"
                                      @input="onSubscriptionRadioOptIn"
                                    />
                                    <div class="cart-shipping-details__label">
                                      <div class="d-flex-center">
                                        <span>Shipping with&nbsp;</span>
                                        <club-logo
                                          :showlabel="false"
                                          :imgHeight="10"
                                        />
                                      </div>
                                      <span class="cart-shipping-details__label-line">
                                        <span v-if="!subscriptionNotApplicable && eligibleForTrial">Free {{ $root.ReferenceData.SubscriptionConfig.trialDays }} day trial -&nbsp;</span>
                                        <a
                                          class="cart-shipping-details__label-link"
                                          @click.prevent="onClickShowClubModal"
                                        >Learn More</a>
                                      </span>
                                    </div>
                                    <div
                                      class="cart-shipping-details__price"
                                      style="display:flex;flex-direction:column;justify-content: flex-end;"
                                    >
                                      <span
                                        class="text-line-through font-size-12"
                                        v-if="applySubscriptionShipping"
                                      >{{ MerchantOrderInfo[item.MerchantId].LowestService.OriginalShippingFee | price }}</span>
                                      <span class="strong price price-deduct">FREE</span>
                                    </div>
                                  </div>

                                  <div v-if="!(MerchantOrderInfo[item.MerchantId].SubscriptionApplicable && applySubscriptionShipping)">
                                    <ui-radio
                                      id="STANDARD"
                                      value="STANDARD"
                                      :checked="true"
                                    />
                                    <div class="font-normal">Standard</div>
                                    <div class="cart-shipping-details__price">
                                      <span
                                        :class="[(MerchantOrderInfo[item.MerchantId].IsFree) ? 'price price-deduct' : 'price-from']"
                                        class="strong asterisk"
                                      > {{ ((MerchantOrderInfo[item.MerchantId].IsFree) ? 'FREE' : MerchantOrderInfo[item.MerchantId].LowestService.OriginalShippingFee) | price }}</span>
                                    </div>
                                  </div>

                                  <div
                                    class="font-size-14"
                                    v-if="MerchantOrderInfo[item.MerchantId].IsBulky && MerchantOrderInfo[item.MerchantId].Merchant.BulkyFee > 0"
                                  >
                                    <div>Bulky surcharge applies</div>
                                    <div class="strong cart-shipping-details__price">
                                      {{ MerchantOrderInfo[item.MerchantId].Merchant.BulkyFee | price }}
                                    </div>
                                  </div>

                                  <template v-if="!CalculatingPromos">
                                    <template v-if="driveThroughEnabled">
                                      <div v-if="checkIfMerchantHasBulkyProduct(item.MerchantId)">Bulky items are not available to collect via MarketPoint or DriveThru locations</div>
                                      <div v-else-if="checkIfMerchantIsRefrigerated(item.MerchantId)">This package is not available to collect via MarketPoint locations</div>
                                      <div v-else-if="MerchantSubTotal(item.MerchantId) > 400">Packages over $400 are not available to collect via MarketPoint or DriveThru locations</div>
                                      <div v-else-if="!MerchantOrderInfo[item.MerchantId].IsPop">This package is not available to collect via MarketPoint or DriveThru locations</div>
                                    </template>
                                    <template v-else>
                                      <div v-if="checkIfMerchantHasBulkyProduct(item.MerchantId)">Bulky items are not available to collect at a MarketPoint location</div>
                                      <div v-else-if="MerchantSubTotal(item.MerchantId) > 400">Packages over $400 are not available to collect at a MarketPoint&nbsp;location</div>
                                      <div v-else-if="!MerchantOrderInfo[item.MerchantId].IsPop">This package is not available to collect at a MarketPoint location</div>
                                    </template>
                                  </template>

                                  <template v-if="item.IsMerchantPartpayRestricted">
                                    <div class="d-flex-center">
                                      <img
                                        :src="$tmconfig.imageHost+'zip-logo-small.svg'"
                                        class="img-responsive partpay-icon"
                                        alt="ZipLogo"
                                      /> Not eligible for Zip
                                    </div>
                                  </template>

                                </div>
                              </div>
                            </div>
                            <ui-feedback
                              class="font-size-13"
                              type="warning"
                              v-if="failedSubscriptionPayment"
                            >
                              Your TheMarket Club membership renewal could not be processed. Please go to <router-link to="/account">My Account</router-link> to update your details
                            </ui-feedback>
                            <ui-feedback
                              class="font-size-13"
                              type="info"
                              v-if="$root.User && $root.User.SubscriptionStatusId == 4 && !eligibleForTrial"
                            >
                              <span v-if="!hasSubscription">Your TheMarket Club membership has ended. You can re-join from&nbsp;</span>
                              <span v-else>Your TheMarket Club membership will expire. You can check the expiry date and re-join from&nbsp;</span>
                              <router-link to="/account">My Account</router-link>
                            </ui-feedback>
                          </td>
                        </tr>
                      </template>
                      <div
                        v-else
                        class="margin-bottom-20"
                      ></div>

                      <tr>
                        <td
                          class="cart-divider"
                          colspan="2"
                        >
                        </td>
                      </tr>
                    </template>

                  </tbody>
                </table>

              </div>
            </div>
            <div class="col-xs-12 no-padding margin-bottom-20">
              <div class="shopping-total cart-shopping-total-container">
                <h1 class="no-text-transform">Order Summary</h1>
                <ul>
                  <li class="shopping-total-price">
                    <em>Subtotal</em>
                    <strong class="price cart-grand-total"><span class="disp">{{ subTotal | price }}</span></strong>
                  </li>
                  <li class="shopping-total-price">
                    <em>Store Promo Savings</em>
                    <strong
                      class="price cart-grand-total"
                      :class="{ 'price-deduct' : promotionTotal > 0 }"
                    ><span v-if="promotionTotal > 0">-</span><span>{{ (promotionTotal > 0) ? promotionTotal : '-'  | price }}</span></strong>
                  </li>
                  <li
                    class="shopping-total-price no-border margin-bottom-5"
                    style="padding-bottom: 0 !important"
                  >

                    <div
                      class="trial-container font-size-16"
                      v-if="!subscriptionNotApplicable && eligibleForTrial && TrialBanner"
                      v-show="!applySubscriptionShipping"
                    >
                      <content-image
                        @onclick="onClickSubscriptionTrialBanner"
                        :imgWidth="600"
                        class="c-pointer"
                        className="img-responsive border-radius-5"
                        imgBucket="bannerimages"
                        :altValue="TrialBanner.BannerName"
                        :imgKey="TrialBanner.BannerImage"
                        useImage
                      />

                      <a
                        class="cart-shipping-details__label-link d-block margin-top-10"
                        @click.prevent="onClickShowClubModal"
                      >Learn More</a>
                    </div>

                    <em :class="{ 'd-block margin-bottom-10': isMarketpointAvailable }">{{ (isMarketpointAvailable) ? 'Ship to or pick up from:' : 'Ship to' }}</em>
                    <ui-select
                      small
                      v-model="SelectedShippingRegion"
                      class="cart-shipping-select"
                      :disabled="disableActions"
                      :options="RegionOptions"
                      :class="{'margin-left-5': !isMarketpointAvailable}"
                    />

                    <strong class="price cart-grand-total">
                      <span
                        v-if="!applySubscriptionShipping || shippingTotal > 0"
                        class="disp"
                      >
                        <span style="font-weight: 100; font-size: 11px;">from </span>
                        {{ shippingTotal | price }}*
                      </span>
                      <span
                        v-else
                        class="disp color-red"
                      >
                        FREE*
                      </span>
                    </strong>
                  </li>
                  <li
                    class="shopping-total-price"
                    style="padding-top: 0 !important"
                  >
                    <div
                      v-if="applySubscriptionShipping && shippingDiscountTotal > 0"
                      class="font-size-16 margin-bottom-10"
                    >
                      <div>Discounts</div>
                      <div class="font-size-14 font-normal d-flex justify-space-between font-italic color-red">
                        <span>TheMarket Club Shipping Discount</span>
                        <span>-{{ shippingDiscountTotal | price }}</span>
                      </div>
                    </div>

                    <div
                      v-if="bulkyFeeTotal"
                      class="font-size-16 margin-bottom-10"
                    >
                      <div>Additional Charges</div>
                      <div class="font-size-14 font-normal d-flex justify-space-between">
                        <span>Bulky Fee(s)</span>
                        <span>{{ bulkyFeeTotal | price }}</span>
                      </div>
                    </div>

                    <span style="font-weight: 100;display: block;margin-top: 5px;">
                      * Rural address & Express delivery may incur additional surcharge
                    </span>
                  </li>
                </ul>
                <div class="margin-top-20">
                  <span style="font-weight: 100;display: block;margin-top: 5px;">* You'll see all delivery options and coupon discounts on the Checkout page.</span>
                </div>
                <div
                  v-if="!partpayIsEligible"
                  class="margin-top-20 font-light"
                >
                  <div class="d-flex align-items-start">
                    <img
                      :src="$tmconfig.imageHost+'zip-logo-small.svg'"
                      style="width:20px !important; height: 20px !important"
                      class="img-responsive partpay-icon"
                      alt="ZipLogo"
                    />
                    <span>Some of the items in your cart are not eligible for payment by Zip.</span>
                  </div>
                  <ol class="padding-left-25">
                    <li class="no-border padding-bottom-0">
                      Proceed to check out & pay for items using credit card.
                      <div class="margin-top-10">OR</div>
                    </li>
                    <li class="no-border padding-top-0">Remove non-eligible items and then proceed to check out & pay using Zip.</li>
                  </ol>
                </div>
                <div
                  class="margin-top-10"
                  v-if="applySubscriptionShipping && eligibleForTrial"
                >
                  <span>By continuing to checkout you accept&nbsp;</span>
                  <a
                    class="hyperlink-color"
                    href="https://help.themarket.com/hc/en-us/articles/360037609593-TheMarket-Club-FAQs"
                    v-link
                    target="_blank"
                    rel="noopener"
                  >TheMarket Club Terms and Conditions</a>
                </div>
              </div>
              <div class="checkout-badge-row">
                <img
                  class="checkout-badge visa-badge"
                  :src="$tmconfig.imageHost + 'visa.svg?v=2'"
                >
                <img
                  class="checkout-badge"
                  :src="$tmconfig.imageHost + 'mastercard.svg'"
                >
                <img
                  class="checkout-badge amex-badge"
                  :src="$tmconfig.imageHost + 'amex.svg'"
                >
                <img
                  v-if="partpayIsEligible"
                  class="checkout-badge"
                  :src="$tmconfig.imageHost + 'zip-logo.svg'"
                >
                <img
                  class="checkout-badge ssl-badge"
                  :src="$tmconfig.imageHost + 'ssl-seal.png'"
                >
              </div>
            </div>
          </div>

          <!-- END HERE -->

          <div
            v-if="orderErrorMessage"
            class="col-xs-12 no-padding"
            style="margin-top: -20px;"
          >
            <div class="alert alert-warning">{{orderErrorMessage}}</div>
          </div>
        </div>

        <div
          class="row"
          v-if="!($mq == 'xsmall') && CombinedCartPromoItemsList.length > 0"
        >
          <div class="col-sm-12 col-md-8">

            <div class="goods-page">
              <div
                class="cart goods-data clearfix"
                style="padding-top:3px;padding-bottom:0px;"
              >
                <div class="table-wrapper-responsive">

                  <table summary="Shopping cart">
                    <tbody
                      v-for="item in CombinedCartPromoItemsList"
                      :key="item.CartItemId"
                    >
                      <template v-if="item.DisplayMerchant">
                        <tr style="border: none;">
                          <td
                            colspan="5"
                            style="border:none; padding-bottom: 0"
                          >
                            <div class="row pos-relative d-flex-center">
                              <template v-if="item.IsSubscription">
                                <div
                                  class="col-xs-1"
                                  style="width:65px;"
                                >
                                  <div
                                    :style="{'background-image': 'url('+ $tmconfig.imageHost +'the-market-logo-arrow.png)' }"
                                    class="c-pointer background-image coupon-card-info__bg"
                                  />
                                </div>
                                <div
                                  class="col-xs-1 cart-merchant-name padding-top-0"
                                  style="width:calc(100% - 65px)"
                                >
                                  <span style="font-weight: bold;">{{ item.DisplayMerchant.MerchantName }}</span>
                                </div>
                              </template>
                              <template v-else>
                                <div
                                  class="col-xs-1"
                                  style="width:65px;"
                                >
                                  <router-link :to="$links.merchant(item.DisplayMerchant)">
                                    <div
                                      :style="'background-image:url(' + item.DisplayMerchant.LogoImg + ');'"
                                      class="cart-merchant-logo"
                                    />
                                  </router-link>
                                </div>
                                <div
                                  class="col-xs-1 cart-merchant-name padding-top-0"
                                  style="width:calc(100% - 65px)"
                                >
                                  <router-link :to="$links.merchant(item.DisplayMerchant)">
                                    <span style="font-weight: bold;">{{ item.DisplayMerchant.MerchantName }}</span>
                                    <span style="display: block;font-size: 12px;color: #8d8d8d;margin-top: 3px;">Store</span>
                                  </router-link>
                                </div>
                              </template>
                            </div>
                          </td>
                        </tr>

                        <tr v-if="item.MerchantStatusId != 2 && !item.IsSubscription">
                          <td
                            style="border:none;padding-bottom:0px;padding-right:0px;"
                            colspan="5"
                          >
                            <ui-feedback
                              class="no-margin"
                              type="warning"
                            >
                              This Merchant is no longer available
                            </ui-feedback>
                          </td>
                        </tr>

                        <tr>
                          <th class="add-p-t goods-page-image">Items</th>
                          <th class="add-p-t goods-page-description"></th>
                          <th class="add-p-t goods-page-quantity">{{ (item.IsSubscription) ? 'Plan' : 'Quantity' }}</th>
                          <th
                            nowrap
                            class="add-p-t goods-page-price"
                          >{{ (item.IsSubscription) ? '' : 'Unit Price' }}</th>
                          <th class="add-p-t goods-page-price padding-left-20">Total</th>
                        </tr>
                      </template>

                      <template v-if="item.MerchantStatusId == 2">
                        <tr v-if="item.IsOutOfStockOrNotAvailableMsg">
                          <td
                            style="border:none;padding-bottom:0px;padding-right:0px;"
                            colspan="5"
                          >
                            <div
                              style="margin:0px;"
                              class="alert alert-warning cart-validation-error"
                            >{{ item.IsOutOfStockOrNotAvailableMsg }}<span v-if="OrderError"> Please remove this item before continuing.</span></div>
                          </td>
                        </tr>

                        <tr v-if="item.Qty > item.QtyAts && !item.IsOutOfStockOrNotAvailableMsg">
                          <td
                            style="border:none;padding-bottom:0px;padding-right:0px;"
                            colspan="5"
                          >
                            <div
                              style="margin:0px;"
                              class="alert alert-warning cart-validation-error"
                            >This quantity not currently available for this product</div>
                          </td>
                        </tr>
                      </template>

                      <tr :class="{'blurred-item' : item.IsOutOfStockOrNotAvailableMsg, 'no-border' : (item.DisplaySubTotal && item.MerchantStatusId != 2) }">
                        <td class="goods-page-image blurred">
                          <router-link
                            v-if="!item.IsSubscription"
                            :to="$links.product(item)"
                          >
                            <div
                              :style="'background-image:url(' + item.ProductImageUrl + ')'"
                              class="image-display-contain-size-square"
                            />
                          </router-link>
                          <div
                            v-else
                            :style="{'background-image': 'url('+ $tmconfig.imageHost +'TheMarketClub-Square.png)' }"
                            class="image-display-cover-size-square"
                          />
                        </td>
                        <td class="goods-page-description">
                          <template v-if="!item.IsSubscription">
                            <h3 class="blurred">
                              <router-link :to="$links.product(item)">{{ item.SkuName }}</router-link>
                            </h3>
                            <ul class="product-item-meta-data blurred">
                              <li>Brand: {{ $root.getBrand(item.BrandId, 'BrandName') }}</li>
                              <li
                                class="size-color-data"
                                v-if="showItemColorLabel(item)"
                              >{{ IsSkuDeliveryItem(item) ? 'Delivery Region' : 'Colour' }}: {{ item.SkuColor }}</li>
                              <li
                                class="size-color-data"
                                v-if="showItemSizeLabel(item)"
                              >{{ IsSkuDeliveryItem(item) ? 'Delivery Date' : 'Size' }}: {{ item.SizeLabel }}</li>
                              <li
                                class="deal-color"
                                v-if="item.PromotionId > 0"
                              >Promo: {{ item.PromotionName }}</li>
                            </ul>

                            <div class="club-logo-container">
                              <club-logo v-if="!item.IsSubscriptionExclude && !item.IsMerchantSubscriptionExclude && subscriptionEnabled" />
                              <p
                                class="club-logo-container__bulky-text"
                                v-if="item.IsBulky"
                              >Bulky surcharge applies</p>
                            </div>
                          </template>

                          <div
                            class="font-medium font-size-14"
                            v-else-if="subscriptionInfo"
                          >
                            <strong class="d-block margin-bottom-5">{{ subscriptionInfo.Title }}</strong>
                            <p class="color-dark-grey margin-bottom-5">{{ subscriptionInfo.Cost }}</p>
                          </div>

                          <div class="d-flex font-size-14 margin-top-10">

                            <a
                              v-if="item.IsOutOfStockOrNotAvailableMsg"
                              href="javascript:;"
                              class="wishlist-buttons-left remove-item-unavailable"
                              v-on:click="removeToCart(item)"
                            >&#10060; Remove this item</a>
                            <a
                              v-else
                              href="javascript:;"
                              class="wishlist-buttons-left"
                              v-on:click="removeToCart(item)"
                            ><i
                                style="color:rgb(153, 153, 153);"
                                class="tm tm-trash"
                              ></i>&nbsp;Remove</a>
                            <span
                              class="font-size-13 d-flex-center"
                              v-if="item.Category && item.Category.IsPartpayRestricted"
                            >
                              <img
                                :src="$tmconfig.imageHost+'zip-logo-small.svg'"
                                class="img-responsive partpay-icon"
                                alt="ZipLogo"
                              /> Not eligible for Zip
                            </span>

                          </div>

                        </td>
                        <td
                          :colspan="(item.IsSubscription) ? 2 : 1"
                          class="goods-page-quantity cart-wishlist-new-price-selection padding-left-0"
                        >
                          <div class="d-flex">
                            <ui-select
                              class="cart-quantity-list"
                              v-model="item.QtyUpdated"
                              @change="itemQtyChange($event, item)"
                              :disabled="disableActions || item.MerchantStatusId != 2"
                              :options="getQtyOptions(item)"
                            />

                            <div
                              v-visible="CalculatingPromos && ChangedCartItemId == item.CartItemId"
                              style="min-height:0;"
                              class="infinite-loader margin-left-10 padding-top-0"
                            >
                              <i class="tm tm-spinner anim-spin"></i>
                            </div>
                          </div>
                        </td>

                        <template v-if="!item.IsSubscription">
                          <td class="goods-page-price blurred">
                            <strong v-if="!item.IsDisplaySale && item.IsSale">{{ item.EffectivePrice | price }}</strong>
                            <template v-else>
                              <strong :class="{ 'text-line-through' : (item.PromotionId && item.PromotionAmount > 0) || item.IsSale }">{{ item.PriceRetail | price }}</strong>
                              <strong
                                class="color-red"
                                v-if="(item.PromotionId && item.PromotionAmount > 0) || item.IsSale"
                              >{{ item.EffectivePrice | price }}</strong>
                            </template>
                          </td>
                        </template>
                        <td class="goods-page-price blurred text-right">
                          <strong v-if="!item.IsDisplaySale && item.IsSale">{{ (item.ItemTotal - item.PromotionAmount) | price }}</strong>
                          <template v-else>
                            <strong :class="{ 'text-line-through' : (item.PromotionId && item.PromotionAmount > 0) || item.IsSale }">{{ (item.PriceRetail * item.QtyUpdated) | price }}</strong>
                            <strong
                              class="color-red"
                              v-if="(item.PromotionId && item.PromotionAmount > 0) || item.IsSale"
                            >{{ (item.ItemTotal - item.PromotionAmount) | price }}</strong>
                          </template>
                        </td>
                      </tr>

                      <!-- Sub total section -->
                      <template v-if="item.DisplaySubTotal">
                        <template v-if="item.MerchantStatusId == 2">
                          <tr
                            style="border-bottom: 0"
                            class="font-size-16"
                          >
                            <td colspan="2">
                              <div class="font-size-14">
                                <template v-if="!CalculatingPromos">
                                  <template v-if="driveThroughEnabled">
                                    <p v-if="checkIfMerchantHasBulkyProduct(item.MerchantId)">Bulky items are not available to collect via MarketPoint or DriveThru locations</p>
                                    <p v-else-if="checkIfMerchantIsRefrigerated(item.MerchantId)">This package is not available to collect via MarketPoint locations</p>
                                    <p v-else-if="MerchantSubTotal(item.MerchantId) > 400">Packages over $400 are not available to collect via MarketPoint or DriveThru locations</p>
                                    <p v-else-if="MerchantOrderInfo[item.MerchantId] && !MerchantOrderInfo[item.MerchantId].IsPop">This package is not available to collect via MarketPoint or DriveThru locations</p>
                                  </template>
                                  <template v-else>
                                    <p v-if="checkIfMerchantHasBulkyProduct(item.MerchantId)">Bulky items are not available to collect at a MarketPoint location</p>
                                    <p v-else-if="MerchantSubTotal(item.MerchantId) > 400">Packages over $400 are not available to collect at a MarketPoint&nbsp;location</p>
                                    <p v-else-if="MerchantOrderInfo[item.MerchantId] && !MerchantOrderInfo[item.MerchantId].IsPop">This package is not available to collect at a MarketPoint location</p>
                                  </template>
                                </template>

                                <p
                                  v-if="item.IsMerchantPartpayRestricted"
                                  class="margin-top-10 d-flex-center"
                                >
                                  <img
                                    :src="$tmconfig.imageHost+'zip-logo-small.svg'"
                                    class="img-responsive partpay-icon"
                                    alt="ZipLogo"
                                  />
                                  This store is not eligible for Zip
                                </p>
                              </div>
                            </td>

                            <td colspan="3">
                              <div
                                class="d-flex-center"
                                style="justify-content:flex-end"
                              >
                                <div class="no-padding text-right margin-right-30">
                                  <div
                                    class="strong"
                                    :class="{ 'margin-bottom-20': GetOrderFieldValue(item.MerchantId, 'PromotionAmount') > 0 || item.IsSubscription }"
                                  >Subtotal</div>
                                  <div
                                    v-if="GetOrderFieldValue(item.MerchantId, 'PromotionAmount') > 0"
                                    class="strong"
                                  >Store Promo Savings</div>
                                </div>
                                <div class="no-padding text-right">
                                  <div
                                    class="price"
                                    :class="{ 'margin-bottom-20': GetOrderFieldValue(item.MerchantId, 'PromotionAmount') > 0 || item.IsSubscription }"
                                  ><strong>{{ MerchantSubTotal(item.MerchantId) | price }}</strong></div>
                                  <div
                                    class="price price-deduct"
                                    v-show="GetOrderFieldValue(item.MerchantId, 'PromotionAmount') > 0"
                                  ><strong>-{{ GetOrderFieldValue(item.MerchantId, 'PromotionAmount') | price }}</strong></div>
                                </div>
                              </div>
                            </td>
                          </tr>

                          <tr
                            v-if="MerchantOrderInfo[item.MerchantId]"
                            class="font-size-16"
                          >
                            <td colspan="5">
                              <div class="d-flex justify-space-between margin-bottom-20">
                                <h3 class="font-size-16 font-bold">Shipping / Pick up</h3>

                                <div class="cart-shipping-details">

                                  <div v-if="((applySubscriptionShipping || (!subscriptionNotApplicable && !applySubscriptionShipping && !MerchantOrderInfo[item.MerchantId].IsFree)) && MerchantOrderInfo[item.MerchantId].SubscriptionApplicable)">
                                    <div class="cart-shipping-details__label">
                                      <div class="d-flex-center">
                                        <span>Shipping with&nbsp;</span>
                                        <club-logo
                                          :showlabel="false"
                                          :imgHeight="12"
                                        />
                                      </div>
                                      <span class="cart-shipping-details__label-line">
                                        <span v-if="!subscriptionNotApplicable && eligibleForTrial">Free {{ $root.ReferenceData.SubscriptionConfig.trialDays }} day trial -&nbsp;</span>
                                        <a
                                          class="cart-shipping-details__label-link"
                                          @click.prevent="onClickShowClubModal"
                                        >Learn More</a>
                                      </span>
                                    </div>
                                    <ui-radio
                                      id="SUBSCRIPTION"
                                      value="SUBSCRIPTION"
                                      :checked="MerchantOrderInfo[item.MerchantId].SubscriptionApplicable && applySubscriptionShipping"
                                      @input="onSubscriptionRadioOptIn"
                                    />
                                    <div
                                      class="cart-shipping-details__price"
                                      style="display:flex;flex-direction:column;justify-content: flex-end;"
                                    >
                                      <span
                                        class="price text-line-through font-size-12"
                                        v-if="applySubscriptionShipping && !MerchantOrderInfo[item.MerchantId].IsFree"
                                      >{{ MerchantOrderInfo[item.MerchantId].LowestService.OriginalShippingFee | price }}</span>
                                      <strong class="price price-deduct">FREE</strong>
                                    </div>
                                  </div>

                                  <div v-if="!(MerchantOrderInfo[item.MerchantId].SubscriptionApplicable && applySubscriptionShipping)">
                                    <div>Standard</div>
                                    <ui-radio
                                      id="STANDARD"
                                      value="STANDARD"
                                      :checked="true"
                                    />
                                    <div class="cart-shipping-details__price">
                                      <span
                                        :class="[(MerchantOrderInfo[item.MerchantId].IsFree) ? 'price price-deduct' : 'price-from']"
                                        class="strong asterisk"
                                      >
                                        {{ ((MerchantOrderInfo[item.MerchantId].IsFree) ? 'FREE' : MerchantOrderInfo[item.MerchantId].LowestService.OriginalShippingFee) | price }}
                                      </span>
                                    </div>
                                  </div>

                                  <div
                                    class="font-size-14"
                                    v-if="MerchantOrderInfo[item.MerchantId].IsBulky && MerchantOrderInfo[item.MerchantId].Merchant.BulkyFee > 0"
                                  >
                                    <div>Bulky surcharge applies</div>
                                    <div class="cart-shipping-details__price">
                                      <strong> {{ MerchantOrderInfo[item.MerchantId].Merchant.BulkyFee | price }}</strong>
                                    </div>
                                  </div>

                                </div>
                              </div>
                              <ui-feedback
                                class="font-size-13"
                                type="warning"
                                v-if="failedSubscriptionPayment"
                              >
                                Your TheMarket Club renewal could not be processed. Please go to <router-link to="/account">My Account</router-link> to update your details
                              </ui-feedback>
                              <ui-feedback
                                class="font-size-13"
                                type="info"
                                v-if="$root.User && $root.User.SubscriptionStatusId == 4 && !eligibleForTrial"
                              >
                                <span v-if="!hasSubscription">Your TheMarket Club membership has ended. You can re-join from&nbsp;</span>
                                <span v-else>Your TheMarket Club membership will expire. You can check the expiry date and re-join from&nbsp;</span>
                                <router-link to="/account">My Account</router-link>
                              </ui-feedback>
                            </td>
                          </tr>
                        </template>
                        <div
                          v-else
                          class="margin-bottom-20"
                        ></div>

                        <tr>
                          <td
                            class="cart-divider"
                            colspan="5"
                          ></td>
                        </tr>
                      </template>

                    </tbody>
                  </table>

                </div>
              </div>
            </div>

            <!-- Charity Appeal -->
            <template v-if="CharityAppealSkuId">
              <div class="cart-charity-appeal cart-charity-appeal-desktop">
                <img
                  class="cart-charity-appeal-image"
                  :src="$links.cdn('st-johns-donation.svg')"
                />

                <div class="cart-charity-appeal-text">
                  <h3 class="cart-charity-appeal-heading margin-bottom-10">
                    Gold Coin Donation to St John's Charity
                  </h3>
                  <h4 class="margin-bottom-0">Cookie Time</h4>
                </div>

                <div class="cart-charity-appeal-controls">
                  <ui-select
                    v-model="CharityAppealQuantity"
                    :options="charityQuantityOptions"
                  ></ui-select>
                  <div
                    large
                    type="primary-red"
                    @click="addCharitySkuToCart"
                    :disabled="charityCanAddDonation"
                  >Add Donation</div>
                </div>
              </div>
            </template>
            <!-- END HERE -->
          </div>

          <div class="col-sm-6 col-md-4 sticky-top">

            <div class="shopping-total cart-shopping-total-container">
              <h1 class="no-text-transform">Order Summary</h1>
              <ul class="margin-bottom-20">
                <li class="shopping-total-price">
                  <em>Subtotal</em>
                  <strong class="price cart-grand-total"><span class="disp">{{ subTotal | price }}</span></strong>
                </li>
                <li class="shopping-total-price">
                  <em>Store Promo Savings</em>
                  <strong
                    class="price cart-grand-total"
                    :class="{ 'price-deduct' : promotionTotal > 0 }"
                  ><span v-if="promotionTotal > 0">-</span><span>{{ (promotionTotal > 0) ? promotionTotal : '-'  | price }}</span></strong>
                </li>

                <slot
                  name="order-summary-shipping-section"
                  :shipping-total="shippingTotal"
                >
                  <li class="shopping-total-price no-border padding-bottom-0 margin-bottom-10">

                    <div
                      class="trial-container font-size-16"
                      v-if="!subscriptionNotApplicable && eligibleForTrial && TrialBanner"
                      v-show="!applySubscriptionShipping"
                    >
                      <content-image
                        @onclick="onClickSubscriptionTrialBanner"
                        :imgWidth="600"
                        class="c-pointer"
                        className="img-responsive border-radius-5"
                        imgBucket="bannerimages"
                        :altValue="TrialBanner.BannerName"
                        :imgKey="TrialBanner.BannerImage"
                        useImage
                      />

                      <a
                        class="cart-shipping-details__label-link margin-bottom-10"
                        @click.prevent="onClickShowClubModal"
                      >Learn More</a>
                    </div>

                    <em :class="{ 'd-block margin-bottom-10': isMarketpointAvailable }">{{ (isMarketpointAvailable) ? 'Ship to or pick up from:' : 'Ship to' }}</em>
                    <ui-select
                      small
                      v-model="SelectedShippingRegion"
                      class="cart-shipping-select"
                      :disabled="disableActions"
                      :options="RegionOptions"
                      :class="{'margin-left-5': !isMarketpointAvailable}"
                    />

                    <strong class="price cart-grand-total">
                      <span
                        v-if="!applySubscriptionShipping || shippingTotal > 0"
                        class="disp"
                      >
                        <span style="font-weight: 100; font-size: 11px;">from </span>
                        {{ shippingTotal | price }}*
                      </span>
                      <span
                        v-else
                        class="disp color-red"
                      >
                        FREE*
                      </span>
                    </strong>
                  </li>
                </slot>
                <li class="shopping-total-price padding-top-0">
                  <div
                    v-if="applySubscriptionShipping && shippingDiscountTotal > 0"
                    class="font-size-16 margin-bottom-10"
                  >
                    <div>Discounts</div>
                    <div class="font-size-14 font-normal d-flex justify-space-between font-italic color-red">
                      <span>TheMarket Club Shipping Discount</span>
                      <span>-{{ shippingDiscountTotal | price }}</span>
                    </div>
                  </div>

                  <div
                    v-if="bulkyFeeTotal"
                    class="font-size-16"
                  >
                    <div>Additional Charges</div>
                    <div class="font-size-14 font-normal d-flex justify-space-between">
                      <span>Bulky Fee(s)</span>
                      <span>{{ bulkyFeeTotal | price }}</span>
                    </div>
                  </div>

                  <span style="font-weight: 100;display: block;margin-top: 5px;">
                    * Rural address & Express delivery may incur additional surcharge
                  </span>
                </li>
                <li class="shopping-total-price">
                  <em>Total</em>
                  <strong class="price cart-grand-total"><span class="disp">{{ orderTotal | price }}</span></strong>

                </li>
              </ul>

              <div
                class="alert alert-warning margin-bottom-15"
                v-if="orderErrorMessage"
              >{{orderErrorMessage}}</div>

              <div
                type="primary-blue"
                block
                large
                :disabled="disableActions || !!orderErrorMessage || $tmconfig.checkoutDisabled"
                @click="checkout()"
                :loading="isAttemptingCheckout"
                icon="tm-lock-locked"
              >Checkout</div>
              <p
                v-if="$tmconfig.checkoutDisabled"
                class="hyperlink-color text-center font-size-16 margin-top-10"
              >Checkout temporarily disabled</p>
              <div class="margin-top-20 font-light">
                <div v-if="applySubscriptionShipping && eligibleForTrial">
                  <span>By continuing to checkout you accept&nbsp;</span>
                  <a
                    class="hyperlink-color"
                    href="https://help.themarket.com/hc/en-us/articles/360037609593-TheMarket-Club-FAQs"
                    v-link
                    target="_blank"
                    rel="noopener"
                  >TheMarket Club Terms and Conditions</a>
                </div>
                <span style="font-weight: 100;display: block;margin-top: 5px;">* You'll see all delivery options and coupon discounts on the Checkout page.</span>
                <div
                  v-if="!partpayIsEligible"
                  class="margin-top-20"
                >
                  <div
                    v-if="(shippingTotal + subTotal < 20)"
                    class="d-flex-center margin-bottom-15"
                  >
                    <img
                      :src="$tmconfig.imageHost+'zip-logo-small.svg'"
                      style="width:20px !important; height:20px !important"
                      class="img-responsive partpay-icon"
                      alt="ZipLogo"
                    />
                    <span>Zip is only available on orders above $20.</span>
                  </div>
                  <template v-else-if="OrderPartpayRestricted">
                    <div class="d-flex align-items-start">
                      <img
                        :src="$tmconfig.imageHost+'zip-logo-small.svg'"
                        style="width:20px !important; height:20px !important"
                        class="img-responsive partpay-icon"
                        alt="ZipLogo"
                      />
                      <span>Some of the items in your cart are not eligible for payment by Zip.</span>
                    </div>
                    <ol>
                      <li class="no-border padding-bottom-0">
                        Proceed to check out & pay for items using credit card.
                        <div class="margin-top-10 margin-bottom-10">OR</div>
                      </li>
                      <li class="no-border padding-top-0">Remove non-eligible items and then proceed to check out & pay using Zip.</li>
                    </ol>
                  </template>
                </div>
              </div>
            </div>
            <div class="checkout-badge-row">
              <img
                class="checkout-badge visa-badge"
                :src="$tmconfig.imageHost + 'visa.svg?v=2'"
              >
              <img
                class="checkout-badge"
                :src="$tmconfig.imageHost + 'mastercard.svg'"
              >
              <img
                class="checkout-badge amex-badge"
                :src="$tmconfig.imageHost + 'amex.svg'"
              >
              <img
                v-if="partpayIsEligible"
                class="checkout-badge"
                :src="$tmconfig.imageHost + 'zip-logo.svg'"
              >
              <img
                class="checkout-badge ssl-badge"
                :src="$tmconfig.imageHost + 'ssl-seal.png'"
              >
            </div>
          </div>
        </div>

        <!-- Charity Appeal => Mobile -->
        <template v-if="CharityAppealSkuId && $mq == 'xsmall'">
          <div class="cart-charity-appeal cart-charity-appeal-mobile">
            <div class="cart-charity-appeal-text">
              <h3 class="cart-charity-appeal-heading margin-bottom-10">
                Gold Coin Donation to St&nbsp;John's&nbsp;Charity
              </h3>
              <h4 class="margin-bottom-0">Cookie Time</h4>
            </div>

            <div class="d-flex">
              <img
                class="cart-charity-appeal-image"
                :src="$links.cdn('st-johns-donation.svg')"
              />
              <div>&nbsp;</div>
              <div class="cart-charity-appeal-controls">
                <ui-select
                  class="margin-bottom-10"
                  v-model="CharityAppealQuantity"
                  :options="charityQuantityOptions"
                ></ui-select>
                <div
                  type="primary-red"
                  @click="addCharitySkuToCart"
                  :disabled="charityCanAddDonation"
                >Add Donation</div>
              </div>
            </div>
          </div>
        </template>
      </div>
    </div>

    <club-learnmore-modal
      v-if="subscriptionEnabled"
      :show="showClubModal"
      :onClose="closeClubModal"
    />

    <template v-if="$mq == 'xsmall'">
      <transition
        :name="($tmconfig.appTransition && $mq == 'xsmall' && isCordovaApp) ? 'slide-up' : null"
        appear
      >
        <div
          v-if="CombinedCartPromoItemsList.length > 0"
          class="position-fixed-bottom bg-white"
        >
          <div
            class="box-row no-padding-top-and-bottom no-box-shadow no-margin"
            style="margin-bottom:0;"
          >
            <div class="row d-flex-center">
              <div class="col-xs-1 inquired-add-to-wishlist-container">
                <div style="color:rgb(141, 141, 141);">Total</div>
                <strong
                  style="font-size:17px;"
                  class="price cart-grand-total"
                ><span class="disp">{{ orderTotal | price }}</span></strong>
              </div>
              <div class="col-xs-1 add-to-cart-button-cont hyperlink-color font-size-16">
                <div
                  v-if="!$tmconfig.checkoutDisabled"
                  type="primary-blue"
                  large
                  block
                  :disabled="disableActions || !!orderErrorMessage"
                  @click="checkout()"
                  :loading="isAttemptingCheckout"
                  icon="tm-lock-locked"
                >Checkout</div>
                <div v-else>Checkout temporarily disabled</div>
              </div>
            </div>
          </div>
        </div>
      </transition>
    </template>

  </div>
</template>      
