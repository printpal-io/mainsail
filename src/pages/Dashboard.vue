<template>
    <div>
        <v-row v-if="isMobile">
            <v-col>
                <status-panel></status-panel>
                <template v-for="component in mobileLayout">
                    <component
                        :is="extractPanelName(component.name)"
                        :key="'dashboard-mobileLayout-' + component.name"
                        :panel-id="extractPanelId(component.name)"></component>
                </template>
                <template>
                   <div>
                      <panel
                         :title="'AI'"
                         :collapsible="true"
                         card-class="ai-panel">
                         <v-row>
                            <v-col class="col-8">
                               <div class="px-2 d-flex flex-column justify-left align-left col">
                                  <div><strong>Status: </strong>{{ displayStatus }}</div>
                                  <div>Sensitivity: {{ displaySensitivity }}</div>
                                  <div>Notifications: {{ displayNotify }}</div>
                                  <div>Pausing: {{ displayAction }}</div>
                               </div>
                            </v-col>
                            <v-col class="col-4">
                               <v-col class="px-2 col-auto d-flex flex-column justify-center align-center">
                                  <v-progress-circular :rotate="-90" :size="55" :width="7" :value="scoreValue" :color="gaugeColorDisp">
                                     {{ scoreValue }}
                                  </v-progress-circular>
                                  <span class="mt-2">Defect Level</span>
                               </v-col>
                            </v-col>
                          </v-row>
                      </panel>
                   </div>
                   <v-dialog v-model="showDialogDisp" :max-width="400" @click:outside="closeDialog" @keydown.esc="closeDialog">
                       <v-card>
                           <v-card-title class="text-h5">AI Notification</v-card-title>
                           <v-card-text class="pb-0">
                               <p class="body-2">
                               Spaghetti defects were detected and the print has been paused. Clear the defects and click "Resume" to resume the print job.
                               </p>
                           </v-card-text>
                           <v-card-actions>
                               <v-spacer />
                               <v-btn text @click="closeDialog">CLOSE</v-btn>
                               <v-btn
                                   color="primary"
                                   text
                                   :disabled="!printerIsPrinting || !klipperReadyForGui"
                                   @click="resumePrint()">
                                   RESUME
                               </v-btn>
                           </v-card-actions>
                       </v-card>
                   </v-dialog>
                   <v-dialog v-model="showNotifyDialogDisp" :max-width="400" @click:outside="closeDialog" @keydown.esc="closeDialog">
                       <v-card>
                           <v-card-title class="text-h5">AI Notification</v-card-title>
                           <v-card-text class="pb-0">
                               <p class="body-2">
                               Spaghetti defects were detected. Press "Pause" to pause the print job.
                               </p>
                           </v-card-text>
                           <v-card-actions>
                               <v-spacer />
                               <v-btn text @click="closeDialog">CLOSE</v-btn>
                               <v-btn
                                   color="primary"
                                   text
                                   :disabled="!printerIsPrinting || !klipperReadyForGui"
                                   @click="pausePrint()">
                                   PAUSE
                               </v-btn>
                           </v-card-actions>
                       </v-card>
                   </v-dialog>
                </template>
            </v-col>
        </v-row>
        <v-row v-else-if="isTablet">
            <v-col class="col-6">
                <status-panel></status-panel>
                <template v-for="component in tabletLayout1">
                    <component
                        :is="extractPanelName(component.name)"
                        :key="'dashboard-tabletLayout1-' + component.name"
                        :panel-id="extractPanelId(component.name)"></component>
                </template>
            </v-col>
            <v-col class="col-6">
                <template v-for="component in tabletLayout2">
                    <component
                        :is="extractPanelName(component.name)"
                        :key="'dashboard-tabletLayout2-' + component.name"
                        :panel-id="extractPanelId(component.name)"></component>
                </template>
                <template>
                   <div>
                      <panel
                         :title="'AI'"
                         :collapsible="true"
                         card-class="ai-panel">
                         <v-row>
                            <v-col class="col-8">
                               <div class="px-2 d-flex flex-column justify-left align-left col">
                                  <div><strong>Status: </strong>{{ displayStatus }}</div>
                                  <div>Sensitivity: {{ displaySensitivity }}</div>
                                  <div>Notifications: {{ displayNotify }}</div>
                                  <div>Pausing: {{ displayAction }}</div>
                               </div>
                            </v-col>
                            <v-col class="col-4">
                               <v-col class="px-2 col-auto d-flex flex-column justify-center align-center">
                                  <v-progress-circular :rotate="-90" :size="55" :width="7" :value="scoreValue" :color="gaugeColorDisp">
                                     {{ scoreValue }}
                                  </v-progress-circular>
                                  <span class="mt-2">Defect Level</span>
                               </v-col>
                            </v-col>
                          </v-row>
                      </panel>
                   </div>
                   <v-dialog v-model="showDialogDisp" :max-width="400" @click:outside="closeDialog" @keydown.esc="closeDialog">
                       <v-card>
                           <v-card-title class="text-h5">AI Notification</v-card-title>
                           <v-card-text class="pb-0">
                               <p class="body-2">
                               Spaghetti defects were detected and the print has been paused. Clear the defects and click "Resume" to resume the print job.
                               </p>
                           </v-card-text>
                           <v-card-actions>
                               <v-spacer />
                               <v-btn text @click="closeDialog">CLOSE</v-btn>
                               <v-btn
                                   color="primary"
                                   text
                                   :disabled="!printerIsPrinting || !klipperReadyForGui"
                                   @click="resumePrint()">
                                   RESUME
                               </v-btn>
                           </v-card-actions>
                       </v-card>
                   </v-dialog>
                   <v-dialog v-model="showNotifyDialogDisp" :max-width="400" @click:outside="closeDialog" @keydown.esc="closeDialog">
                       <v-card>
                           <v-card-title class="text-h5">AI Notification</v-card-title>
                           <v-card-text class="pb-0">
                               <p class="body-2">
                               Spaghetti defects were detected. Press "Pause" to pause the print job.
                               </p>
                           </v-card-text>
                           <v-card-actions>
                               <v-spacer />
                               <v-btn text @click="closeDialog">CLOSE</v-btn>
                               <v-btn
                                   color="primary"
                                   text
                                   :disabled="!printerIsPrinting || !klipperReadyForGui"
                                   @click="pausePrint()">
                                   PAUSE
                               </v-btn>
                           </v-card-actions>
                       </v-card>
                   </v-dialog>
                </template>
            </v-col>
        </v-row>
        <v-row v-else-if="isDesktop">
            <v-col class="col-5">
                <status-panel></status-panel>
                <template v-for="component in desktopLayout1">
                    <component
                        :is="extractPanelName(component.name)"
                        :key="'dashboard-desktopLayout1-' + component.name"
                        :panel-id="extractPanelId(component.name)"></component>
                </template>
                <template>
                   <div>
                      <panel
                         :title="'AI'"
                         :collapsible="true"
                         card-class="ai-panel">
                         <v-row>
                            <v-col class="col-8">
                               <div class="px-2 d-flex flex-column justify-left align-left col">
                                  <div><strong>Status: </strong>{{ displayStatus }}</div>
                                  <div>Sensitivity: {{ displaySensitivity }}</div>
                                  <div>Notifications: {{ displayNotify }}</div>
                                  <div>Pausing: {{ displayAction }}</div>
                               </div>
                            </v-col>
                            <v-col class="col-4">
                               <v-col class="px-2 col-auto d-flex flex-column justify-center align-center">
                                  <v-progress-circular :rotate="-90" :size="55" :width="7" :value="scoreValue" :color="gaugeColorDisp">
                                     {{ scoreValue }}
                                  </v-progress-circular>
                                  <span class="mt-2">Defect Level</span>
                               </v-col>
                            </v-col>
                          </v-row>
                      </panel>
                   </div>
                   <v-dialog v-model="showDialogDisp" :max-width="400" @click:outside="closeDialog" @keydown.esc="closeDialog">
                       <v-card>
                           <v-card-title class="text-h5">AI Notification</v-card-title>
                           <v-card-text class="pb-0">
                               <p class="body-2">
                               Spaghetti defects were detected and the print has been paused. Clear the defects and click "Resume" to resume the print job.
                               </p>
                           </v-card-text>
                           <v-card-actions>
                               <v-spacer />
                               <v-btn text @click="closeDialog">CLOSE</v-btn>
                               <v-btn
                                   color="primary"
                                   text
                                   :disabled="!printerIsPrinting || !klipperReadyForGui"
                                   @click="resumePrint()">
                                   RESUME
                               </v-btn>
                           </v-card-actions>
                       </v-card>
                   </v-dialog>
                   <v-dialog v-model="showNotifyDialogDisp" :max-width="400" @click:outside="closeDialog" @keydown.esc="closeDialog">
                       <v-card>
                           <v-card-title class="text-h5">AI Notification</v-card-title>
                           <v-card-text class="pb-0">
                               <p class="body-2">
                               Spaghetti defects were detected. Press "Pause" to pause the print job.
                               </p>
                           </v-card-text>
                           <v-card-actions>
                               <v-spacer />
                               <v-btn text @click="closeDialog">CLOSE</v-btn>
                               <v-btn
                                   color="primary"
                                   text
                                   :disabled="!printerIsPrinting || !klipperReadyForGui"
                                   @click="pausePrint()">
                                   PAUSE
                               </v-btn>
                           </v-card-actions>
                       </v-card>
                   </v-dialog>
                </template>
            </v-col>
            <v-col class="col-7">
                <template v-for="component in desktopLayout2">
                    <component
                        :is="extractPanelName(component.name)"
                        :key="'dashboard-desktopLayout2-' + component.name"
                        :panel-id="extractPanelId(component.name)"></component>
                </template>
            </v-col>
        </v-row>
        <v-row v-else-if="isWidescreen">
            <v-col class="col-3">
                <status-panel></status-panel>
                <template v-for="component in widescreenLayout1">
                    <component
                        :is="extractPanelName(component.name)"
                        :key="'dashboard-desktopLayout1-' + component.name"
                        :panel-id="extractPanelId(component.name)"></component>
                </template>
            </v-col>
            <v-col class="col-5">
                <template v-for="component in widescreenLayout2">
                    <component
                        :is="extractPanelName(component.name)"
                        :key="'dashboard-desktopLayout2-' + component.name"
                        :panel-id="extractPanelId(component.name)"></component>
                </template>
            </v-col>
            <v-col class="col-4">
                <template v-for="component in widescreenLayout3">
                    <component
                        :is="extractPanelName(component.name)"
                        :key="'dashboard-desktopLayout3-' + component.name"
                        :panel-id="extractPanelId(component.name)"></component>
                </template>
                <template>
                   <div>
                      <panel
                         :title="'AI'"
                         :collapsible="true"
                         card-class="ai-panel">
                         <v-row>
                            <v-col class="col-8">
                               <div class="px-2 d-flex flex-column justify-left align-left col">
                                  <div><strong>Status: </strong>{{ displayStatus }}</div>
                                  <div>Sensitivity: {{ displaySensitivity }}</div>
                                  <div>Notifications: {{ displayNotify }}</div>
                                  <div>Pausing: {{ displayAction }}</div>
                               </div>
                            </v-col>
                            <v-col class="col-4">
                               <v-col class="px-2 col-auto d-flex flex-column justify-center align-center">
                                  <v-progress-circular :rotate="-90" :size="55" :width="7" :value="scoreValue" :color="gaugeColorDisp">
                                     {{ scoreValue }}
                                  </v-progress-circular>
                                  <span class="mt-2">Defect Level</span>
                               </v-col>
                            </v-col>
                          </v-row>
                      </panel>
                   </div>
                   <v-dialog v-model="showDialogDisp" :max-width="400" @click:outside="closeDialog" @keydown.esc="closeDialog">
                       <v-card>
                           <v-card-title class="text-h5">AI Notification</v-card-title>
                           <v-card-text class="pb-0">
                               <p class="body-2">
                               Spaghetti defects were detected and the print has been paused. Clear the defects and click "Resume" to resume the print job.
                               </p>
                           </v-card-text>
                           <v-card-actions>
                               <v-spacer />
                               <v-btn text @click="closeDialog">CLOSE</v-btn>
                               <v-btn
                                   color="primary"
                                   text
                                   :disabled="!printerIsPrinting || !klipperReadyForGui"
                                   @click="resumePrint()">
                                   RESUME
                               </v-btn>
                           </v-card-actions>
                       </v-card>
                   </v-dialog>
                   <v-dialog v-model="showNotifyDialogDisp" :max-width="400" @click:outside="closeDialog" @keydown.esc="closeDialog">
                       <v-card>
                           <v-card-title class="text-h5">AI Notification</v-card-title>
                           <v-card-text class="pb-0">
                               <p class="body-2">
                               Spaghetti defects were detected. Press "Pause" to pause the print job.
                               </p>
                           </v-card-text>
                           <v-card-actions>
                               <v-spacer />
                               <v-btn text @click="closeDialog">CLOSE</v-btn>
                               <v-btn
                                   color="primary"
                                   text
                                   :disabled="!printerIsPrinting || !klipperReadyForGui"
                                   @click="pausePrint()">
                                   PAUSE
                               </v-btn>
                           </v-card-actions>
                       </v-card>
                   </v-dialog>
                </template>
            </v-col>
        </v-row>
    </div>
