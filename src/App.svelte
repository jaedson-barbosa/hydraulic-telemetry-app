<script lang="ts">
  import * as mqtt from 'mqtt/dist/mqtt.min'
  import { LineChart } from 'chartist'
  import { onMount } from 'svelte'

  const client = mqtt.connect('ws://localhost:8083') // 54.80.140.156

  client.on('connect', () => {
    client.subscribe('jaedson/tocloud', (err: any) => console.error(err))
  })

  interface Message {
    adc_state: {
      battery_mv: number
      ldo_inp_mv: number
      pressure_mv: number
    }
    n_pulses: number
    time_sec: number
  }

  let last_values: Message[] = []
  let chart_ldo_inp_mv: LineChart
  let chart_battery_mv: LineChart
  let chart_n_pulses: LineChart
  let chart_pressure_mv: LineChart

  function getLabel(msg: Message, index: number) {
    if (index % 10) return ''
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
    chart_battery_mv = new LineChart('#battery_mv', default_data)
    chart_n_pulses = new LineChart('#n_pulses', default_data)
    chart_pressure_mv = new LineChart('#pressure_mv', default_data)
  })

  client.on('message', (_topic: string, message: string) => {
    const msg = JSON.parse(message) as Message
    if (last_values.length > 100) {
      last_values.shift()
    }
    msg.time_sec = new Date().valueOf()
    last_values.push(msg)
    const labels = last_values.map(getLabel)
    chart_ldo_inp_mv.update({
      labels,
      series: [last_values.map((v) => v.adc_state.ldo_inp_mv)],
    })
    chart_battery_mv.update({
      labels,
      series: [last_values.map((v) => v.adc_state.battery_mv)],
    })
    chart_n_pulses.update({
      labels,
      series: [last_values.map((v) => v.n_pulses)],
    })
    chart_pressure_mv.update({
      labels,
      series: [last_values.map((v) => v.adc_state.pressure_mv / 100)],
    })
  })
</script>

<main class="p-8 flex gap-2 flex-col">
  <h1 class="text-3xl font-bold text-center mb-2">
    MONITORAMENTO DE PRESSÃO E VAZÃO UTILIZANDO INTERNET DAS COISAS E COMPUTAÇÃO
    EM NUVEM
  </h1>

  <section class="flex gap-2">
    <div class="flex flex-col flex-1">
      <h3 class="text-center">Tensão de entrada do LDO (mV)</h3>
      <div class="ct-chart w-full h-56" id="ldo_inp_mv" />
    </div>
    <div class="flex flex-col flex-1">
      <h3 class="text-center">Tensão da bateria (mV)</h3>
      <div class="ct-chart w-full h-56" id="battery_mv" />
    </div>
  </section>

  <section class="flex gap-2">
    <div class="flex flex-col flex-1">
      <h3 class="text-center">Pulsos do sensor de vazão</h3>
      <div class="ct-chart w-full h-56" id="n_pulses" />
    </div>
    <div class="flex flex-col flex-1">
      <h3 class="text-center">Corrente no sensor de pressão (mA)</h3>
      <div class="ct-chart w-full h-56" id="pressure_mv" />
    </div>
  </section>

  <section class="flex justify-center">
    <a
      class="bg-red-500 font-bold text-xl uppercase py-4 px-8 rounded-lg text-white"
      href="http://54.80.140.156/result.json"
      download="dados.json"
    >
      Baixar resultados
    </a>
  </section>
</main>

<style lang="postcss">
  :global(html) {
    background-color: theme(colors.gray.100);
  }
</style>
