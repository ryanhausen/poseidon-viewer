<template>
  <v-container>
    <v-form>
      <v-select
        v-model="variable"
        :items="variables"
        :item-text="'label'"
        label="variable"
        dense
        outlined
        return-object
      >
      </v-select>
      <v-select
        v-model="colormap"
        :items="colormaps"
        label="colormap"
        dense
        outlined
      ></v-select>
      <v-divider style="margin: 8px 0 8px 0"/>
          {{variable.units}}<v-img contain :src="colormapUrl" />
      <v-slider v-model="tempMin" :min="variable.vmin" :max="variable.vmax" :step="(variable.vmax-variable.vmin)/100.0" :label="'min ['+variable.units+']'" class="align-center" @end="updateValues()">
        <template v-slot:append>
          <v-text-field
            v-model="tempMin"
            type="number"
            style="width: 80px"
            @change="updateValues()"
          ></v-text-field>
        </template>
      </v-slider>
      <v-slider v-model="tempMax" :min="variable.vmin" :max="variable.vmax" :step="(variable.vmax-variable.vmin)/100.0" :label="'max ['+variable.units+']'" class="align-center" @end="updateValues()">
        <template v-slot:append>
          <v-text-field
            v-model="tempMax"
            type="number"
            style="width: 80px"
            @change="updateValues()"
          ></v-text-field>
        </template>
      </v-slider>
      <v-divider style="margin: 8px 0 8px 0"/>
      <v-slider
        v-model="tempTimestamp"
        :max="maxTimestamp"
        label="timestamp"
        class="align-center"
        @end="updateValues()"
        style="max-height:50px"
        hide-details="true"
      >
        <template v-slot:append>
          <v-text-field
            v-model="tempTimestamp"
            type="number"
            style="width: 60px"
            @change="updateValues()"
          ></v-text-field>
        </template>
      </v-slider>
      {{getDate(tempTimestamp)}}

      <v-row style="margin-top:16px">
        <v-col class="col-12">
          <v-row align="center">
            <v-text-field
              label="step"
              v-model="step"
              type="number"
              style="max-width: 60px; margin-left: 12px; margin-right: 12px"
            ></v-text-field> hours
            <v-btn icon @click="decreaseTimestamp()" style="margin-left: 16px">
              <v-icon>mdi-rewind</v-icon>
            </v-btn>
            <v-btn icon @click="increaseTimestamp()">
              <v-icon>mdi-fast-forward</v-icon>
            </v-btn>
            <!--
            <v-btn icon @click="toggleTimer()">
              <v-icon id="play">{{ icon }}</v-icon>
            </v-btn>
            -->
          </v-row>
        </v-col>
      </v-row>
      <v-divider style="margin: 16px 0 8px 0"/>
      <v-slider
        v-model="tempDepth"
        :max="maxDepth"
        label="depth"
        class="align-center"
        hide-details="true"
        @end="updateValues()"
        style="max-height:50px"
      >
        <template v-slot:append>
          <v-text-field
            v-model="tempDepth"
            type="number"
            style="width: 60px"
            @change="updateValues()"
          ></v-text-field>
        </template>
        
      </v-slider>
      {{z_levels[tempDepth]}} m
      <!--
      <v-checkbox
        v-model="showVelocity"
        label="Velocity (animation demo)"
      ></v-checkbox>
      -->
      <v-checkbox
        v-model="showGrid"
        label="Show grid"
      ></v-checkbox>
    </v-form>
  </v-container>
</template>

<script>
import { mapState } from "vuex";
import { mapFields } from "vuex-map-fields";

export default {
  data: () => ({
    step: 10,
    timer: null,
    icon: "mdi-play",
    colormapUrl: '',

    tempMin: Number(),
    tempMax: Number(),
    tempTimestamp: Number(),
    tempDepth: Number(),
  }),
  computed: {
    ...mapState(["maxTimestamp", "variables", "colormaps", "maxDepth","z_levels"]),
    ...mapFields(["timestamp", "depth", "variable", "colormap", "min", "max", "showVelocity", "showGrid"]),
  },
  mounted() {
    this.variable = this.variables[0];
  },
  watch: {
    variable: function() { 
      this.tempMin = this.variable.vmin;
      this.tempMax = this.variable.vmax;
      if (this.variable.name == 'velocity') {
        this.colormap = 'seismic'
      } 
      else if (this.variable.name == 'SST' || this.variable.name == 'Eta') {
        this.colormap = 'RdBu_r'
      }
      else if (this.variable.name == 'SSS') {
        this.colormap = 'cmo.haline'
      }
      else {
        this.colormap = 'viridis'
      }
      this.updateColormapUrl();
      this.updateValues();
    },
    min() {
      this.updateColormapUrl();
    },
    max() {
      this.updateColormapUrl();
    },
    colormap() {
      this.updateColormapUrl();
    }
  },
  methods: {
    getDate(h) {
      var d = new Date(Date.parse('2012-04-25T00:00:00.000000Z'))
      d.setHours(d.getHours()+h)
      return d.toISOString()
    },
    updateValues() {
      this.min = this.tempMin;
      this.max = this.tempMax;
      this.depth = this.tempDepth;
      this.timestamp = this.tempTimestamp;
    },
    updateColormapUrl() {
      this.colormapUrl = process.env.VUE_APP_SERVICE_URL + `/api/colormap?vmin=${this.min}&vmax=${this.max}&colormap=${this.colormap}`;
    },
    increaseTimestamp() {
      if (Number(this.tempTimestamp) + Number(this.step) <= this.maxTimestamp) {
        this.tempTimestamp = Number(this.tempTimestamp) + Number(this.step);
      } else {
        this.tempTimestamp = this.maxTimestamp;
      }
      this.updateValues();
    },
    decreaseTimestamp() {
      if (Number(this.tempTimestamp) - Number(parseInt(this.step)) >= 0) {
        this.tempTimestamp = Number(this.tempTimestamp) - Number(this.step);
      } else {
        this.timestempTimestamptamp = 0;
      }
      this.updateValues();
    },
    /*
    handleTimer() {
      if (this.timestamp == this.maxTimestamp) {
        this.cancelTimer();
      } else {
        this.increaseTimestamp();
      }
    },
    cancelTimer() {
      clearInterval(this.timer);
      this.timer = null;
      this.icon = "mdi-play";
    },
    toggleTimer() {
      if (this.timer === null) {
        this.timer = setInterval(this.handleTimer, 500);
        this.icon = "mdi-stop";
      } else {
        this.cancelTimer();
      }
    },
    */
  },
};
</script>