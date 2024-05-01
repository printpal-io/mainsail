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
</template>

<script lang="ts">
import { Component, Mixins } from 'vue-property-decorator'
import BaseMixin from '@/components/mixins/base'


export default class PrintWatchPanel extends Mixins(BaseMixin) {
    enableAI = false;
    notifyThresholdVal : number = 30;
    actionThresholdVal : number = 60;
    bufferLength : number = 16;
    status = false;
    enableNotify = false;
    enableAction = false;
    score : number = 0;

    timer = null;
    intervalId = null;
    gaugeColor = 'primary';
    errorMsg = '';

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
}
</script>

<style scoped></style>