</template>

<script lang="ts">
import Component from 'vue-class-component'
import { Mixins } from 'vue-property-decorator'
import ExtruderControlPanel from '@/components/panels/ExtruderControlPanel.vue'
import DashboardMixin from '@/components/mixins/dashboard'
import KlippyStatePanel from '@/components/panels/KlippyStatePanel.vue'
import MachineSettingsPanel from '@/components/panels/MachineSettings/MachineSettingsPanel.vue'
import MacrogroupPanel from '@/components/panels/MacrogroupPanel.vue'
import MacrosPanel from '@/components/panels/MacrosPanel.vue'
import MiniconsolePanel from '@/components/panels/MiniconsolePanel.vue'
import MinSettingsPanel from '@/components/panels/MinSettingsPanel.vue'
import MiscellaneousPanel from '@/components/panels/MiscellaneousPanel.vue'
import SpoolmanPanel from '@/components/panels/SpoolmanPanel.vue'
import StatusPanel from '@/components/panels/StatusPanel.vue'
import ToolheadControlPanel from '@/components/panels/ToolheadControlPanel.vue'
import TemperaturePanel from '@/components/panels/TemperaturePanel.vue'
import WebcamPanel from '@/components/panels/WebcamPanel.vue'

@Component({
    components: {
        ExtruderControlPanel,
        KlippyStatePanel,
        MachineSettingsPanel,
        MacrogroupPanel,
        MacrosPanel,
        MiniconsolePanel,
        MinSettingsPanel,
        MiscellaneousPanel,
        SpoolmanPanel,
        StatusPanel,
        ToolheadControlPanel,
        TemperaturePanel,
        WebcamPanel,
    },
})
export default class PageDashboard extends Mixins(DashboardMixin) {
    get mobileLayout() {
        return this.$store.getters['gui/getPanels']('mobile', 0, true)
    }

