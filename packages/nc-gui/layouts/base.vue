<script lang="ts" setup>
import { computed, navigateTo, ref, useGlobal, useNuxtApp, useRoute, useSidebar } from '#imports'

const { signOut, signedIn, isLoading, user, currentVersion } = useGlobal()

useSidebar('nc-left-sidebar', { hasSidebar: false })

const route = useRoute()

const email = computed(() => user.value?.email ?? '---')

const hasSider = ref(false)

const sidebar = ref<HTMLDivElement>()

const logout = async () => {
  await signOut()
  navigateTo('/signin')
}

const { hooks } = useNuxtApp()

const isDashboard = computed(() => !!route.params.projectType)

/** when page suspensions have finished, check if a sidebar element was teleported into the layout */
hooks.hook('page:finish', () => {
  if (sidebar.value) {
    hasSider.value = sidebar.value?.children.length > 0
  }
})
</script>

<template>
  <a-layout id="nc-app" has-sider>
    <Transition name="slide">
      <div v-show="hasSider" id="nc-sidebar-left" ref="sidebar" />
    </Transition>

    <a-layout class="!flex-col">
      <a-layout-header v-if="!route.meta.public && signedIn && !route.meta.hideHeader" class="nc-navbar">
        <div
          v-if="!route.params.projectType"
          v-e="['c:navbar:home']"
          data-testid="nc-noco-brand-icon"
          class="transition-all duration-200 p-2 cursor-pointer transform hover:scale-105 nc-noco-brand-icon"
          @click="navigateTo('/')"
        >
          <a-tooltip placement="bottom">
            <template #title>
              {{ currentVersion }}
            </template>
            <div class="flex items-center gap-2">
              <img v-if="!isDashboard" width="120" alt="NocoDB" src="~/assets/img/brand/nocodb-full-color.png" />
              <img v-else width="25" alt="NocoDB" src="~/assets/img/icons/512x512.png" />
            </div>
          </a-tooltip>
        </div>

        <div class="!text-white flex justify-center">
          <div v-show="isLoading" class="flex items-center gap-2 ml-3" data-testid="nc-loading">
            {{ $t('general.loading') }}

            <MdiReload :class="{ 'animate-infinite animate-spin': isLoading }" />
          </div>
        </div>

        <div class="flex-1" />

        <LazyGeneralReleaseInfo />

        <a-tooltip placement="bottom" :mouse-enter-delay="1">
          <template #title> Switch language</template>

          <div class="flex pr-4 items-center text-white">
            <LazyGeneralLanguage class="cursor-pointer text-2xl hover:text-accent" />
          </div>
        </a-tooltip>

        <template v-if="signedIn">
          <a-dropdown :trigger="['click']" overlay-class-name="nc-dropdown-user-accounts-menu">
            <MdiDotsVertical
              data-testid="nc-menu-accounts"
              class="md:text-xl cursor-pointer hover:text-accent nc-menu-accounts"
              @click.prevent
            />

            <template #overlay>
              <a-menu class="!py-0 leading-8 !rounded">
                <a-menu-item key="0" data-testid="nc-menu-accounts__user-settings" class="!rounded-t">
                  <nuxt-link v-e="['c:navbar:user:email']" class="nc-project-menu-item group !no-underline" to="/account/users">
                    <MdiAccountCircleOutline class="mt-1 group-hover:text-accent" />&nbsp;
                    <div class="prose group-hover:text-primary">
                      <div>Account</div>
                      <div class="text-xs text-gray-500">{{ email }}</div>
                    </div>
                  </nuxt-link>
                </a-menu-item>

                <a-menu-divider class="!m-0" />
                <!--                <a-menu-item v-if="isUIAllowed('appStore')" key="0" class="!rounded-t">
                  <nuxt-link
                    v-e="['c:settings:appstore', { page: true }]"
                    class="nc-project-menu-item group !no-underline"
                    to="/admin/users"
                  >
                    <MdiShieldAccountOutline class="mt-1 group-hover:text-accent" />&nbsp;

                    &lt;!&ndash; todo: i18n &ndash;&gt;
                    <span class="prose group-hover:text-primary">Account management</span>
                  </nuxt-link>
                </a-menu-item>

                <a-menu-divider class="!m-0" /> -->

                <a-menu-item key="1" class="!rounded-b group">
                  <div v-e="['a:navbar:user:sign-out']" class="nc-project-menu-item group" @click="logout">
                    <MdiLogout class="group-hover:text-accent" />&nbsp;

                    <span class="prose group-hover:text-primary">
                      {{ $t('general.signOut') }}
                    </span>
                  </div>
                </a-menu-item>
              </a-menu>
            </template>
          </a-dropdown>
        </template>
      </a-layout-header>

      <a-tooltip placement="bottom">
        <template #title> Switch language</template>

        <LazyGeneralLanguage v-if="!signedIn && !route.params.projectId" class="nc-lang-btn" />
      </a-tooltip>

      <div class="w-full h-full overflow-hidden">
        <slot />
      </div>
    </a-layout>
  </a-layout>
</template>

<style lang="scss">
.nc-lang-btn {
  @apply color-transition flex items-center justify-center fixed bottom-10 right-10 z-99 w-12 h-12 rounded-full shadow-md shadow-gray-500 p-2 !bg-primary text-white ring-opacity-100 active:(ring ring-accent) hover:(ring ring-accent);

  &::after {
    @apply rounded-full absolute top-0 left-0 right-0 bottom-0 transition-all duration-150 ease-in-out bg-primary;
    content: '';
    z-index: -1;
  }

  &:hover::after {
    @apply transform scale-110 ring ring-accent ring-opacity-100;
  }

  &:active::after {
    @apply ring ring-accent ring-opacity-100;
  }
}

.nc-navbar {
  @apply flex !bg-white items-center !pl-2 !pr-5;
}
</style>
