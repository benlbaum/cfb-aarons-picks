<template>
    <div>
      <h1>Week {{ selectedWeek }} - Aaron's Prediction Stats</h1>
      <div class="chart-container">
        <LineChart :chartData="chartData" :chartOptions="chartOptions" />
      </div>
      <select v-model="selectedWeek" @change="fetchWeekData">
        <option v-for="week in weeks" :key="week" :value="week">Week {{ week }}</option>
      </select>
    </div>
  </template>
  
  <script>
  import { Line } from 'vue-chartjs';
  import { Chart as ChartJS, Title, Tooltip, Legend, LineElement, CategoryScale, LinearScale, PointElement } from 'chart.js';
  
  // Register necessary components for Chart.js
  ChartJS.register(Title, Tooltip, Legend, LineElement, CategoryScale, LinearScale, PointElement);
  
  export default {
    components: {
      LineChart: {
        extends: Line,
        props: ['chartData', 'chartOptions'],
        mounted() {
          this.renderChart(this.chartData, this.chartOptions);
        }
      }
    },
    data() {
      return {
        selectedWeek: 2,  // Default to Week 2
        weeks: Array.from({ length: 16 }, (v, i) => i + 1),
        chartData: null,
        chartOptions: {
          responsive: true,
          maintainAspectRatio: false,
          plugins: {
            legend: { position: 'top' },
            title: { display: true, text: "Aaron's Prediction Stats" }
          }
        }
      };
    },
    methods: {
      async fetchWeekData() {
        const range = `Week${this.selectedWeek}!A1:O100`;  // Adjust range based on your data structure
        const url = `https://sheets.googleapis.com/v4/spreadsheets/${process.env.VUE_APP_SHEET_ID}/values/${range}?key=${process.env.VUE_APP_API_KEY}`;
        
        try {
          const response = await fetch(url);
          const data = await response.json();
          this.processData(data.values);
        } catch (error) {
          console.error("Error fetching data: ", error);
        }
      },
      processData(values) {
        const totalGames = values.length - 1;
        const correctPredictions = values.filter(row => row[14] === '1').length;  // Assuming column O is 'Aaron Correct'
  
        const correctAbove50 = values.filter(row => parseFloat(row[3]) > 50 && row[14] === '1').length;
        const correctAbove60 = values.filter(row => parseFloat(row[3]) > 60 && row[14] === '1').length;
        const correctAbove70 = values.filter(row => parseFloat(row[3]) > 70 && row[14] === '1').length;
        const correctAbove80 = values.filter(row => parseFloat(row[3]) > 80 && row[14] === '1').length;
        const correctAbove90 = values.filter(row => parseFloat(row[3]) > 90 && row[14] === '1').length;
  
        // Update the chart data
        this.chartData = {
          labels: ['Total Games', '>50% Correct', '>60% Correct', '>70% Correct', '>80% Correct', '>90% Correct'],
          datasets: [
            {
              label: "Aaron's Prediction Accuracy",
              data: [
                totalGames,
                correctAbove50,
                correctAbove60,
                correctAbove70,
                correctAbove80,
                correctAbove90
              ],
              borderColor: 'rgba(75, 192, 192, 1)',
              backgroundColor: 'rgba(75, 192, 192, 0.2)',
              fill: true
            }
          ]
        };
      }
    },
    mounted() {
      this.fetchWeekData();
    }
  };
  </script>
  
  <style>
  .chart-container {
    width: 80%;
    height: 400px;
    margin: 0 auto;
  }
  </style>
  