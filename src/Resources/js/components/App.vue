 <template lang="pug">
#main-wrapper
  #loading-overlay(v-if="isPageLoading")
    span.loading-icon

  #main-header
    .container.is-fluid
      nav.navbar(
        role="navigation"
        aria-label="main navigation"
      )
        .navbar-brand

          router-link.navbar-item(
            id="app-name"
            :to="{ name: 'issues' }"
          )

            img(
              :src="bugphixLogo"
              title="Bugphix logo"
              alt="Bugphix logo"
            )

          a.navbar-burger.burger(
            role="button"
            class="navbar-burger burger"
            aria-label="menu"
            aria-expanded="false"
            data-target="project-header-dropdown"
            @click="toggleMenu = !toggleMenu"
          )
            span(aria-hidden="true")
            span(aria-hidden="true")
            span(aria-hidden="true")

        .navbar-menu(
          :class="{'is-active' : toggleMenu}"
        )
          .navbar-start
            router-link.navbar-item(
              v-for="(item, key) in menuLinks"
              :key="`menu-links-${key}`"
              :to="{ name:key }"
            ) {{item}}


          .navbar-end(v-if="logoutUrl")
            .navbar-item.has-dropdown.is-hoverable(id="project-header-dropdown")
              a.navbar-link {{menuProjectName}}

              .navbar-dropdown.is-right
                a.navbar-item(
                    v-for="(item, key) in projectListOptions"
                    :key="`project-list-${key}`"
                    :class="item.project_id === getActiveProjectId ? 'is-active' : ''"
                    @click="changeProject(item.project_id)"
                  ) {{item.project_name}} – {{ item.project_id }} {{ item.is_active === false ? '(deactivated)' : '' }}

            .navbar-item
              .buttons
                a.button.is-link.is-light(
                  :href="logoutUrl"
                ) Logout

  #main-content
    router-view

  .footer
    .content.has-text-centered
      p
        a(
          href="https://github.com/bugphix/bugphix-laravel"
          target="_blank"
        ) Bugphix
        span.copyright © 2020. All rights reserved –
        a(
          href="https://github.com/jericizon"
          target="_blank"
        ) Jeric

  .notification-groups
    notifications(position="bottom right")

</template>

<script>

import { mapGetters, mapMutations, mapState, mapActions } from 'vuex';

export default {
  name: 'app',
  data() {
    return {
      toggleMenu: false,
      logoutUrl: '',
      menuLinks: {
        'issues': 'Issues',
        'projects': 'Projects',
        'users': 'Users',
      },
    };
  },
  beforeMount(){
    const projectId = this.$route.params.projectId || '';
    this.bootApplication(projectId);
  },
  mounted(){
    this.setActivePage(this.$route.name);
    this.logoutUrl = window.Bugphix.logout_url
  },
  methods: {
    ...mapMutations([
      'setActivePage',
    ]),
    ...mapActions([
      'bootApplication',
      'setActiveProject',
    ]),
    changeProject(projectId){
      this.setActiveProject(projectId);
      window.location.reload();
    },
  },
  computed: {
    ...mapState([
      'isPageLoading',
      'projectListOptions',
    ]),
    ...mapGetters([
      'getActiveProjectId',
      'getActiveProjectName',
    ]),
    menuProjectName(){
      if(!this.getActiveProjectName) return 'Loading...';
      return `Active project: (${this.getActiveProjectName})`
    },
    bugphixLogo(){
      const {assets_url} = window.Bugphix;
      return `${assets_url}/images/logo.png`
    }
  },
  watch: {
    $route(to, from) {
      // react to route changes...
      // console.log('$route', to, from)
      this.setActivePage(to.name);
    }
  },
};

</script>

<style lang="scss">
  #main-wrapper{
    background: #fff;

    #app-name{
      img{
        max-height: 50px;
      }
    }

    #main-content{
      background: #f2f7ff;
    }

    #loading-overlay {
      width: 100%;
      top: 55px;
      right: 0;
      height: 100%;
      position: fixed;
      background: rgba(255, 255, 255, 0.8);
      z-index: 20;
      display: flex;
      align-items: center;
      justify-content: center;
    }

    #main-header{
      box-shadow: 20px 0px 20px rgba(41, 121, 255, 0.2);
    }

    #main-content{
      padding: 40px 0;
      min-height: calc(100vh - 115px);
    }

    .footer{
      padding: 20px;

      span.copyright {
        margin: 0 5px;
      }
    }
  }

  #project-header-dropdown{
    .button {
      border: none;
      font-weight: 600;
      font-size: 1.25em;
      box-shadow: none;
    }
  }
</style>
