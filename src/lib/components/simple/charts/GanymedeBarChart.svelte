<script lang="ts">
  import { Bar } from 'svelte-chartjs'
  import { theme } from '$lib';
  import {
    Chart as ChartJS,
    Title,
    Tooltip,
    Legend,
    LineElement,
    LinearScale,
    PointElement,
    CategoryScale,
    BarElement,
  } from 'chart.js';
  import { onMount, type ComponentProps } from 'svelte';
  import lodash from 'lodash'

  ChartJS.register(
    Title,
    Tooltip,
    Legend,
    LineElement,
    BarElement,
    LinearScale,
    PointElement,
    CategoryScale
  );

  type TooltipLabelParameter = Parameters<NonNullable<NonNullable<NonNullable<NonNullable<NonNullable<ComponentProps<Bar>['options']>['plugins']>['tooltip']>['callbacks']>['label']>>[0]

  export let data: {
      labels: string[],
      datasets: {
        label: string,
        data: number[],
        backgroundColor?: string,
        borderColor?: string,
        hoverBackgroundColor?: string[]
        tension?: number,
      }[]
    } = {
      labels: [],
      datasets: []
    },
    horizontal: boolean = false,
    responsive: boolean = true,
    maintainAspectRatio: boolean = true,
    showLegend: boolean = true,
    showYTicks: boolean = false,
    showXTicks: boolean = false,
    displayYGrid: boolean = true,
    lineWidth: number = 1,
    enableZoom: boolean = true,
    resetZoom: boolean = false,
    tooltipLabel: ((tooltip: TooltipLabelParameter) => string) | undefined = undefined,
    yTickLabel: ((tickValue: string | number, index: number, ticks: any[]) => (string | number)) | undefined = undefined,
    xTickLabel: ((tickValue: string | number, index: number, ticks: any[]) => (string | number)) | undefined = undefined,
    xTickStepSize: number | undefined = undefined,
    yTickStepSize: number | undefined = undefined,
    xMax: number | undefined = undefined,
    yMax: number | undefined = undefined,
    rgbTooltipColor: string | undefined = undefined,
    rgbTooltipBackgroundColor: string | undefined = undefined,
    rgbBackgroundColor: string | undefined = undefined,
    width: string | number | undefined = undefined,
    height: string | number | undefined = undefined

  let mounted: boolean = false,
    zoomMounted: boolean = false

  onMount(() => {
    mounted = true
  })


  $: if(!rgbTooltipColor && !!$theme.colors?.[$theme.active]['dark']['primary']['300']) 
    rgbTooltipColor = $theme.colors?.[$theme.active]['dark']['primary']['300']
  $: if(!rgbTooltipBackgroundColor && !!$theme.colors?.[$theme.active]['dark']['primary']['900']) 
    rgbTooltipBackgroundColor = $theme.colors?.[$theme.active]['dark']['primary']['900']
  $: if(!rgbBackgroundColor && !!$theme.colors?.[$theme.active]['dark']['background']['200']) 
    rgbBackgroundColor = $theme.colors?.[$theme.active]['dark']['background']['200']

  $: finalTooltipColor = !!rgbTooltipColor ? `rgb(${rgbTooltipColor})` : undefined
  $: finalTooltipBackgroundColor = !!rgbTooltipBackgroundColor ? `rgb(${rgbTooltipBackgroundColor})` : undefined
  $: finalBackgroundColor = !!rgbBackgroundColor ? `rgb(${rgbBackgroundColor}, .3)` : undefined  

  let chart: ComponentProps<Bar>['chart']
  $: if(!!chart && !!enableZoom && !!resetZoom) {
    chart.resetZoom()
    resetZoom = false
  }

  $: if(mounted) {
    import('chartjs-plugin-zoom').then(({ default: zoomPlugin }) => {
      ChartJS.register(zoomPlugin)
      zoomMounted = true
      setTimeout(() => {
        chart?.resetZoom()
      }, 20);
    })
  }

  let chartOptions: ComponentProps<Bar>['options']
  $: chartOptions = {
      barPercentage: 0.9,
      borderRadius: 2,
      categoryPercentage: 0.3,
      indexAxis: horizontal ? 'y' : 'x',
      responsive: responsive,
      maintainAspectRatio: maintainAspectRatio,
      plugins: {
        tooltip: {
          displayColors: false,
          titleColor: finalTooltipColor,
          backgroundColor: finalTooltipBackgroundColor,
          titleFont: {
            size: 14
          },
          bodyFont: {
            size: 14,
            weight: "bold"
          },
          callbacks: {
            label: tooltipLabel
          }
        },
        legend: {
          display: showLegend
        },
        zoom: {
          pan: {
            enabled: enableZoom,
            mode: 'x',
            modifierKey: 'ctrl',
          },
          zoom: {
            drag: {
              enabled: enableZoom
            },
            mode: 'x',
          },
        }
      },
      interaction: {
        intersect: false,
      },
      scales: {
        x: {
          max: xMax,
          display: true,
          title: {
            display: true
          },
          grid: {
            display: false
          },
          border: {
            display: false
          },
          ticks: {
            display: showXTicks,
            callback: xTickLabel,
            stepSize: xTickStepSize
          }
        },
        y: {
          max: yMax,
          display: displayYGrid,
          title: {
          },
          grid: {
            lineWidth: lineWidth,
            color: finalBackgroundColor
          },
          border: {
            dash: [10,10],
            display: false
          },
          ticks: {
            display: showYTicks,
            callback: yTickLabel,
            stepSize: yTickStepSize
          }
        }
      }
    }
</script>

<Bar
  bind:data={data}
  options={chartOptions}
  bind:chart={chart}
  bind:width={width}
  bind:height={height}
></Bar>