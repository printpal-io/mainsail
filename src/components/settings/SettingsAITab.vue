<style scoped></style>

<template>
    <div>
        <v-card v-if="!form.bool" flat>
            <v-card-text>
                <div class="d-flex align-center">
                    <v-icon style="opacity: 0.7">{{ mdiEyeOutline }}</v-icon>
                    <v-card-title class="mx-n2">
                        AI
                    </v-card-title>
                    <v-divider class="ml-3"></v-divider>
                </div>
                <settings-row title="Enable Monitoring" sub-title="Turn on or off the visual failure monitoring">
                    <v-switch v-model="displayEnableAI" hide-details></v-switch>
                </settings-row>
                <v-divider class="my-2"></v-divider>
                <settings-row
                    title="Sensitivity"
                    sub-title="Change how fast the AI takes an action or sends a notification.">
                    <v-select
                        v-model="displaySensitivity"
                        :items="['Ultra-Fast (1 minutes)', 'Fast (3 minutes)', 'Medium (6 minutes)', 'Long (12 minutes)']"
                        class="mt-0"
                        hide-details
                        outlined
                        dense />
                </settings-row>
                <v-divider class="my-2"></v-divider>
                <settings-row title="Enable Notifications" sub-title="Send a notification to the UI once the Notification Threshold has been passed.">
                    <v-switch v-model="displayNotify" hide-details></v-switch>
                </settings-row>
                <v-divider class="my-2"></v-divider>
                <settings-row title="Notification Threshold" sub-title="The Defect score at which a notification will be sent.">
                    <v-slider
									    v-model="displayNotificationThresh"
									    :max="100"
									    :min="0"
									    hide-details
											thumb-label="always"
											class="pt-8"
									  >
									  </v-slider>
                </settings-row>
                <v-divider class="my-2"></v-divider>
                <settings-row title="Enable Print Pause" sub-title="Pause the print when the Action Threshold has been passed.">
                    <v-switch v-model="displayAction" hide-details></v-switch>
                </settings-row>
                <v-divider class="my-2"></v-divider>
                <settings-row title="Action Threshold" sub-title="The Defect score at which an action (print pause) will be taken.">
                    <v-slider
                      v-model="displayActionThresh"
                      :max="100"
                      :min="0"
                      hide-details
                      thumb-label="always"
                      class="pt-8"
                    >
                    </v-slider>
                </settings-row>
                <v-divider class="my-2"></v-divider>
                <settings-row title="Save Settings">
                    <v-btn v-on:click="writeSettings" medium color="primary">Save</v-btn>
                </settings-row>
            </v-card-text>
        </v-card>
    </div>
</template>

<script lang="ts">
import { Component, Mixins, Watch } from 'vue-property-decorator'
import BaseMixin from '../mixins/base'
import SettingsRow from '@/components/settings/SettingsRow.vue'
import { Debounce } from 'vue-debounce-decorator'
import { mdiFilter, mdiPencil, mdiFilterOff, mdiDelete, mdiConsoleLine, mdiEyeOutline } from '@mdi/js'

interface consoleForm {
    bool: boolean
    id: string | null
    valid: boolean
    name: string
    regex: string
}

@Component({
    components: { SettingsRow },
})
export default class SettingsConsoleTab extends Mixins(BaseMixin) {
    mdiFilter = mdiFilter
    mdiFilterOff = mdiFilterOff
    mdiPencil = mdiPencil
    mdiDelete = mdiDelete
    mdiConsoleLine = mdiConsoleLine
    mdiEyeOutline = mdiEyeOutline

    enableAI = false;
    notifyThresholdVal : number = 30;
    actionThresholdVal : number = 60;
    bufferLength : number = 16;
    enableNotify = false;
    enableAction = false;
    emailAddress = '';


    private form: consoleForm = {
        bool: false,
        valid: false,
        name: '',
        regex: '',
        id: null,
    }

    private rules = {
        required: (value: string) => value !== '' || 'required',
        unique: (value: string) => !this.existsPresetName(value) || 'Name already exists',
    }

