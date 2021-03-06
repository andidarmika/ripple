<template>
  <transition name="rpl-header-fade">
    <div v-show="headerVisible"
      class="rpl-site-header"
      :class="{
        'rpl-site-header--open': menuContentOpen,
        'rpl-site-header--sticky': stickyActive,
      }"
    >
      <div class="rpl-site-header__inner">
        <!-- Top Bar -->
        <div class="rpl-site-header__top">
          <div class="rpl-site-header__logo-container">
            <!-- Menu Button -->
            <button
              v-if="searchState !== 'opened'"
              class="rpl-site-header__btn rpl-site-header__btn--menu"
              :class="{'rpl-site-header__btn--menu-open' : (menuState === 'opened')}"
              :aria-expanded="(menuState === 'opened').toString()"
              @click="menuToggle()"
            >
              <rpl-icon :symbol="menuButton[menuState].icon" color="white"></rpl-icon>
              <span>{{ menuButton[menuState].text }}</span>
            </button>
            <!-- Logo -->
            <div v-if="!menuContentOpen && logo" class="rpl-site-header__title">
              <rpl-link :href="logo.url">
                <img :src="logo.image" :alt="logo.alt" />
              </rpl-link>
            </div>
          </div>
          <!-- Top Menu -->
          <div
            v-if="(searchState === 'closed') && ((menuContentOpen && (menuState === 'opened')) || menuLayout === 'horizontal')"
            class="rpl-site-header__menu-container"
            :class="{
              'rpl-site-header__menu-container--horizontal': (menuLayout === 'horizontal'),
              'rpl-site-header__menu-container--vertical': (menuLayout === 'vertical')
            }"
          >
            <div class="rpl-site-header__menu">
              <rpl-menu
                :menu="links"
                :layout="menuLayout"
                title="Main Menu"
                :open="(menuState === 'opened')"
                @rootMenuClicked="rootMenuClicked"
              />
            </div>
          </div>
          <!-- Search Button -->
          <button v-if="showSearch" @click="searchToggle()" class="rpl-site-header__btn rpl-site-header__btn--search" :class="{'rpl-site-header__btn--search-open' : (searchState === 'opened')}">
            <span>{{ searchButton[searchState].text }}</span>
            <rpl-icon :symbol="searchButton[searchState].icon" color="white" />
          </button>
        </div>
        <!-- Search Content -->
        <div v-if="menuContentOpen && searchState == 'opened'" class="rpl-site-header__search-container">
          <rpl-search :terms="searchTerms" @search="searchFunc" />
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
import RplMenu from './menu'
import RplSearch from './search'
import RplIcon from '@dpc-sdp/ripple-icon'
import RplLink from '@dpc-sdp/ripple-link'

