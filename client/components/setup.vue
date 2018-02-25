<template lang="pug">
  div
    .container
      .content(v-cloak)

        //- ==============================================
        //- WELCOME
        //- ==============================================

        template(v-if='state === "welcome"')
          .panel
            h2.panel-title.is-featured
              span Welcome!
              i(v-if='loading')
            .panel-content.is-text
              .welcome
                img(src='svg/logo-wikijs.svg', alt='Wiki.js Logo')
                h2 A modern, lightweight and powerful wiki app built on NodeJS, Git and Markdown
              p This installation wizard will guide you through the steps needed to get your wiki up and running in no time!
              p Detailed information about installation and usage can be found on the #[a(href='https://wiki.requarks.io/docs') official documentation site]. #[br] Should you have any question or would like to report something that doesn't look right, feel free to create a new issue on the #[a(href='https://github.com/Requarks/wiki/issues') GitHub project].
            .panel-content.form-sections
              section
                p
                  svg.icons.is-18.is-outlined.has-right-pad.is-text: use(xlink:href='#nc-cd-reader')
                  span You are about to install Wiki.js #[strong {{wikiVersion}}].
              section
                p.control.is-fullwidth
                  input#ipt-telemetry(type='checkbox', v-model='conf.telemetry', name='ipt-telemetry')
                  label.label(for='ipt-telemetry') Allow Telemetry
                  span.desc Help Wiki.js developers improve this app with anonymized #[a(href='https://wiki.requarks.io/docs/telemetry') telemetry].
                p.control.is-fullwidth
                  input#ipt-upgrade(type='checkbox', v-model='conf.upgrade', name='ipt-upgrade')
                  label.label(for='ipt-upgrade') Upgrade from Wiki.js 1.x
                  span.desc Check this box if you are upgrading from Wiki.js 1.x and wish to migrate your existing data.
            .panel-footer
              .progress-bar: div(v-bind:style='{width: currentProgress}')
              button.button.is-small.is-light-blue(v-on:click='proceedToSyscheck', v-bind:disabled='loading') Start

        //- ==============================================
        //- SYSTEM CHECK
        //- ==============================================

        template(v-else-if='state === "syscheck"')
          .panel
            h2.panel-title.is-featured
              span Wiki.js
              i(v-if='loading')
            .panel-content.is-text
              .is-logo
                svg.icons.is-64: use(xlink:href='#nc-metrics')
                h4 System Check
              p(v-if='loading') #[svg.icons.is-24.is-text: use(xlink:href='#wk-msdots')] Checking your system for compatibility...
              p(v-if='!loading && syscheck.ok')
                ul
                  li(v-for='rs in syscheck.results') #[svg.icons.is-18.is-text: use(xlink:href='#nc-check-bold')] {{rs}}
              p(v-if='!loading && syscheck.ok')
                svg.icons.is-18.is-text: use(xlink:href='#nc-check-bold')
                strong  Looks good! No issues so far.
              p(v-if='!loading && !syscheck.ok') #[svg.icons.is-18.is-text: use(xlink:href='#nc-square-remove-12')] Error: {{ syscheck.error }}
            .panel-footer
              .progress-bar: div(v-bind:style='{width: currentProgress}')
              button.button.is-small.is-light-blue.is-outlined(v-on:click='proceedToWelcome', v-bind:disabled='loading') Back
              button.button.is-small.is-teal(v-on:click='proceedToSyscheck', v-if='!loading && !syscheck.ok') Check Again
              button.button.is-small.is-red.is-outlined(v-on:click='proceedToGeneral', v-if='!loading && !syscheck.ok') Continue Anyway
              button.button.is-small.is-light-blue(v-on:click='proceedToGeneral', v-if='loading || syscheck.ok', v-bind:disabled='loading') Continue

        //- ==============================================
        //- GENERAL
        //- ==============================================

        template(v-else-if='state === "general"')
          .panel
            h2.panel-title.is-featured
              span Wiki.js
              i(v-if='loading')
            .panel-content.form-sections
              section
                .is-logo
                  svg.icons.is-64: use(xlink:href='#nc-webpage')
                  h4 General Information
                p.control.is-fullwidth
                  label.label Site Title
                  input(type='text', placeholder='e.g. Wiki', v-model='conf.title', data-vv-scope='general', name='ipt-title', v-validate='{ required: true, min: 2 }')
                  span.desc The site title will appear in the top left corner on every page and within the window title bar.
              section.columns
                .column.is-half
                  p.control
                    label.label Site UI Language
                    select(v-model='conf.lang')
                      option(:value='lg.id', v-for='lg in langs') {{lg.name}}
                    span.desc The language in which navigation, help and other UI elements will be displayed.
                .column.is-half
                  p.control.is-fullwidth
                    label.label Site Relative URL Path
                    input(type='text', placeholder='/', v-model='conf.path', data-vv-scope='general', name='ipt-path', v-validate='{ required: true, min: 1 }')
                    span.desc The relative path to your wiki. Unless you configure a reverse proxy in front of Wiki.js to handle requests made to a sub-directory, #[strong it is recommended to leave the default value].
              section.columns
                .column.is-half
                  p.control
                    label.label Server Port
                    input(type='text', placeholder='e.g. 80', v-model.number='conf.port', data-vv-scope='general', name='ipt-port', v-validate='{ required: true }')
                    span.desc The port on which Wiki.js will listen to. Usually port 80 if connecting directly, or a random port (e.g. 3000) if using a web server in front of it. Set #[strong $(PORT)] to use the PORT environment variable.
                .column.is-half
                  p.control.is-fullwidth
                    input#ipt-public(type='checkbox', v-model='conf.public', data-vv-scope='general', name='ipt-public')
                    label.label(for='ipt-public') Public Access
                    span.desc Should the site be accessible (read only) without login.
                  p.control.is-fullwidth
                    input#ipt-selfregister(type='checkbox', v-model='conf.selfRegister', data-vv-scope='general', name='ipt-selfregister')
                    label.label(for='ipt-selfregister') Allow Self-Registration
                    span.desc Can users create their own account to gain access?
              section
                p.control.is-fullwidth
                  label.label Local Server Repository Path
                  input(type='text', placeholder='e.g. ./repo', v-model='conf.pathRepo', data-vv-scope='general', name='ipt-repopath', v-validate='{ required: true, min: 2 }')
                  span.desc The path where the local git repository will be created, used to store content in markdown files and uploads.#[br] #[strong It is recommended to leave the default value].
            .panel-footer
              .progress-bar: div(v-bind:style='{width: currentProgress}')
              button.button.is-small.is-light-blue.is-outlined(v-on:click='proceedToSyscheck', v-bind:disabled='loading') Back
              button.button.is-small.is-light-blue(v-on:click='proceedToConsiderations', v-bind:disabled='loading || errors.any("general")') Continue

        //- ==============================================
        //- CONSIDERATIONS
        //- ==============================================

        template(v-else-if='state === "considerations"')
          .panel
            h2.panel-title.is-featured
              span Wiki.js
              i(v-if='loading')
            .panel-content.is-text
              .is-logo
                svg.icons.is-64: use(xlink:href='#nc-radar')
                h4 Important Considerations
              h3 Is Wiki.js going to be behind a web server (e.g. nginx / apache / IIS) or proxy?
              p
                ul
                  li - Make sure the upload limit is sufficient. Most web servers have a low limit (e.g. 2 MB) by default.
                  li - Do not rewrite URLs after the domain. This can cause unexpected issues in Wiki.js navigation.
                  li - Do not remove or alter the client IP when proxying the requests. This can cause the authentication brute force protection to engage unexpectedly.
              template(v-if='considerations.https')
                h3 The site will not be using HTTPS? #[svg.icons.is-20.is-outlined.animated.fadeOut.infinite: use(xlink:href='#nc-alert')]
                p The host URL you specified is not HTTPS. It is highly recommended to use HTTPS. You must use a web server / proxy (e.g. nginx / apache / IIS) in front of Wiki.js to use HTTPS. Wiki.js does not provide HTTPS handling by itself.
              template(v-if='considerations.port')
                h3 You are using a non-standard port.
                p If you are not planning on using a web server / proxy in front of Wiki.js, be aware that users will need to specify the port when accessing the wiki. Make sure this is the intended behavior. Otherwise set a standard HTTP port such as 80.
            .panel-footer
              .progress-bar: div(v-bind:style='{width: currentProgress}')
              button.button.is-small.is-light-blue.is-outlined(v-on:click='proceedToGeneral', v-bind:disabled='loading') Back
              button.button.is-small.is-light-blue(v-on:click='proceedToGit', v-bind:disabled='loading') Continue

        //- ==============================================
        //- GIT
        //- ==============================================

        template(v-else-if='state === "git"')
          .panel
            h2.panel-title.is-featured
              span Wiki.js
              i(v-if='loading')
            .panel-content.is-text
              .is-logo
                img(src='svg/logo-git.svg', alt='Git Logo')
                h4 Git Repository
              p Wiki.js stores article content and uploads locally on disk. All content is then regularly kept in sync with a remote git repository. This acts a backup protection and provides history / revert features. While optional, it is <strong>HIGHLY</strong> recommended to setup the remote git repository connection.
            .panel-content.form-sections
              section.columns
                .column.is-two-thirds
                  p.control.is-fullwidth
                    label.label Repository URL
                    input(type='text', placeholder='e.g. git@github.com/org/repo.git or https://github.com/org/repo', v-model='conf.gitUrl', data-vv-scope='git', name='ipt-giturl', v-validate='{ required: true, min: 5 }')
                    span.desc The full git repository URL to connect to.
                .column
                  p.control.is-fullwidth
                    label.label Branch
                    input(type='text', placeholder='e.g. master', v-model='conf.gitBranch', data-vv-scope='git', name='ipt-gitbranch', v-validate='{ required: true, min: 2 }')
                    span.desc The git branch to use when synchronizing changes.
              section.columns
                .column.is-one-third
                  p.control.is-fullwidth
                    label.label Authentication
                    select(v-model='conf.gitAuthType')
                      option(value='ssh') SSH using Private Key file (recommended)
                      option(value='sshenv') SSH using Private Key in environment variable
                      option(value='sshdb') SSH using Private Key in database
                      option(value='basic') Basic Credentials
                    span.desc The authentication method used to connect to your remote Git repository.
                  p.control.is-fullwidth
                    input#ipt-git-verify-ssl(type='checkbox', v-model='conf.gitAuthSSL')
                    label.label(for='ipt-git-verify-ssl') Verify SSL
                .column(v-show='conf.gitAuthType === "basic"')
                  p.control.is-fullwidth
                    label.label Username
                    input(type='text', v-model='conf.gitAuthUser')
                    span.desc The username for the remote git connection.
                .column(v-show='conf.gitAuthType === "basic"')
                  p.control.is-fullwidth
                    label.label Password
                    input(type='password', v-model='conf.gitAuthPass')
                    span.desc The password for the remote git connection.
                .column(v-show='conf.gitAuthType === "ssh"')
                  p.control.is-fullwidth
                    label.label Private Key location
                    input(type='text', placeholder='e.g. /etc/wiki/keys/git.pem', v-model='conf.gitAuthSSHKey')
                    span.desc The full path to the #[strong unencrypted] private key on disk.
                .column(v-show='conf.gitAuthType === "sshenv"')
                  p.control.is-fullwidth
                    label.label Private Key Environment Variable
                    input(type='text', placeholder='e.g. GIT_PRIVATE_KEY', v-model='conf.gitAuthSSHKeyEnv')
                    span.desc The environment variable containing the private key.
                .column(v-show='conf.gitAuthType === "sshdb"')
                  p.control.is-fullwidth
                    label.label Private Key Contents
                    textarea(v-model='conf.gitAuthSSHKeyDB')
                    span.desc Paste the contents of the private key in the above field
              section.columns
                .column.is-one-third
                  p.control.is-fullwidth
                    input#ipt-git-show-user-email(type='checkbox', v-model='conf.gitShowUserEmail')
                    label.label(for='ipt-git-show-user-email') Commit using User Email
                    span.desc When enabled, commits are made as the current user name and email. If unchecked, the current user name will still be used but the default commit author email will be used instead.
                .column
                  p.control.is-fullwidth
                    label.label Default Commit Author Email
                    input(type='text', placeholder='e.g. user@example.com', v-model.number='conf.gitServerEmail', data-vv-scope='git', name='ipt-gitsrvemail', v-validate='{ required: true, email: true }')
                    span.desc The default/fallback email to use when creating commits to the git repository.
            .panel-footer
              .progress-bar: div(v-bind:style='{width: currentProgress}')
              button.button.is-small.is-light-blue.is-outlined(v-on:click='proceedToGeneral', v-bind:disabled='loading') Back
              button.button.is-small.is-light-blue.is-outlined(v-on:click='conf.gitUseRemote = false; proceedToGitCheck()', v-bind:disabled='loading') Skip this step
              button.button.is-small.is-light-blue(v-on:click='conf.gitUseRemote = true; proceedToGitCheck()', v-bind:disabled='loading || errors.any("git")') Continue

        //- ==============================================
        //- GIT CHECK
        //- ==============================================

        template(v-else-if='state === "gitcheck"')
          .panel
            h2.panel-title.is-featured
              span Wiki.js
              i(v-if='loading')
            .panel-content.is-text
              .is-logo
                img(src='svg/logo-git.svg', alt='Git Logo')
                h4 Git Repository Check
              p(v-if='loading') #[svg.icons.is-24.is-text: use(xlink:href='#wk-msdots')] Verifying Git repository settings...
              p(v-if='!loading && gitcheck.ok')
                ul
                  li(v-for='rs in gitcheck.results') #[svg.icons.is-18.is-text: use(xlink:href='#nc-check-bold')] {{rs}}
              p(v-if='!loading && gitcheck.ok')
                svg.icons.is-18.is-text: use(xlink:href='#nc-check-bold')
                strong  Git settings are correct!
              p(v-if='!loading && !gitcheck.ok') #[svg.icons.is-18.is-text: use(xlink:href='#nc-square-remove')] Error: {{ gitcheck.error }}
            .panel-footer
              .progress-bar: div(v-bind:style='{width: currentProgress}')
              button.button.is-small.is-light-blue.is-outlined(v-on:click='proceedToGit', v-bind:disabled='loading') Back
              button.button.is-small.is-teal(v-on:click='proceedToGitCheck', v-if='!loading && !gitcheck.ok') Try Again
              button.button.is-small.is-light-blue(v-on:click='proceedToAdmin', v-if='loading || gitcheck.ok', v-bind:disabled='loading') Continue

        //- ==============================================
        //- ADMINISTRATOR ACCOUNT
        //- ==============================================

        template(v-else-if='state === "admin"')
          .panel
            h2.panel-title.is-featured
              span Wiki.js
              i(v-if='loading')
            .panel-content.is-text
              .is-logo
                svg.icons.is-64: use(xlink:href='#nc-man-black')
                h4 Administrator Account
              p A root administrator account will be created for local authentication. From this account, you can create or authorize more users.
            .panel-content.form-sections
              section
                p.control.is-fullwidth
                  label.label Administrator Email
                  input(type='text', placeholder='e.g. admin@example.com', v-model='conf.adminEmail', data-vv-scope='admin', name='ipt-adminemail', v-validate='{ required: true, email: true }')
                  span.desc The email address of the administrator account
              section.columns
                .column
                  p.control.is-fullwidth
                    label.label Password
                    input(type='password', v-model='conf.adminPassword', data-vv-scope='admin', name='ipt-adminpwd', v-validate='{ required: true, min: 8 }')
                    span.desc At least 8 characters long.
                .column
                  p.control.is-fullwidth
                    label.label Confirm Password
                    input(type='password', v-model='conf.adminPasswordConfirm', data-vv-scope='admin', name='ipt-adminpwd2', v-validate='{ required: true, confirmed: "ipt-adminpwd" }')
                    span.desc Verify your password again.
            .panel-footer
              .progress-bar: div(v-bind:style='{width: currentProgress}')
              button.button.is-small.is-light-blue.is-outlined(v-on:click='proceedToGit', v-bind:disabled='loading') Back
              button.button.is-small.is-light-blue(v-on:click='proceedToUpgrade', v-bind:disabled='loading || errors.any("admin")') Continue

        //- ==============================================
        //- UPGRADE FROM 1.x
        //- ==============================================

        template(v-else-if='state === "upgrade"')
          .panel
            h2.panel-title.is-featured
              span Wiki.js
              i(v-if='loading')
            .panel-content.is-text
              .is-logo
                svg.icons.is-64: use(xlink:href='#nc-spaceship')
                h4 Upgrade from Wiki.js 1.x
              p Migrating from a Wiki.js 1.x installation is quick and simple.
            .panel-content.form-sections
              section
                p.control.is-fullwidth
                  label.label Connection String to Wiki.js 1.x MongoDB database
                  input(type='text', placeholder='mongodb://', v-model='conf.upgMongo', data-vv-scope='upgrade', name='ipt-mongo', v-validate='{ required: true, min: 2 }')
                  span.desc A MongoDB database connection string where a Wiki.js 1.x installation is located. #[strong No alterations will be made to this database. ]
            .panel-footer
              .progress-bar: div(v-bind:style='{width: currentProgress}')
              button.button.is-small.is-light-blue.is-outlined(v-on:click='proceedToAdmin', v-bind:disabled='loading') Back
              button.button.is-small.is-light-blue(v-on:click='proceedToFinal', v-bind:disabled='loading || errors.any("general")') Continue

        //- ==============================================
        //- FINAL
        //- ==============================================

        template(v-else-if='state === "final"')
          .panel
            h2.panel-title.is-featured
              span Wiki.js
              i(v-if='loading')
            .panel-content.is-text
              .is-logo(v-if='loading')
                svg.icons.is-64: use(xlink:href='#wk-msdots')
                h4 Finalizing your installation...
              .is-logo(v-if='!loading && final.ok')
                svg.icons.is-64: use(xlink:href='#nc-check-bold')
                h4 Installation complete!
              p(v-if='!loading && final.ok'): strong Wiki.js was configured successfully and is now ready for use.
              p(v-if='!loading && final.ok') Click the #[strong Start] button below to access your newly configured wiki.
              p(v-if='!loading && !final.ok') #[svg.icons.is-18.is-text: use(xlink:href='#nc-square-remove')] Error: {{ final.error }}
            .panel-footer
              .progress-bar: div(v-bind:style='{width: currentProgress}')
              button.button.is-small.is-light-blue.is-outlined(v-on:click='proceedToAdmin', v-if='!loading && !final.ok', v-bind:disabled='loading') Back
              button.button.is-small.is-teal(v-on:click='proceedToFinal', v-if='!loading && !final.ok') Try Again
              button.button.is-small.is-green(v-on:click='finish', v-if='loading || final.ok', v-bind:disabled='loading') Start

    footer
      small Wiki.js Installation Wizard
      small(v-if='conf.telemetry') Telemetry Client ID: {{telemetryId}}