    private consoleHeightTmp = 300

    hostname() {
        return this.$store.state.socket.hostname
    }

    mounted() {
        this.consoleHeightTmp = this.consoleHeight
        this.fetchSettings();
    }

    get consoleFilters() {
        return this.$store.getters['gui/console/getConsolefilters'] ?? []
    }

    get availableDirections() {
        return [
            {
                text: this.$t('Settings.ConsoleTab.DirectionTable'),
                value: 'table',
            },
            {
                text: this.$t('Settings.ConsoleTab.DirectionShell'),
                value: 'shell',
            },
        ]
    }

    get consoleDirection() {
        return this.$store.state.gui.console.direction ?? 'table'
    }

    set consoleDirection(newVal) {
        this.$store.dispatch('gui/console/saveSetting', { name: 'direction', value: newVal })
    }

    get availableEntryStyles() {
        return [
            {
                text: this.$t('Settings.ConsoleTab.EntryStyleDefault'),
                value: 'default',
            },
            {
                text: this.$t('Settings.ConsoleTab.EntryStyleCompact'),
                value: 'compact',
            },
        ]
    }

    get entryStyle() {
        return this.$store.state.gui.console.entryStyle ?? 'default'
    }

    set entryStyle(newVal) {
        this.$store.dispatch('gui/console/saveSetting', { name: 'entryStyle', value: newVal })
    }

    get consoleHeight() {
        return this.$store.state.gui.console.height ?? 300
    }

    set consoleHeight(newVal) {
        this.$store.dispatch('gui/console/saveSetting', { name: 'height', value: newVal })
    }

    @Watch('consoleHeight')
    consoleHeightChanged(newVal: number) {
        this.consoleHeightTmp = newVal
    }

    @Debounce(500)
    updateConsoleHeight(newVal: number) {
        this.consoleHeight = newVal
    }

    get hideWaitTemperatures() {
        return this.$store.state.gui.console.hideWaitTemperatures
    }

    set hideWaitTemperatures(newVal) {
        this.$store.dispatch('gui/console/saveSetting', { name: 'hideWaitTemperatures', value: newVal })
    }

    get hideTimelapse() {
        return this.$store.state.gui.console.hideTlCommands
    }

    set hideTimelapse(newVal) {
        this.$store.dispatch('gui/console/saveSetting', { name: 'hideTlCommands', value: newVal })
    }

    get displayEnableAI() {
      return this.enableAI;
    }

    set displayEnableAI(newVal) {
      this.enableAI = newVal;
      if (newVal) {
        this.initMonitor();
      } else {
        this.stopMonitor();
      }
    }

    get displayEmailAddr() {
      return this.emailAddress;
    }

    set displayEmailAddr(newVal) {
      this.emailAddress = newVal;
    }

    get displaySensitivity() {
      if (this.bufferLength <= 8) {
        return 'Ultra-Fast (1 minutes)';
      } else if (this.bufferLength > 8 && this.bufferLength <= 16) {
        return 'Fast (3 minutes)';
      } else if (this.bufferLength > 16 && this.bufferLength <=32) {
        return 'Medium (6 minutes)';
      } else {
        return 'Long (12 minutes)';
      }
    }

    set displaySensitivity(newVal) {
      if (newVal == 'Ultra-Fast (1 minutes)') {
        this.bufferLength = 8;
      } else if (newVal == 'Fast (3 minutes)') {
        this.bufferLength = 16;
      } else if (newVal == 'Medium (6 minutes)') {
        this.bufferLength = 32;
      } else {
        this.bufferLength = 64;
      }
    }

    get displayNotify() {
      return this.enableNotify;
    }

    get displayAction() {
      return this.enableAction;
    }

    set displayNotify(newVal) {
      this.enableNotify = newVal;
    }

    set displayAction(newVal) {
      this.enableAction = newVal;
    }

    get displayActionThresh() {
      return this.actionThresholdVal;
    }

    set displayActionThresh(newVal) {
      this.actionThresholdVal = newVal;
    }