export default {
  name: 'RplSiteHeader',
  props: {
    logo: Object,
    links: Array,
    breakpoint: Number,
    searchTerms: Array,
    sticky: Boolean,
    hideOnScroll: { default: true, type: Boolean },
    showSearch: { default: false, type: Boolean }
  },
  components: {
    RplIcon,
    RplLink,
    RplMenu,
    RplSearch
  },
  data: function () {
    return {
      menuContentOpen: false,
      searchState: 'closed',
      menuState: 'closed',
      lastRootMenuClicked: -1,
      headerVisible: true,
      lastScrollTop: 0,
      stickyActive: false,
      offsetTop: 0,
      menuWideEnabled: null,
      menuLayout: 'vertical',
      menuButton: {
        opened: {
          text: 'Close menu',
          icon: 'close'
        },
        closed: {
          text: 'Menu',
          icon: 'hamburger'
        }
      },
      searchButton: {
        opened: {
          text: 'Close search',
          icon: 'close'
        },
        closed: {
          text: 'Search website',
          icon: 'search'
        }
      }
    }
  },
  watch: {
    // Close menu after vue route changed.
    $route: function (to, from) {
      if (this.menuContentOpen) {
        // Remove focus on any links - this fixes a bug in IE11 [SDPA-1178].
        document.activeElement.blur()

        // Close the menu / search.
        if (this.menuState === 'opened') {
          this.menuToggle()
        } else if (this.searchState === 'opened') {
          this.searchToggle()
        }
      }
    }
  },
  methods: {
    searchToggle: function () {
      this.menuContentOpen = !(this.menuContentOpen && this.searchState === 'opened')
      this.searchState = this.menuContentOpen ? 'opened' : 'closed'
      this.menuState = 'closed'
      this.$emit('open', this.menuContentOpen)
    },
    menuToggle: function () {
      this.menuContentOpen = !(this.menuContentOpen && this.menuState === 'opened')
      this.searchState = 'closed'
      this.menuState = this.menuContentOpen ? 'opened' : 'closed'
      this.$emit('open', this.menuContentOpen)
    },
    windowResize: function (e) {
      var w = window.innerWidth || document.documentElement.clientWidth
      if (w >= this.breakpoint && (this.menuWideEnabled || this.menuWideEnabled === null)) {
        // Desktop.
        this.menuWideEnabled = false
        this.menuLayout = 'horizontal'
        // Close menu on vertical -> horizontal: avoids incorrect display if vertical is on root.
        if (this.menuState === 'opened' && this.menuContentOpen) {
          this.menuState = 'closed'
          this.menuContentOpen = false
        }
      } else if (w < this.breakpoint && (!this.menuWideEnabled || this.menuWideEnabled === null)) {
        // Mobile.
        this.menuWideEnabled = true
        this.menuLayout = 'vertical'
      }
    },
    rootMenuClicked: function (rootMenuIndex) {
      this.menuContentOpen = !(this.menuContentOpen && this.lastRootMenuClicked === rootMenuIndex)
      this.menuState = this.menuContentOpen ? 'opened' : 'closed'
      this.lastRootMenuClicked = rootMenuIndex
      this.$emit('open', this.menuContentOpen)
    },
    searchFunc: function (value) {
      this.$emit('search', value)
    },
    scroll: function () {
      let scrollTop = window.pageYOffset || document.documentElement.scrollTop

      if (this.stickyActive === false && scrollTop > this.$el.offsetTop) {
        // reserve the offsetTop
        this.offsetTop = this.$el.offsetTop
        // When scroll header to top, make header sticky
        this.stickyActive = true
      } else if (this.stickyActive === true && scrollTop > this.offsetTop) {
        this.stickyActive = true
      } else {
        this.stickyActive = false
      }

      if (scrollTop > this.lastScrollTop && this.stickyActive) {
        // scroll down and is sticky
        this.headerVisible = false
      } else {
        // scroll up or is not sticky
        this.headerVisible = true
      }
      this.lastScrollTop = scrollTop <= 0 ? 0 : scrollTop
    }
  },
  mounted: function () {
    if (process.browser) {
      window.addEventListener('resize', this.windowResize)
      this.windowResize()
      if (this.hideOnScroll) {
        window.addEventListener('scroll', this.scroll)
      }
    }
  },
  beforeDestroy: function () {
    if (process.browser) {
      window.removeEventListener('resize', this.windowResize)
      if (this.hideOnScroll) {
        window.removeEventListener('scroll', this.scroll)
      }
    }
  }
}
</script>

