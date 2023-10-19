<script lang="ts">
  import * as mqtt from 'mqtt/dist/mqtt.min'
  import { LineChart } from 'chartist'
  import { onMount } from 'svelte'

  const client = mqtt.connect('ws://54.80.140.156:8083')

  client.on('connect', () => {
    client.subscribe('jaedson/tocloud', (err: any) => console.error(err))
  })

  interface Message {
    adc_state: {
      battery_ma: number
      battery_mv: number
      ldo_inp_mv: number
      esp_vin_mv: number
      pressure_mv: number
    }
    n_pulses: number
    time_sec: number
  }

  interface MessageBatch {
    data: Message[]
    time_sec: number
  }

  let last_values: Message[] = []
  let chart_ldo_inp_mv: LineChart
  let chart_battery_ma: LineChart
  let chart_battery_mv: LineChart
  let chart_n_pulses: LineChart
  let chart_esp_vin_mv: LineChart
  let chart_pressure_mv: LineChart

  function getLabel(msg: Message) {
    return new Date(msg.time_sec)
      .toLocaleTimeString()
      .split(':')
      .filter((_, i) => i > 0)
      .join(':')
  }

  onMount(() => {
    const default_data = {
      labels: [],
      series: [[]],
    }
    chart_ldo_inp_mv = new LineChart('#ldo_inp_mv', default_data)
    chart_battery_ma = new LineChart('#battery_ma', default_data)
    chart_battery_mv = new LineChart('#battery_mv', default_data)
    chart_n_pulses = new LineChart('#n_pulses', default_data)
    chart_esp_vin_mv = new LineChart('#esp_vin_mv', default_data)
    chart_pressure_mv = new LineChart('#pressure_mv', default_data)
  })

  client.on('message', (_topic: string, message: string) => {
    const batch = JSON.parse(message) as MessageBatch
    for (const parsed of batch.data) {
      parsed.time_sec = new Date().valueOf()
      if (last_values.length > 24) last_values.shift()
      last_values.push(parsed)
      const labels = last_values.map(getLabel)
      chart_ldo_inp_mv.update({
        labels,
        series: [last_values.map((v) => v.adc_state.ldo_inp_mv)],
      })
      chart_battery_ma.update({
        labels,
        series: [last_values.map((v) => v.adc_state.battery_ma)],
      })
      chart_battery_mv.update({
        labels,
        series: [last_values.map((v) => v.adc_state.battery_mv)],
      })
      chart_n_pulses.update({
        labels,
        series: [last_values.map((v) => v.n_pulses)],
      })
      chart_esp_vin_mv.update({
        labels,
        series: [last_values.map((v) => v.adc_state.esp_vin_mv)],
      })
      chart_pressure_mv.update({
        labels,
        series: [last_values.map((v) => v.adc_state.pressure_mv / 100)],
      })
    }
  })
</script>

<main class="p-8 flex gap-2 flex-col">
  <h1 class="text-3xl font-bold text-center mb-2">
    MONITORAMENTO DE PRESSÃO E VAZÃO UTILIZANDO INTERNET DAS COISAS E COMPUTAÇÃO
    EM NUVEM
  </h1>

  <section class="flex gap-2">
    <div class="flex flex-col flex-1">
      <h3 class="text-center">
        Tensão de entrada do LDO (mV)
      </h3>
      <div class="ct-chart w-full h-56" id="ldo_inp_mv" />
    </div>
    <div class="flex flex-col flex-1">
      <h3 class="text-center">Corrente de descarga (mA)</h3>
      <div class="ct-chart w-full h-56" id="battery_ma" />
    </div>
  </section>

  <section class="flex gap-2">
    <div class="flex flex-col flex-1">
      <h3 class="text-center">Pulsos do sensor de vazão</h3>
      <div class="ct-chart w-full h-56" id="n_pulses" />
    </div>
    <div class="flex flex-col flex-1">
      <h3 class="text-center">Tensão da bateria (mV)</h3>
      <div class="ct-chart w-full h-56" id="battery_mv" />
    </div>
  </section>

  <section class="flex gap-2">
    <div class="flex flex-col flex-1">
      <h3 class="text-center">Tensão de entrada no ESP (mV)</h3>
      <div class="ct-chart w-full h-56" id="esp_vin_mv" />
    </div>
    <div class="flex flex-col flex-1">
      <h3 class="text-center">Corrente no sensor de pressão (mA)</h3>
      <div class="ct-chart w-full h-56" id="pressure_mv" />
    </div>
  </section>

  <section class="flex justify-center">
    <a class="bg-red-500 font-bold text-xl uppercase py-4 px-8 rounded-lg text-white" href="http://54.80.140.156/result.json" download="dados.json">Baixar resultados</a>
  </section>
</main>

<style lang="postcss">
  :global(html) {
    background-color: theme(colors.gray.100);
  }
</style>
