<template>
  <div :class="['plan-container overflow-auto flex-grow-1 bg-light', viewOptions.viewMode]" v-dragscroll>
    <div class="menu p-2 bg-white border rounded-right border-left-0" :class="{'menu-hidden': menuHidden}">
      <header>
        <button v-on:click="menuHidden = !menuHidden" class="btn">
          <i class="fa fa-cogs p-0"></i>
        </button>
        <strong v-if="!menuHidden">
          Display Options
        </strong>
      </header>
      <div v-if="!menuHidden">
        <div class="form-group">
          <div>
            View
          </div>
          <div class="btn-group btn-group-sm">
            <button class="btn btn-outline-secondary" :class="{'active': viewOptions.viewMode == viewModes.FULL}" v-on:click="viewOptions.viewMode = viewModes.FULL">full</button>
            <button class="btn btn-outline-secondary" :class="{'active': viewOptions.viewMode == viewModes.COMPACT}" v-on:click="viewOptions.viewMode = viewModes.COMPACT">compact</button>
            <button class="btn btn-outline-secondary" :class="{'active': viewOptions.viewMode == viewModes.DOT}" v-on:click="viewOptions.viewMode = viewModes.DOT">dot</button>
          </div>
        </div>
        <div class="form-group">
          <div>Graph metric</div>
          <div class="btn-group btn-group-sm">
            <button class="btn btn-outline-secondary" :class="{'active': viewOptions.highlightType === highlightTypes.NONE}" v-on:click="viewOptions.highlightType = highlightTypes.NONE">none</button>
            <button class="btn btn-outline-secondary" :class="{'active': viewOptions.highlightType === highlightTypes.DURATION}" v-on:click="viewOptions.highlightType = highlightTypes.DURATION" :disabled="!node[planService.ACTUAL_DURATION_PROP]">duration</button>
            <button class="btn btn-outline-secondary" :class="{'active': viewOptions.highlightType === highlightTypes.ROWS}" v-on:click="viewOptions.highlightType = highlightTypes.ROWS" :disabled="!node[planService.ACTUAL_ROWS_PROP]">rows</button>
            <button class="btn btn-outline-secondary" :class="{'active': viewOptions.highlightType === highlightTypes.COST}" v-on:click="viewOptions.highlightType = highlightTypes.COST">cost</button>
          </div>
        </div>
        <div class="form-check">
          <input id="showPlannerEstimate" type="checkbox" v-model="viewOptions.showPlannerEstimate" class="form-check-input" :disabled="!node[planService.ACTUAL_ROWS_PROP]">
          <label for="showPlannerEstimate" class="form-check-label">Estimations</label>
        </div>
        <div class="form-check">
          <input id="showTags" type="checkbox" v-model="viewOptions.showTags" class="form-check-input">
          <label class="clickable" for="showTags"> Tags</label>
        </div>
      </div>
    </div>

    <div class="plan grab-bing h-100 w-100">
      <ul>
        <li>
          <plan-node :node="node" :plan="plan" :viewOptions="viewOptions"/>
        </li>
      </ul>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue, Watch } from 'vue-property-decorator';
import PlanNode from '@/components/PlanNode.vue';
import { PlanService } from '@/services/plan-service';
import { HighlightType, ViewMode } from '../enums';

import VueDragscroll from 'vue-dragscroll';
Vue.use(VueDragscroll);

import { dragscroll } from 'vue-dragscroll';

@Component({
  name: 'plan',
  components: {
    PlanNode,
  },
  directives: {
    dragscroll,
  },
})
export default class Plan extends Vue {
  @Prop([Object, Array]) private planJson!: any | any[];
  @Prop(String) private planQuery!: string;
  private plan: any;
  private node!: any;
  private rootContainer: any;
  private menuHidden: boolean = true;

  private viewOptions: any = {
    showHighlightBar: false,
    showPlannerEstimate: false,
    showTags: true,
    highlightType: HighlightType.NONE,
    viewMode: ViewMode.FULL,
  };

  private highlightTypes = HighlightType;
  private viewModes = ViewMode;

  private planService = new PlanService();

  private created(): void {
    this.loadPlan();
  }

  private loadPlan(): void {
    let planJson = this.planJson;
    if (planJson instanceof Array) {
      planJson = planJson[0];
    }
    this.node = planJson.Plan;
    this.plan = this.planService.createPlan('', planJson, this.planQuery);
    this.rootContainer = this.plan.content;
    this.plan.planStats = {
      executionTime: this.rootContainer['Execution Time'] || this.rootContainer['Total Runtime'],
      planningTime: this.rootContainer['Planning Time'] || 0,
      maxRows: this.rootContainer[this.planService.MAXIMUM_ROWS_PROP] || 0,
      maxCost: this.rootContainer[this.planService.MAXIMUM_COSTS_PROP] || 0,
      maxDuration: this.rootContainer[this.planService.MAXIMUM_DURATION_PROP] || 0,
    };
  }
}
</script>

<style lang="scss">
@import '../assets/scss/plan';

</style>