<template>
  <canvas
    v-if="csvContents.length"
    id="myChart"
    width="1400"
    height="700"
  ></canvas>
  <button v-if="myChart" @click="resetZoom">Reset zoom</button>
  <form action="/target" class="dropzone"></form>
</template>

<script>
import {
  Chart,
  ArcElement,
  LineElement,
  BarElement,
  PointElement,
  BarController,
  BubbleController,
  DoughnutController,
  LineController,
  PieController,
  PolarAreaController,
  RadarController,
  ScatterController,
  CategoryScale,
  LinearScale,
  LogarithmicScale,
  RadialLinearScale,
  TimeScale,
  TimeSeriesScale,
  Decimation,
  Filler,
  Legend,
  Title,
  Tooltip,
  SubTitle,
} from 'chart.js';
import 'chartjs-adapter-luxon';

Chart.register(
  ArcElement,
  LineElement,
  BarElement,
  PointElement,
  BarController,
  BubbleController,
  DoughnutController,
  LineController,
  PieController,
  PolarAreaController,
  RadarController,
  ScatterController,
  CategoryScale,
  LinearScale,
  LogarithmicScale,
  RadialLinearScale,
  TimeScale,
  TimeSeriesScale,
  Decimation,
  Filler,
  Legend,
  Title,
  Tooltip,
  SubTitle
);

import zoomPlugin from 'chartjs-plugin-zoom';
Chart.register(zoomPlugin);

import Dropzone from 'dropzone';
import 'dropzone/dist/dropzone.css';

const COLORS = ['blue', 'red', 'green', 'orange'];

export default {
  data() {
    return {
      csvContents: [],
      myChart: null,
    };
  },

  methods: {
    processCSVtoDataset(csv) {
      const rows = csv.split('\n');
      const header = rows.shift(); // get header

      const data = [];
      let firsTS = null;
      for (const row of rows) {
        if (!row) continue;
        const cells = row.split(',');

        const ts = Number(cells[0]);
        const elapsed = Number(cells[1]);
        if (firsTS == null) {
          firsTS = ts;
        }

        data.push({
          x: ts - firsTS,
          y: elapsed,
        });
      }

      //console.log('data', data);
      return data;
    },

    update() {
      if (this.myChart) {
        this.myChart.destroy();
      }

      const ctx = document.getElementById('myChart').getContext('2d');

      const data = {
        datasets: this.csvContents.map((csv, i) => {
          return {
            label: 'Requests #' + (i + 1),
            data: this.processCSVtoDataset(csv),
            backgroundColor: COLORS[i],
          };
        }),
      };

      this.myChart = new Chart(ctx, {
        type: 'scatter',
        data: data,
        options: {
          responsive: true,
          animation: false,
          animations: { colors: false, x: false },
          transitions: {
            active: { animation: { duration: 0 } },
            zoom: {
              animation: {
                duration: 0,
              },
            },
          },
          /*interaction: {
            mode: 'point',
          },*/
          elements: {
            point: {
              radius: 1,
              borderWidth: 0,
              hitRadius: 0,
              hoverRadius: 1,
              hoverBorderWidth: 0,
            },
          },
          scales: {
            xAxis: {
              type: 'time',
              time: {
                parser: false,
                minUnit: 'minute',
                //displayFormats: 'mm:ss',
              },
            },
            yAxis: {
              title: {
                display: true,
                text: 'Milliseconds',
              },
            },
          },
          plugins: {
            tooltip: { enabled: false },
            legend: {
              enabled: false,
            },

            title: {
              display: true,
              text: 'Chart.js Scatter Chart',
            },

            zoom: {
              zoom: {
                wheel: {
                  enabled: true,
                },
                pinch: {
                  enabled: true,
                },
                drag: {
                  enabled: true,
                },
                mode: 'x',
              },
            },
          },
        },
      });
    },

    async readFile(file) {
      return new Promise((resolve) => {
        const reader = new FileReader();
        reader.addEventListener('load', (event) => {
          resolve(event.target.result);
        });
        reader.readAsText(file);
      });
    },

    resetZoom() {
      this.myChart.resetZoom();
    },
  },

  mounted() {
    let myDropzone = new Dropzone('.dropzone', {
      autoProcessQueue: false,
      addRemoveLinks: true,
    });
    myDropzone.on('addedfile', async (file) => {
      console.log(`File selected: ${file.name}`, file.type);
      if (file.type == 'application/vnd.ms-excel' || file.type == 'text/csv') {
        this.csvContents.push(await this.readFile(file));
        file.index = this.csvContents.length - 1;
        //console.log('csvContent', this.csvContent);
        this.$nextTick(() => {
          this.update();
          //myDropzone.removeFile(file);
        });
      } else {
        myDropzone.removeFile(file);
        alert(
          `Invalid file format (${file.type}). Select the requests.csv file.`
        );
      }
    });

    myDropzone.on('removedfile', async (file) => {
      console.log('File removed', file);
      if (file.index != null) {
        this.csvContents.splice(file.index, 1);
        this.$nextTick(() => {
          this.update();
          //myDropzone.removeFile(file);
        });
      }
    });

    //this.update();
  },
};
</script>
<style></style>
