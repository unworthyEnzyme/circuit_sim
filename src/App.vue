<template>
  <div class="row">
    <div>
      <div class="column">
        <img src="./assets/circuit.png" alt="" width="300" />
        <h2>H Parametreleri</h2>
        <label for="h_ie">
          h<sub>ie</sub>
          <input
            type="number"
            id="h_ie"
            placeholder="Giriş empedansı (Ω)"
            v-model="h_ie"
          />
        </label>
        <label for="h_ie">
          h<sub>re</sub>
          <input
            type="number"
            id="h_ie"
            placeholder="Giriş-çıkış geri besleme katsayısı (boyutsuz)"
            v-model="h_re"
          />
        </label>
        <label for="h_ie">
          h<sub>fe</sub>
          <input
            type="number"
            id="h_ie"
            placeholder="Akım kazancı (boyutsuz)"
            v-model="h_fe"
          />
        </label>
        <label for="h_oe">
          h<sub>oe</sub>
          <input
            type="number"
            id="h_oe"
            placeholder="Çıkış iletkenliği (1/Ω)"
            v-model="h_oe"
          />
        </label>
      </div>
      <div class="column">
        <h2>Dirençler</h2>
        <label for="r_c">
          R<sub>C</sub>
          <input
            type="number"
            id="r_c"
            placeholder="Kollektör direnci (Ω)"
            v-model="r_c"
          />
        </label>
        <label for="r_e">
          R<sub>E</sub>
          <input
            type="number"
            id="r_e"
            placeholder="Emetör direnci (Ω)"
            v-model="r_e"
          />
        </label>
        <label for="r_l">
          R<sub>L</sub>
          <input
            type="number"
            id="r_l"
            placeholder="Yük direnci (Ω)"
            v-model="r_l"
          />
        </label>
        <label for="r_s">
          R<sub>S</sub>
          <input
            type="number"
            id="r_s"
            placeholder="Giriş direnci (Ω)"
            v-model="r_s"
          />
        </label>
      </div>
      <div class="column">
        <h2>Gerilim kaynakları</h2>
        <label for="v_cc">
          V<sub>CC</sub>
          <input
            type="number"
            id="v_cc"
            placeholder="Besleme gerilimi (V)"
            v-model="v_cc"
          />
          />
        </label>
        <label for="v_in">
          V<sub>in</sub>
          <input
            type="number"
            id="v_in"
            placeholder="Giriş gerilimi (V)"
            v-model="v_in"
          />
          />
        </label>
      </div>
    </div>
    <div class="column">
      <img src="./assets/circuit-dc.png" alt="" width="300" />
      <ul>
        <li>I<sub>B</sub> = {{ i_b.toExponential() }}</li>
        <li>I<sub>C</sub> = {{ i_c.toExponential() }}</li>
        <li>I<sub>E</sub> = {{ i_e.toExponential() }}</li>
        <li>V<sub>CE</sub> = {{ v_ce }}</li>
        <li>V<sub>CB</sub> = {{ v_cb }}</li>
        <li>Çalışma bölgesi: {{ calisma_bolgesi }}</li>
      </ul>
    </div>
    <div class="column">
      <img src="./assets/circuit-ac.png" width="300" />
      <ul>
        <li>V<sub>out</sub> = {{ v_out.toExponential() }}</li>
        <li>R<sub>in</sub> = {{ r_in.toExponential() }}</li>
        <li>A<sub>i</sub> = {{ a_i }}</li>
        <li>A<sub>v</sub> = {{ a_v }}</li>
        <li>Z<sub>in</sub> = {{ z_in }}</li>
        <li>Z<sub>out</sub> = {{ z_out }}</li>
        <li>A<sub>i</sub> (total) = {{ a_i_total }}</li>
        <li>A<sub>v</sub> (total) = {{ a_v_total }}</li>
      </ul>
      <div>
        <svg ref="chart" width="800" height="400"></svg>
      </div>
    </div>
  </div>
</template>

