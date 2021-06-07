<template>
  <div id="headless-recorder" class="recorder">
    <div class="header">
      <a href="#" @click="goHome">
        DPC recorder <span class="text-muted"><small>{{version}}</small></span>

      </a>
      <div class="left">
        <div class="recording-badge" v-show="isRecording">
          <span class="red-dot"></span>
          {{recordingBadgeText}}
        </div>
        <a href="#" @click="toggleShowHelp" class="header-button">
          <img src="/images/help.svg" alt="help" width="18px">
        </a>
        <a href="#" @click="openOptions" class="header-button">
          <img src="/images/settings.svg" alt="settings" width="18px">
        </a>
      </div>
    </div>
    <div class="main">
      <div class="tabs" v-show="!showHelp">
        <RecordingTab :code="code" :is-recording="isRecording" :live-events="liveEvents" v-show="!showResultsTab"/>

        <div class="recording-footer" v-show="!showResultsTab">
          <button class="btn btn-sm" @click="toggleRecord" :class="isRecording ? 'btn-danger' : 'btn-primary'">
            {{recordButtonText}}
          </button>
          <button class="btn btn-sm btn-primary btn-outline-primary" @click="togglePause" v-show="isRecording">
            {{pauseButtonText}}
          </button>
          <a href="#" @click="showResultsTab = true" v-show="code">view code</a>
        </div>
        <ResultsTab :puppeteer="recordingInJson" :playwright="recordingInJson" :options="options" v-if="showResultsTab" />

        <div class="results-footer" v-if="showResultsTab">
          <button class="btn btn-sm btn-primary" @click="restart">Restart</button>
          <button class="btn btn-sm btn-primary" @click="sendToServer">Send to server</button>

          <a href="#" v-clipboard:copy="getCodeForCopy()" @click.prevent="setCopying">{{copyLinkText}}</a>
        </div>
      </div>
      <div class="results-footer">
          <label>
            Region of intereset (ROI):
            <input type="checkbox" v-model="isROI" @change="toogleROI" />
          </label>
        </div>
    </div>
  </div>
</template>

<script>
  import { version } from '../../../package.json'
  import PuppeteerCodeGenerator from '../../code-generator/PuppeteerCodeGenerator'
  import PlaywrightCodeGenerator from '../../code-generator/PlaywrightCodeGenerator'
  import RecordingTab from './RecordingTab.vue'
  import ResultsTab from './ResultsTab.vue'
  import HelpTab from './HelpTab.vue'
  import ChecklyBadge from './ChecklyBadge.vue'

  import actions from '../../models/extension-ui-actions'