    get displayNotificationThresh() {
      return this.notifyThresholdVal
    }

    set displayNotificationThresh(newVal) {
      this.notifyThresholdVal = newVal;
    }

    existsPresetName(name: string) {
        return this.consoleFilters.findIndex((filter: any) => filter.name === name && filter.id !== this.form.id) >= 0
    }

    clearForm() {
        this.form.bool = false
        this.form.id = null
        this.form.name = ''
        this.form.regex = ''
    }

    toggleFilter(filter: any) {
        const values = {
            name: filter.name,
            bool: !filter.bool,
            regex: filter.regex,
        }

        this.$store.dispatch('gui/console/filterUpdate', { id: filter.id, values })
    }

    createFilter() {
        this.clearForm()
        this.form.bool = true
    }

    editFilter(filter: any) {
        this.form.name = filter.name
        this.form.id = filter.id
        this.form.regex = filter.regex

        this.form.bool = true
    }

    saveFilter() {
        if (this.form.valid) {
            const filter = {
                name: this.form.name,
                bool: this.form.bool,
                regex: this.form.regex,
            }

            if (this.form.id) this.$store.dispatch('gui/console/filterUpdate', { id: this.form.id, values: filter })
            else this.$store.dispatch('gui/console/filterStore', { values: filter })

            this.clearForm()
        }
    }

    deleteFilter(id: string) {
        this.$store.dispatch('gui/console/filterDelete', id)
    }

    fetchSettings() {
      const timeout = 5000;
      const controller = new AbortController();
      const id = setTimeout(() => controller.abort(), timeout);
      fetch('http://' + this.hostname() + ':8989/machine/printwatch/get_settings', {signal : controller.signal})
			.then((response) => response.json())
			.then((data) => {
        console.log(data);
				clearTimeout(id);
        this.notifyThresholdVal = Math.round(data.settings.thresholds.notification * 100.0);
        this.actionThresholdVal = Math.round(data.settings.thresholds.action * 100.0);
        this.enableAI = data.settings.monitoring_on;
        this.enableAction = data.settings.actions.pause;
        this.enableNotify = data.settings.actions.notify;
        this.bufferLength = data.settings.buffer_length;
        this.emailAddress = data.settings.email_addr;
			})
			.catch(error => {
				clearTimeout(id);
			});
    }

    writeSettings() {
      const timeout = 5000;
      const controller = new AbortController();
      const id = setTimeout(() => controller.abort(), timeout);

      var ss = {
        email_addr : this.emailAddress,
        notification_threshold : this.notifyThresholdVal,
        action_threshold : this.actionThresholdVal,
        buffer_length : this.bufferLength,
        notify_action : this.enableNotify,
        pause_action : this.enableAction
      };

      fetch('http://' + this.hostname() + ':8989/machine/printwatch/set_settings', {
        method : 'POST',
        headers: {
          'Accept': 'application/json',
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(ss),
        signal : controller.signal
      })
			.then((response) => response.text())
			.then((data) => {
				clearTimeout(id);
			})
			.catch(error => {
				clearTimeout(id);
				console.log('There was a problem with the fetch operation:', error);
			});
    }

    initMonitor() {
        // API call to backend
        const timeout = 5000;
        const controller = new AbortController();
        const id = setTimeout(() => controller.abort(), timeout);

        fetch('http://' + this.hostname() + ':8989/machine/printwatch/monitor_init', {signal : controller.signal})
        .then((response) => response.json())
        .then(() => {
          clearTimeout(id);
        })
        .catch(error => {
          clearTimeout(id);
        });
    }

    stopMonitor() {
        // API call to backend
        const timeout = 5000;
        const controller = new AbortController();
        const id = setTimeout(() => controller.abort(), timeout);

        fetch('http://' + this.hostname() + ':8989/machine/printwatch/monitor_off', {signal : controller.signal})
        .then((response) => response.json())
        .then(() => {
          clearTimeout(id);
        })
        .catch(error => {
          clearTimeout(id);
        });
    }
}
</script>
