<template>
  <v-dialog
    v-model="show"
    max-width="550"
    persistent
    data-test="system-parameters-add-timestamping-service-dialog"
  >
    <template v-slot:activator="{ on: { click } }">
      <large-button
        data-test="system-parameters-timestamping-services-add-button"
        outlined
        @click="click"
        :disabled="selectableTimestampingServices.length === 0"
        :requires-permission="permissions.ADD_TSP"
      >
        {{ $t('systemParameters.timestampingServices.action.add.button') }}
      </large-button>
    </template>
    <v-card class="xrd-card">
      <v-card-title>
        <span data-test="dialog-title" class="headline">{{
          $t('systemParameters.timestampingServices.action.add.dialog.title')
        }}</span>
      </v-card-title>
      <v-card-text class="content-wrapper">
        <v-container>
          <v-row>
            <v-col>
              {{
                $t(
                  'systemParameters.timestampingServices.action.add.dialog.info',
                )
              }}
            </v-col>
          </v-row>
          <v-radio-group
            v-model="selectedTimestampingServiceName"
            data-test="system-parameters-add-timestamping-service-dialog-radio-group"
          >
            <v-row
              class="option-row"
              v-for="timestampingService in selectableTimestampingServices"
              :key="timestampingService.name"
            >
              <v-col>
                <v-radio
                  :name="timestampingService.name"
                  :label="timestampingService.name"
                  :value="timestampingService.name"
                />
              </v-col>
            </v-row>
          </v-radio-group>
        </v-container>
      </v-card-text>
      <v-card-actions class="xrd-card-actions">
        <v-spacer></v-spacer>
        <large-button
          data-test="system-parameters-add-timestamping-service-dialog-cancel-button"
          outlined
          @click="close"
          >{{ $t('action.cancel') }}</large-button
        >
        <large-button
          data-test="system-parameters-add-timestamping-service-dialog-add-button"
          :loading="loading"
          :disabled="selectedTimestampingService === undefined"
          @click="add"
          >{{ $t('action.add') }}</large-button
        >
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script lang="ts">
import Vue from 'vue';
import * as api from '@/util/api';
import { Permissions } from '@/global';
import LargeButton from '@/components/ui/LargeButton.vue';
import { Prop } from 'vue/types/options';
import { TimestampingService } from '@/openapi-types';

export default Vue.extend({
  name: 'AddTimestampingServiceDialog',
  components: {
    LargeButton,
  },
  props: {
    configuredTimestampingServices: {
      type: Array as Prop<TimestampingService[]>,
      required: true,
    },
  },
  data() {
    return {
      loading: false,
      show: false,
      approvedTimestampingServices: [] as TimestampingService[],
      selectedTimestampingServiceName: '',
      permissions: Permissions,
    };
  },
  computed: {
    selectableTimestampingServices(): TimestampingService[] {
      return [...this.approvedTimestampingServices].filter(
        (approvedService) =>
          !this.configuredTimestampingServices.some(
            (configuredService) =>
              approvedService.name === configuredService.name,
          ),
      );
    },
    selectedTimestampingService(): TimestampingService | undefined {
      return this.approvedTimestampingServices.find(
        (approvedService) =>
          approvedService.name === this.selectedTimestampingServiceName,
      );
    },
  },
  methods: {
    fetchApprovedTimestampingServices(): void {
      api
        .get<TimestampingService[]>('/timestamping-services')
        .then((resp) => (this.approvedTimestampingServices = resp.data))
        .catch((error) => this.$store.dispatch('showError', error));
    },
    add(): void {
      this.loading = true;
      api
        .post('/system/timestamping-services', this.selectedTimestampingService)
        .then(() => {
          this.$emit('added');
          this.loading = false;
          this.close();
          this.$store.dispatch(
            'showSuccess',
            'systemParameters.timestampingServices.action.add.dialog.success',
          );
        })
        .catch((error) => this.$store.dispatch('showError', error));
    },
    close(): void {
      this.show = false;
      this.selectedTimestampingServiceName = '';
    },
  },
  created(): void {
    this.fetchApprovedTimestampingServices();
  },
});
</script>

<style scoped lang="scss">
@import '../../../assets/dialogs';
@import '../../../assets/colors';
.option-row {
  border-bottom: solid 1px $XRoad-Grey10;
}

.content-wrapper {
  color: #000000 !important;
}

.v-label {
  color: $XRoad-Black !important;
}
</style>
