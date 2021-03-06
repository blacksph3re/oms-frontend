<template>
  <aside class="menu app-sidebar animated" :class="{ slideInLeft: show, slideOutLeft: !show, 'is-menu-opened': navbar.menuOpened }">
    <div v-for="(category, categoryIndex) in menu" v-bind:key="category.categoryName">
      <p class="menu-label">
        {{ category.categoryName }}
      </p>
      <ul class="menu-list">
        <li v-for="(item, index) in category.components" exact="true" v-bind:key="item.name" v-if="userHasPermissions(item)">
          <router-link :to="{ name: item.name }" v-if="item.name" :aria-expanded="isExpanded(item) ? 'true' : 'false'" @click.native="toggle(index, categoryIndex, item)">
            <span><i :class="['fas', item.icon]"></i></span>
            {{ item.label }}
            <span class="icon is-small is-angle" v-if="item.children && item.children.length">
              <i class="fa fa-angle-down"></i>
            </span>
          </router-link>
          <a :aria-expanded="isExpanded(item)" v-else @click="toggle(index, categoryIndex, item)">
            <span><i :class="['fas', item.icon]"></i></span>
            {{ item.label }}
            <span class="icon is-small is-angle" v-if="item.children && item.children.length">
              <i class="fa fa-angle-down"></i>
            </span>
          </a>

          <ul v-show="isExpanded(item)">
            <li v-for="subItem in item.children" v-bind:key="subItem.name" v-if="userHasPermissions(subItem)">
              <router-link :to="{ name: subItem.name }" exact="true">
                {{ subItem.label }}
              </router-link>
            </li>
          </ul>
        </li>
      </ul>
    </div>
  </aside>
</template>

<script>
import { mapGetters, mapActions } from 'vuex'

export default {
  props: {
    show: Boolean
  },
  data () {
    return {
      isReady: false
    }
  },
  mounted () {
    let route = this.$route
    if (route.name) {
      this.isReady = true
      this.shouldExpandMatchItem(route)
    }
  },
  computed: mapGetters({
    menu: 'menuitems',
    permissions: 'permissions',
    navbar: 'navbar',
    device: 'device'
  }),
  methods: {
    ...mapActions([
      'expandMenu',
      'toggleSidebar',
      'toggleNavbarMenu'
    ]),
    userHasPermissions (item) {
      // If item has no permissions attribute, show it.
      if (!item.permissions) {
        return true
      }

      // Otherwise, check if user has such a permission.
      return item.permissions.some(itemPermission =>
        this.permissions.find(userPermission => userPermission.combined.includes(itemPermission)))
    },
    isExpanded (item) {
      return item.expanded
    },
    toggle (index, categoryIndex, item) {
      this.expandMenu({
        index: index,
        categoryIndex: categoryIndex,
        expanded: !item.expanded
      })
    },
    shouldExpandMatchItem (route) {
      let matched = route.matched
      if (matched.length === 0) {
        return
      }

      let lastMatched = matched[matched.length - 1]
      let parent = lastMatched.parent || lastMatched
      const isParent = parent === lastMatched

      if (isParent) {
        const p = this.findParentFromMenu(route)
        if (p) {
          parent = p
        }
      }

      if ('expanded' in parent && !isParent) {
        this.expandMenu({
          item: parent,
          expanded: true
        })
      }
    },
    findParentFromMenu (route) {
      const menu = this.menu
      for (let i = 0, l = menu.length; i < l; i++) {
        const item = menu[i]
        const k = item.children && item.children.length
        if (k) {
          for (let j = 0; j < k; j++) {
            if (item.children[j].name === route.name) {
              return item
            }
          }
        }
      }
    }
  },
  watch: {
    $route (route) {
      this.isReady = true
      this.shouldExpandMatchItem(route)
      if (this.device.isMobile) {
        this.toggleSidebar({ opened: false })
        this.toggleNavbarMenu({ opened: false })
      }
    }
  }

}
</script>

<style lang="scss">
@import "~bulma/sass/utilities/initial-variables";
@import "~bulma/sass/utilities/derived-variables";
@import '~bulma/sass/utilities/mixins';

.app-sidebar {
  position: fixed;
  z-index: 1;
  top: 53px;
  left: 0;
  bottom: 0;
  padding: 20px 0 50px;
  width: 180px;
  min-width: 45px;
  max-height: 100vh;
  height: calc(100% - 50px);
  background: #FFF;
  box-shadow: 0 2px 3px rgba(17, 17, 17, 0.1), 0 0 0 1px rgba(17, 17, 17, 0.1);
  overflow-y: auto;
  overflow-x: hidden;

  &.is-menu-opened {
    @include mobile() {
      transform: translate3d(-180px, 0, 0);
      padding-top: 160px;
    }

    @include tablet-only() {
      padding-top: 160px
    }
  }

  .icon {
    vertical-align: baseline;
    &.is-angle {
      position: absolute;
      right: 10px;
      transition: transform .377s ease;
    }
  }

  .menu-label {
    padding-left: 10px;
    margin-top: 1em;
  }

  .menu-list {
    li a {
      &[aria-expanded="true"] {
        .is-angle {
          transform: rotate(180deg);
        }
      }
    }

    li a + ul {
      margin: 0 10px 0 15px;
    }
  }

}
</style>