export default {
    name: 'App',
    components: { ResultsTab, RecordingTab, HelpTab },
    data () {
      return {
        recordingInJson: '',
        code: '',
        codeForPlaywright: '',
        options: {},
        showResultsTab: false,
        showHelp: false,
        liveEvents: [],
        recording: [],
        isRecording: false,
        isPaused: false,
        isCopying: false,
        isROI: false,
        bus: null,
        version,
        currentResultTab: null
      }
    },
    mounted () {
      this.loadState(() => {
        if (this.isRecording) {
          console.debug('opened in recording state, fetching recording events')
          this.$chrome.storage.local.get(['recording', 'options', 'isROI'], ({ recording, isROI }) => {
            console.debug('loaded recording', recording)
            this.liveEvents = recording
            this.isROI=isROI
          })
        }

        if (!this.isRecording && this.code) {
          this.showResultsTab = true
        }
      })
      this.bus = this.$chrome.extension.connect({ name: 'recordControls' })
    },
    methods: {
      toggleRecord () {
        if (this.isRecording) {
          this.stop()
        } else {
          this.start()
        }
        this.isRecording = !this.isRecording
        this.storeState()
      },
      togglePause () {
        if (this.isPaused) {
          this.bus.postMessage({ action: actions.UN_PAUSE })
          this.isPaused = false
        } else {
          this.bus.postMessage({ action: actions.PAUSE })
          this.isPaused = true
        }
        this.storeState()
      },
      start () {
        this.cleanUp()
        console.debug('start recorder')
        this.bus.postMessage({ action: actions.START })
      },
      stop () {
        console.debug('stop recorder')
        this.bus.postMessage({ action: actions.STOP })

        this.$chrome.storage.local.get(['dpcRecording', 'recording', 'options'], ({dpcRecording, recording, options }) => {
          console.debug('loaded recording', recording)
          console.debug('loaded options', options)

          this.recording = JSON.stringify(dpcRecording)
          //const codeOptions = options ? options.code : {}

          this.recordingInJson=JSON.stringify(dpcRecording)
          //const codeGen = new PuppeteerCodeGenerator(codeOptions)
          //const codeGenPlaywright = new PlaywrightCodeGenerator(codeOptions)
          this.code = this.dpcRecording //codeGen.generate(this.recording)
          this.codeForPlaywright = this.dpcRecording //codeGenPlaywright.generate(this.recording)
          this.showResultsTab = true
          //this.storeState()
        })
      },
      restart () {
        console.log('restart')
        this.cleanUp()
        this.bus.postMessage({ action: actions.CLEAN_UP })
      },
      sendToServer () {
        alert('sending to server...')
      },
      cleanUp () {
        this.recording = this.liveEvents = []
        this.code = ''
        this.codeForPlaywright = ''
        this.showResultsTab = this.isRecording = this.isPaused = false
        this.storeState()
      },
      openOptions () {
        if (this.$chrome.runtime.openOptionsPage) {
          this.$chrome.runtime.openOptionsPage()
        }
      },
      loadState (cb) {
        this.$chrome.storage.local.get(['controls', 'code', 'options', 'codeForPlaywright'], ({ controls, code, options, codeForPlaywright }) => {
          if (controls) {
            this.isRecording = controls.isRecording
            this.isPaused = controls.isPaused
          }

          if (code) {
            this.code = code
          }

          if (codeForPlaywright) {
            this.codeForPlaywright = codeForPlaywright
          }

          if (options) {
            this.options = options
          }
          cb()
        })
      },
      storeState () {
        this.$chrome.storage.local.set({
          code: this.code,
          codeForPlaywright: this.codeForPlaywright,
          controls: {
            isRecording: this.isRecording,
            isPaused: this.isPaused
          }
        })
      },
      setCopying () {
        this.isCopying = true
        setTimeout(() => { this.isCopying = false }, 1500)
      },
      goHome () {
        this.showResultsTab = false
        this.showHelp = false
      },
      toggleShowHelp () {
        this.showHelp = !this.showHelp
      },
      getCodeForCopy () {
        return this.recordingInJson
      },
      toogleROI(){
        chrome.runtime.sendMessage({
          action: 'toogleROI',
          value: this.isROI
        })
      this.$chrome.storage.local.set({
          isROI: this.isROI
        })
      }
    },
    computed: {
      recordingBadgeText () {
        return this.isPaused ? 'paused' : 'recording'
      },
      recordButtonText () {
        return this.isRecording ? 'Stop' : 'Record'
      },
      pauseButtonText () {
        return this.isPaused ? 'Resume' : 'Pause'
      },
      copyLinkText () {
        return this.isCopying ? 'copied!' : 'copy to clipboard'
      }
    }
}
</script>

<style lang="scss" scoped>
  @import "~styles/_animations.scss";
  @import "~styles/_variables.scss";
  @import "~styles/_mixins.scss";
  .recorder {
    font-size: 14px;

    .header {
      @include header();

      a {
        color: $gray-dark;
      }

      .left {
        margin-left: auto;
        display: flex;
        justify-content: flex-start;
        align-items: center;

        .recording-badge {
          color: $brand-danger;
          .red-dot {
            height: 9px;
            width: 9px;
            background-color: $brand-danger;
            border-radius: 50%;
            display: inline-block;
            margin-right: .4rem;
            vertical-align: middle;
            position: relative;
          }
        }

        .header-button {
          margin-left: $spacer;
          img {
            vertical-align: middle;
          }
        }
      }
    }

    .recording-footer {
      @include footer();
      img {
        margin-left: 8px;
        width: 80px;
        vertical-align: middle;
      }
    }
    .results-footer {
      @include footer()
    }
  }
</style>
