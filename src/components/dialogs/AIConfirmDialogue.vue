<template>
    <v-dialog v-model="bool" :max-width="400" @click:outside="closeDialog" @keydown.esc="closeDialog">
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
                <v-btn
                    color="danger"
                    text
                    :disabled="!printerIsPrinting || !klipperReadyForGui"
                    @click="cancelPrint()">
                    CANCEL
                </v-btn>
            </v-card-actions>
        </v-card>
    </v-dialog>
</template>

<script lang="ts">
import { Component, Mixins, Prop } from 'vue-property-decorator'
import BaseMixin from '@/components/mixins/base'
import { FileStateGcodefile } from '@/store/files/types'
import SettingsRow from '@/components/settings/SettingsRow.vue'
import { mdiPrinter3d } from '@mdi/js'
import { defaultBigThumbnailBackground } from '@/store/variables'

@Component({
    components: {
        SettingsRow,
    },
})
export default class AIConfirmtDialog extends Mixins(BaseMixin) {
    mdiPrinter3d = mdiPrinter3d

    @Prop({ required: true, default: false })
    declare readonly bool: boolean

    @Prop({ required: true, default: '' })
    declare readonly currentPath: string

    @Prop({ required: true })
    declare file: FileStateGcodefile

    get timelapseEnabled() {
        return this.$store.state.server.timelapse?.settings?.enabled ?? false
    }

    set timelapseEnabled(newVal) {
        this.$socket.emit(
            'machine.timelapse.post_settings',
            { enabled: newVal },
            { action: 'server/timelapse/initSettings' }
        )
    }

    get bigThumbnailBackground() {
        return this.$store.state.gui.uiSettings.bigThumbnailBackground ?? defaultBigThumbnailBackground
    }

    get bigThumbnailStyle() {
        if (defaultBigThumbnailBackground.toLowerCase() === this.bigThumbnailBackground.toLowerCase()) {
            return {}
        }

        return { backgroundColor: this.bigThumbnailBackground }
    }

    get maxThumbnailWidth() {
        return this.file?.big_thumbnail_width ?? 400
    }

    resumePrint() {
        this.closeDialog()
        this.$socket.emit('printer.print.resume', {}, { action: 'switchToDashboard' })
    }

    cancelPrint() {
        this.closeDialog()
        this.$socket.emit('printer.print.cancel', {}, { action: 'switchToDashboard' })
    }

    closeDialog() {
        this.$emit('closeDialog')
    }
}
</script>