<style>
.column {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.row {
  display: flex;
  gap: 1rem;
}
</style>

<script setup lang="ts">
import { computed, ref, watch } from "vue";
import * as d3 from "d3";
//inputs
// Giriş Empedansı.
const h_ie = ref(1.1e3);
// Geri Besleme Gerilimi.
const h_re = ref(2.5e-4);
// Akım Kazancı
const h_fe = ref(100);
// Çıkış Empedansı
const h_oe = ref(25e-6);

const r_c = ref(4.7e3);
const r_e = ref(1e3);
const r_l = ref(10e3);
const r_s = ref(600);

const v_cc = ref(12);
const v_in = ref(1);

//outputs
const v_be = 0.7;
const i_b = computed(
  () =>
    (v_in.value - v_be) /
    (r_s.value + (h_ie.value + (1 + h_fe.value) * r_e.value))
);
const i_c = computed(() => h_fe.value * i_b.value);
const i_e = computed(() => i_c.value + i_b.value);
const v_ce = computed(
  () => v_cc.value - i_c.value * r_c.value - i_e.value * r_e.value
);
const v_cb = computed(() => v_ce.value + v_be);

function isClose(a: number, b: number, tolerance = 0.001) {
  return Math.abs(a - b) < tolerance;
}

const calisma_bolgesi = computed(() => {
  if (v_ce.value > v_be) return "Aktif Bolge";
  if (isClose(v_ce.value, 0.2, 0.2)) return "Doyma Bolgesi";
  return "Kesme Bolgesi";
});

const v_out = computed(() => i_e.value * r_l.value);
const r_in = computed(() => h_ie.value / (1 + h_fe.value));
const a_i = computed(() => 1 + h_fe.value);
const a_v = computed(() => v_out.value / v_in.value);
const z_in = computed(() => r_in.value + r_s.value);
const z_out = computed(() => 1 / (h_oe.value + (1 + h_fe.value) / r_e.value));
const a_i_total = computed(() => a_i.value);
const a_v_total = computed(() => v_out.value / v_in.value);

// AC analysis
const v_in_ac_points = ref<{ time: number; value: number }[]>([]);
const v_out_ac_points = ref<{ time: number; value: number }[]>([]);
const F = 1;
const MILLISECONDS_TO_SECONDS = 0.001;
const t = ref(0);
requestAnimationFrame(function step(time) {
  const v_in_ac =
    v_in.value * Math.sin(2 * Math.PI * F * time * MILLISECONDS_TO_SECONDS);
  v_in_ac_points.value.push({
    time: time * MILLISECONDS_TO_SECONDS,
    value: v_in_ac,
  });
  v_out_ac_points.value.push({
    time: time * MILLISECONDS_TO_SECONDS,
    value: a_v_total.value * v_in_ac,
  });
  t.value = time * MILLISECONDS_TO_SECONDS;
  requestAnimationFrame(step);
});

watch(t, () => {
  updateChart();
});

const chart = ref<SVGSVGElement | null>(null);
const updateChart = () => {
  const first_dataset = v_in_ac_points;
  const second_dataset = v_out_ac_points;
  const svg = d3.select(chart.value);
  svg.selectAll("*").remove();

  const margin = { top: 20, right: 20, bottom: 30, left: 50 };
  const width = +svg.attr("width") - margin.left - margin.right;
  const height = +svg.attr("height") - margin.top - margin.bottom;
  const g = svg
    .append("g")
    .attr("transform", `translate(${margin.left},${margin.top})`);

  const x = d3
    .scaleLinear()
    .domain([Math.max(0, t.value - 10), Math.max(10, t.value)]) // Fixed domain from 0 to 10 seconds
    .range([0, width]);

  const y = d3
    .scaleLinear()
    .domain([
      Math.min(
        d3.min(first_dataset.value, (d) => d.value) ?? 1,
        d3.min(second_dataset.value, (d) => d.value) ?? 1
      ),
      Math.max(
        d3.max(first_dataset.value, (d) => d.value) ?? 1,
        d3.max(second_dataset.value, (d) => d.value) ?? 1
      ),
    ]) // Fixed domain from -0.01 to 0.01
    .range([height, 0]);

  const line = d3
    .line<{ time: number; value: number }>()
    .x((d) => x(d.time)) // Offset time to keep the latest 10 seconds in view
    .y((d) => y(d.value));

  g.append("g")
    .attr("class", "axis axis--x")
    .attr("transform", `translate(0,${height})`)
    .call(
      d3
        .axisBottom(x)
        .ticks(10)
        .tickFormat((d) => `${d}s`)
    );

  g.append("g").attr("class", "axis axis--y").call(d3.axisLeft(y));

  g.append("path")
    .datum(first_dataset.value)
    .attr("d", line)
    .attr("fill", "none")
    .attr("stroke", "#35A7FF")
    .attr("stroke-width", 1.5);

  g.append("path")
    .datum(second_dataset.value)
    .attr("d", line)
    .attr("fill", "none")
    .attr("stroke", "#FF5964")
    .attr("stroke-width", 1.5);
};
</script>