    get tabletLayout1() {
        return this.$store.getters['gui/getPanels']('tablet', 1, true)
    }

    get tabletLayout2() {
        return this.$store.getters['gui/getPanels']('tablet', 2, true)
    }

    get desktopLayout1() {
        return this.$store.getters['gui/getPanels']('desktop', 1, true)
    }

    get desktopLayout2() {
        return this.$store.getters['gui/getPanels']('desktop', 2, true)
    }

    get widescreenLayout1() {
        return this.$store.getters['gui/getPanels']('widescreen', 1, true)
    }

    get widescreenLayout2() {
        return this.$store.getters['gui/getPanels']('widescreen', 2, true)
    }

    get widescreenLayout3() {
        return this.$store.getters['gui/getPanels']('widescreen', 3, true)
    }

    extractPanelName(name: string) {
        return name.split('_')[0] + '-panel'
    }

    extractPanelId(name: string) {
        return name.split('_')[1] ?? null
    }

    enableAI = false;
    notifyThresholdVal : number = 30;
    actionThresholdVal : number = 60;
    bufferLength : number = 16;
    status = false;
    enableNotify = false;
    enableAction = false;
    score : number = 0;
    showDialog = false;
    showNotifyDialog = false;

    timer = null;
    intervalId = null;
    gaugeColor = 'primary';
    errorMsg = '';

