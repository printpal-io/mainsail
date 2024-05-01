<style scoped></style>

<template>
    <panel
        v-if="showAIPanel"
        :title="'AI'"
        :collapsible="true"
        card-class="ai-panel">
    </panel>
</template>

<script lang="ts">
import { Component, Mixins } from 'vue-property-decorator'
import BaseMixin from '@/components/mixins/base'
import Panel from '@/components/ui/Panel.vue'
@Component({
    components: { Panel },
})
export default class AIPanel extends Mixins(BaseMixin) {


    get showAIPanel() {
        return (
            this.klipperReadyForGui
        )
    }
}
</script>
