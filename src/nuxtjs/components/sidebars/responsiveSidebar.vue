<template>
  <div :class="drawerMenuClass()">
    <div class="side-menu-nav">
      <div class="nav-icon" @click.stop="onSetHideDrawer()">
        <i class="icon-cross2"></i>
      </div>
    </div>
    <a class="sidebar-menu-profile mt-0">
      <img
        class="sidebar-menu-profile-img"
        src="~/assets/hc-libs/images/main/face_default.png"
        alt="user-profile"
      />
      <div class="sidebar-menu-profile-item">
        <div class="sidebar-menu-profile-item-name">Firstname Lastname</div>
        <div class="sidebar-menu-profile-item-email">example@email.com</div>
      </div>
    </a>
    <div class="sidebar-menu-main-disabled">
      <i class="mi-chat-bubble sidebar-menu-main-icon"></i>
      <span>Chatbot Training</span>
    </div>
    <div class="collapse-chatbot-sidebar">
      <div
        class="sidebar-menu-main-item"
        @click="linkTo('/')"
        @click.stop="onSetHideDrawer()"
      >
        New Word
      </div>
      <div
        class="sidebar-menu-main-item"
        @click="linkTo('/chatbot-training/learned-word')"
        @click.stop="onSetHideDrawer()"
      >
        Learned Word
      </div>
      <div
        class="sidebar-menu-main-item"
        @click="linkTo('/chatbot-training/feature-management')"
        @click.stop="onSetHideDrawer()"
      >
        Feature Management
      </div>
      <div
        class="sidebar-menu-main-item"
        @click="linkTo('/chatbot-training/group-management')"
        @click.stop="onSetHideDrawer()"
      >
        Group Management
      </div>
    </div>
    <div @click.stop="onSetHideDrawer()">
      <nuxt-link
        :to="localePath('/dashboard/all-patient-group')"
        class="sidebar-menu-main"
      >
        <i class="mi-dashboard sidebar-menu-main-icon"></i>
        <span>Dashboard</span>
      </nuxt-link>
    </div>
    <div class="sidebar-menu-main-signout">
      <a href="#">
        <img
          src="~assets/hc-libs/images/main/logout.svg"
          width="20"
          class="sidebar-menu-main-icon"
        />
        <span>Signout</span>
      </a>
    </div>
  </div>
</template>

<script>
export default {
  name: 'HealthcareSideMenu',
  props: ['isDrawer'],
  data: () => ({
    isOpen: false,
    menus: [],
  }),
  computed: {},
  watch: {
    isDrawer(newVal) {
      this.isOpen = newVal
    },
  },
  beforeMount() {},
  methods: {
    drawerMenuClass() {
      return {
        'side-menu-container-close': !this.isOpen,
        'side-menu-container-open': this.isOpen,
      }
    },
    onSetHideDrawer() {
      this.$emit('onHideDrawer', this.isOpen)
    },
    linkTo(path) {
      this.$router.push(path)
    },
  },
}
</script>

<style lang="scss">
.side-menu {
  @mixin container {
    display: flex;
    position: fixed;
    flex-direction: column;
    width: 256px;
    height: 100%;
    border-right: 1px solid #ececec;
    width: 256px;
    background: #fff;
    border-bottom: 1px solid #dddddd;
    z-index: 99;
  }
  &-container-open {
    @include container();
  }

  &-container-close {
    @include container();
    margin-left: -300px;
  }

  &-nav {
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: flex-end;
    height: 56px;
    width: 100%;
    border-bottom: 1px solid #dddddd;
  }
}

@media only screen and (max-width: 1200px) {
  .side-menu {
    &-container-open {
      z-index: 999;
      box-shadow: 6px 0px 18px 0px rgba(0, 0, 0, 0.06);
      transition: all 0.2s linear;
    }

    &-container-close {
      z-index: 999;
      box-shadow: 6px 0px 18px 0px rgba(0, 0, 0, 0.06);
      transition: all 0.2s linear;
    }
  }
}
</style>