<style lang="scss">
  @import "~@dpc-sdp/ripple-global/style";
  @import "scss/site_header";

  $rpl-site-header-logo-width: auto !default;
  $rpl-site-header-text-color: rpl-color('white') !default;
  $rpl-site-header-border-radius: rem(4px) !default;
  $rpl-site-header-background-color: rpl-color('primary') !default;
  $rpl-site-header-background-color-open: rpl-color('dark_primary') !default;
  $rpl-site-header-top-padding: ($rpl-space * 6) ($rpl-space * 5) !default;
  $rpl-site-header-menu-toggle-border-right: 1px solid rpl-color('white') !default;
  $rpl-site-header-menu-toggle-border-spacing: $rpl-space-2 !default;
  $rpl-site-header-menu-toggle-icon-margin: auto $rpl-space-2 auto 0 !default;
  $rpl-site-header-search-toggle-icon-margin: auto 0 auto $rpl-space-2 !default;

  .rpl-site-header {
    $root: &;
    @include rpl_body;
    position: absolute;
    z-index: $rpl-zindex-header;
    padding: $rpl-header-horizontal-padding-xs;
    box-sizing: border-box;
    width: 100%;

    @include rpl_breakpoint('s') {
      padding: $rpl-header-horizontal-padding-s;
    }

    &__inner {
      overflow: hidden;
      background-color: $rpl-site-header-background-color;
      border-radius: $rpl-site-header-border-radius;
      transition: height .25s;
      height: $rpl-site-header-top-height-s;
      @include rpl_breakpoint('m') {
        height: $rpl-site-header-top-height-l;
      }
    }

    &--sticky:not(#{$root}--open) {
      position: fixed;
      top: 0;
    }

    &--open {
      position: fixed;
      top: 0;
      height: 100vh;

      #{$root}__inner {
        margin: 0;
        border-radius: rem(4px);
        background-color: $rpl-site-header-background-color-open;
        height: 100%;
      }
    }

    &__top {
      display: flex;
      align-items: center;
      justify-content: space-between;
      box-sizing: border-box;
      padding: $rpl-site-header-top-padding;
      border-radius: $rpl-site-header-border-radius;
      height: $rpl-site-header-top-height-s;
      @include rpl_breakpoint('m') {
        height: $rpl-site-header-top-height-l;
      }

      &__title {
        font-size: rpl-font-size(l);
      }
    }

    &__logo-container {
      display: flex;
      align-items: center;

      .rpl-link {
        display: flex;
        flex-flow: column;
      }

      img {
        width: $rpl-site-header-logo-width;
        margin-left: $rpl-site-header-menu-toggle-border-spacing;
      }
    }

    // Menu Container - changes for vert / horizontal
    &__menu-container {
      &--vertical {
        width: 100%;
        position: absolute;
        bottom: 0;
        left: $rpl-header-horizontal-padding-xs;
        right: $rpl-header-horizontal-padding-xs;
        overflow-x: hidden;
        overflow-y: auto;
        -webkit-overflow-scrolling: touch;
        top: $rpl-site-header-top-height-s;

        @include rpl_breakpoint('s') {
          left: $rpl-header-horizontal-padding-s;
          right: $rpl-header-horizontal-padding-s;
        }
      }

      &--horizontal {
        margin-left: auto;
      }
    }

    &__search-container {
        position: relative;
        background-color: $rpl-site-header-background-color-open;
        overflow-x: hidden;
        overflow-y: auto;
        -webkit-overflow-scrolling: touch;
        top: 0;
        height: calc(100vh - #{$rpl-site-header-top-height-s});
        @include rpl_breakpoint('l') {
          height: calc(100vh - #{$rpl-site-header-top-height-l});
        }
    }

    &__btn {
      background-color: transparent;
      border: 0;
      padding: 0;
      display: flex;
      align-items: center;
      cursor: pointer;

      &--menu {
        padding-right: $rpl-site-header-menu-toggle-border-spacing;
        border-right: $rpl-site-header-menu-toggle-border-right;
        @include rpl_breakpoint('l') {
          display: none;
        }

        .rpl-icon {
          margin: $rpl-site-header-menu-toggle-icon-margin;
        }
      }

      &--menu-open {
        border-right: 0;
        @include rpl_breakpoint('l') {
          display: flex;
        }
      }

      &--search {
        span {
          @include rpl_breakpoint_down('m') {
            @include rpl_visually_hidden;
          }
          @include rpl_breakpoint('m') {
            margin-right: $rpl-space;
            display: block;
          }
          @include rpl_breakpoint_between('l', 'xl') {
            @include rpl_visually_hidden;
          }
          @include rpl_breakpoint('xl') {
            display: block;
          }
        }

        .rpl-icon {
          margin: $rpl-site-header-search-toggle-icon-margin;
        }
      }

      &--search-open {
        span {
          display: flex;
        }
      }

      span {
        @include rpl_typography_font('xs', 1em, 'medium');
        color: $rpl-site-header-text-color;
      }

      .rpl-icon {
        display: inline;
      }
    }
  }
</style>
