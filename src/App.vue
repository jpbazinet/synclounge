<template>
  <v-app>
    <TheSidebarLeft />
    <router-view name="rightSidebar" />

    <v-app-bar
      app
      scroll-off-screen
      :scroll-threshold="1"
      style="z-index: 5;"
    >
      <v-app-bar-nav-icon @click="SET_LEFT_SIDEBAR_OPEN" />

      <router-link
        :to="{ name: 'RoomCreation' }"
      >
        <picture>
          <source
            srcset="@/assets/images/logos/movienight.png"
            :media="smallLogoMedia"
          >
          <img
            height="42"
            src="@/assets/images/logos/movienight.png"
            style="vertical-align: middle;"
          >
        </picture>
      </router-link>

      <TheNowPlayingChip
        v-if="showNowPlaying"
        class="pl-4"
      />

      <v-spacer />