    get showDialogDisp() {
      return this.showDialog
    }

    get showNotifyDialogDisp() {
      return this.showNotifyDialog
    }

    get scoreValue() {
        return this.score
    }

    set scoreValue(newVal) {
        this.score = newVal;
    }

    get emailAddress() {
        return this.email
    }

    set emailAddress(newVal) {
        this.email = newVal;
    }

    get notifyThreshold() {
        return this.notifyThresholdVal
    }

    get actionThreshold() {
        return this.actionThresholdVal
    }

    get enableMonitoring() {
        return this.enableAI
    }

    get enableNotification() {
        return this.enableNotify
    }

    get enableActionPause() {
        return this.enableAction
    }

    set enableMonitoring(newVal) {
        this.enableAI = newVal;
    }

    set enableNotification(newVal) {
        this.enableNotify  = newVal;
    }

    set enableActionPause(newVal) {
        this.enableAction  = newVal;
    }

    set notifyThreshold(newVal) {
        this.notifyThresholdVal = newVal;
    }

    set actionThreshold(newVal) {
        this.actionThresholdVal = newVal;
    }

    get gaugeColorDisp() {
        return this.gaugeColor;
    }

    get errorMsgDispl() {
      return this.errorMsg;
    }

    get displayStatus() {
        if (!this.enableAI) {
          return 'Disabled';
        } else if (this.enableAI && this.status) {
          if (this.errorMsg != '') {
            return this.errorMsg;
          }
          return 'Monitoring';
        } else if (this.enableAI && !this.status) {
          return 'Idle';
        } else {
          return 'Disabled';
        }
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

    get displayNotify() {
      if (this.enableNotify) {
        return 'Enabled';
      } else {
        return 'Disabled';
      }
    }

    get displayAction() {
      if (this.enableAction) {
        return 'Enabled';
      } else {
        return 'Disabled';
      }
    }

    setGaugeColor() {
			if(this.defectLevel < 20) {
				this.gaugeColor = "success";
			} else if (this.defectLevel >= 20 && this.defectLevel < 40.0) {
				this.gaugeColor = "amber";
			} else if (this.defectLevel >= 40 && this.defectLevel < 60.0) {
				this.gaugeColor = "warning";
			} else {
				this.gaugeColor = "error";
			}
		}

    resetNotify() {
      const timeout = 5000;
      const controller = new AbortController();
      const id = setTimeout(() => controller.abort(), timeout);
      fetch('http://' + this.hostname() + ':8989/machine/printwatch/reset_notify', {signal : controller.signal})
			.then((response) => response.json())
			.then((data) => {
        console.log(data);
				clearTimeout(id);
			})
			.catch(error => {
				clearTimeout(id);
			});
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
			})
			.catch(error => {
				clearTimeout(id);
			});
    }

    fetchScores() {
      const timeout = 5000;
      const controller = new AbortController();
      const id = setTimeout(() => controller.abort(), timeout);
      fetch('http://' + this.hostname() + ':8989/machine/printwatch/monitor', {signal : controller.signal})
			.then((response) => response.json())
			.then((data) => {
				clearTimeout(id);
        if (data.status == 8000) {
          this.enableNotify = data.items.status.settings.actions.notify;
          this.enableAction = data.items.status.settings.actions.pause;
          this.bufferLength = data.items.status.settings.buffer_length;
          this.enableAI = data.items.status.settings.monitoring_on;
          this.status = data.items.status.active;
          if (data.items.status.active) {
            this.score = parseInt(parseFloat(data.items.status.ema) * 100);
            this.errorMsg = data.items.status.message;
          } else {
            this.score = 0;
          }
          this.setGaugeColor();
          if (data.items.status.paused_by) {
            this.showDialog = true;
          } else {
            this.showDialog = false;
          }
          if (data.items.status.notify && !this.showDialog) {
            this.showNotifyDialog = true;
          } else {
            this.showNotifyDialog = false;
          }
				} else {
          this.enableAI = false;
          this.status = false;
          this.score = 0;
          this.setGaugeColor();
        }
			})
			.catch(error => {
				clearTimeout(id);
				console.log('There was a problem with the fetch operation:', error);
			});
    }

    writeSettings() {
      const timeout = 5000;
      const controller = new AbortController();
      const id = setTimeout(() => controller.abort(), timeout);

      var ss = {
        email_addr : this.email,
        notification_threshold : this.notifyThresholdVal,
        action_threshold : this.actionThresholdVal,
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
			.then((response) => response.json())
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
					console.log('There was a problem with the fetch operation:', error);
				});
    }

    checkAndStartInterval() {
      this.fetchScores(); // Initial API call

			if (this.intervalId != null) {
				clearInterval(this.intervalId);
			}
			this.intervalId = setInterval(() => {
				this.fetchScores();
			}, 5000); // Every 5 seconds

    }

    stopInterval() {
      clearInterval(this.intervalId);
    }


    hostname() {
        return this.$store.state.socket.hostname
    }

    mounted() {
      console.log('attempting mount');
      this.fetchSettings();
      this.checkAndStartInterval();
      this.setGaugeColor();
    }

    beforeDestroy() {
      this.stopInterval();
    }

    resumePrint() {
        this.closeDialog()
        this.$socket.emit('printer.print.resume', {}, { action: 'switchToDashboard' })
    }

    pausePrint() {
        this.closeDialog()
        this.$socket.emit('printer.print.pause', {}, { action: 'switchToDashboard' })
    }

    cancelPrint() {
        this.closeDialog()
        this.$socket.emit('printer.print.cancel', {}, { action: 'switchToDashboard' })
    }

    closeDialog() {
        this.showDialog = false;
        this.showNotifyDialog = false;
        this.$emit('closeDialog')
        this.resetNotify();
    }
}
</script>
