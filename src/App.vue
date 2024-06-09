<template>
  <div class="row">
    <div>
      <div class="column">
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
import { computed, ref } from "vue";
//inputs
const h_ie = ref(1.1e3);
const h_re = ref(2.5e-4);
const h_fe = ref(100);
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

const calisma_bolgesi = computed(() => {
  if (v_ce.value < 0) {
    return "Kesim Bölgesi";
  } else if (v_ce.value < v_be) {
    return "Doyma Bölgesi";
  } else {
    return "Aktif Bölge";
  }
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
const t = ref(0);
const v_in_ac_points = ref<number[]>([]);
const v_out_ac_points = ref<number[]>([]);
const F = 1e3;
setInterval(() => {
  //TODO: for now i'm using 10mV amplitude but this should configurable.
  const v_in_ac = 0.01 * Math.sin(2 * Math.PI * F * t.value);
  v_in_ac_points.value.push(v_in_ac);
  v_out_ac_points.value.push(a_v_total.value * v_in_ac);
  t.value += 0.01;
}, 10);
</script>