</template>

<script>

/* global siteConfig */

import axios from 'axios'
import _ from 'lodash'

export default {
  props: {
    telemetryId: {
      type: String,
      required: true
    },
    wikiVersion: {
      type: String,
      required: true
    },
    langs: {
      type: Array,
      required: true
    }
  },
  data() {
    return {
      loading: false,
      state: 'welcome',
      syscheck: {
        ok: false,
        error: '',
        results: []
      },
      dbcheck: {
        ok: false,
        error: ''
      },
      gitcheck: {
        ok: false,
        error: ''
      },
      final: {
        ok: false,
        error: '',
        redirectUrl: ''
      },
      conf: {
        adminEmail: '',
        adminPassword: '',
        adminPasswordConfirm: '',
        gitAuthPass: '',
        gitAuthSSHKey: '',
        gitAuthSSHKeyEnv: '',
        gitAuthSSHKeyDB: '',
        gitAuthSSL: true,
        gitAuthType: 'ssh',
        gitAuthUser: '',
        gitBranch: 'master',
        gitServerEmail: '',
        gitShowUserEmail: true,
        gitUrl: '',
        gitUseRemote: (siteConfig.git !== false),
        lang: siteConfig.lang || 'en',
        path: siteConfig.path || '/',
        pathRepo: './repo',
        port: siteConfig.port || 80,
        public: (siteConfig.public === true),
        selfRegister: (siteConfig.selfRegister === true),
        telemetry: true,
        title: siteConfig.title || 'Wiki',
        upgrade: false,
        upgMongo: 'mongodb://'
      },
      considerations: {
        https: false,
        port: false,
        localhost: false
      }
    }
  },
  computed: {
    currentProgress: function () {
      let perc = '0%'
      switch (this.state) {
        case 'welcome':
          perc = '0%'
          break
        case 'syscheck':
          perc = (this.syscheck.ok) ? '15%' : '5%'
          break
        case 'general':
          perc = '25%'
          break
        case 'considerations':
          perc = '30%'
          break
        case 'git':
          perc = '50%'
          break
        case 'gitcheck':
          perc = (this.gitcheck.ok) ? '70%' : '55%'
          break
        case 'admin':
          perc = '75%'
          break
        case 'upgrade':
          perc = '85%'
          break
      }
      return perc
    }
  },
  mounted: function () {
    /* if (appconfig.paths) {
      this.conf.pathData = appconfig.paths.data || './data'
      this.conf.pathRepo = appconfig.paths.repo || './repo'
    }
    if (appconfig.git !== false && _.isPlainObject(appconfig.git)) {
      this.conf.gitUrl = appconfig.git.url || ''
      this.conf.gitBranch = appconfig.git.branch || 'master'
      this.conf.gitShowUserEmail = (appconfig.git.showUserEmail !== false)
      this.conf.gitServerEmail = appconfig.git.serverEmail || ''
      if (_.isPlainObject(appconfig.git.auth)) {
        this.conf.gitAuthType = appconfig.git.auth.type || 'ssh'
        this.conf.gitAuthSSHKey = appconfig.git.auth.privateKey || ''
        this.conf.gitAuthUser = appconfig.git.auth.username || ''
        this.conf.gitAuthPass = appconfig.git.auth.password || ''
        this.conf.gitAuthSSL = (appconfig.git.auth.sslVerify !== false)
      }
    } */
  },
  methods: {
    proceedToWelcome: function (ev) {
      this.state = 'welcome'
      this.loading = false
    },
    proceedToSyscheck: function (ev) {
      let self = this
      this.state = 'syscheck'
      this.loading = true
      self.syscheck = {
        ok: false,
        error: '',
        results: []
      }

      _.delay(() => {
        axios.post('/syscheck', self.conf).then(resp => {
          if (resp.data.ok === true) {
            self.syscheck.ok = true
            self.syscheck.results = resp.data.results
          } else {
            self.syscheck.ok = false
            self.syscheck.error = resp.data.error
          }
          self.loading = false
          self.$nextTick()
        }).catch(err => {
          window.alert(err.message)
        })
      }, 1000)
    },
    proceedToGeneral: function (ev) {
      let self = this
      self.state = 'general'
      self.loading = false
      self.$nextTick(() => {
        self.$validator.validateAll('general')
      })
    },
    proceedToConsiderations: function (ev) {
      this.considerations = {
        https: false,
        port: false, // TODO
        localhost: false
      }
      this.state = 'considerations'
      this.loading = false
    },
    proceedToGit: function (ev) {
      let self = this
      self.state = 'git'
      self.loading = false
      self.$nextTick(() => {
        self.$validator.validateAll('git')
      })
    },
    proceedToGitCheck: function (ev) {
      let self = this
      this.state = 'gitcheck'
      this.loading = true
      self.gitcheck = {
        ok: false,
        results: [],
        error: ''
      }

      _.delay(() => {
        axios.post('/gitcheck', self.conf).then(resp => {
          if (resp.data.ok === true) {
            self.gitcheck.ok = true
            self.gitcheck.results = resp.data.results
          } else {
            self.gitcheck.ok = false
            self.gitcheck.error = resp.data.error
          }
          self.loading = false
          self.$nextTick()
        }).catch(err => {
          window.alert(err.message)
        })
      }, 1000)
    },
    proceedToAdmin: function (ev) {
      let self = this
      self.state = 'admin'
      self.loading = false
      self.$nextTick(() => {
        self.$validator.validateAll('admin')
      })
    },
    proceedToUpgrade: function (ev) {
      if (this.conf.upgrade) {
        this.state = 'upgrade'
        this.loading = false
        this.$nextTick(() => {
          this.$validator.validateAll('upgrade')
        })
      } else {
        this.proceedToFinal()
      }
    },
    proceedToFinal: function (ev) {
      let self = this
      self.state = 'final'
      self.loading = true
      self.final = {
        ok: false,
        error: '',
        redirectUrl: ''
      }

      _.delay(() => {
        axios.post('/finalize', self.conf).then(resp => {
          if (resp.data.ok === true) {
            self.$helpers._.delay(() => {
              self.final.ok = true
              switch (resp.data.redirectPort) {
                case 80:
                  self.final.redirectUrl = `http://${window.location.hostname}${resp.data.redirectPath}/login`
                  break
                case 443:
                  self.final.redirectUrl = `https://${window.location.hostname}${resp.data.redirectPath}/login`
                  break
                default:
                  self.final.redirectUrl = `http://${window.location.hostname}:${resp.data.redirectPort}${resp.data.redirectPath}/login`
                  break
              }
              self.loading = false
            }, 5000)
          } else {
            self.final.ok = false
            self.final.error = resp.data.error
            self.loading = false
          }
          self.$nextTick()
        }).catch(err => {
          window.alert(err.message)
        })
      }, 1000)
    },
    finish: function (ev) {
      window.location.assign(this.final.redirectUrl)
    }
  }
}

</script>