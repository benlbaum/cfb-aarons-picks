<template>
    <div id="app" class="dark-mode">
      <h1>{{ pageTitle }}</h1>
  
      <!-- Centered Week Selector without the label -->
      <div class="week-dropdown">
        <select id="week-select" v-model="selectedWeek" @change="changeWeek">
          <option v-for="week in weeks" :key="week" :value="week">Week {{ week }}</option>
        </select>
      </div>
  
      <!-- Table or No Data message -->
      <div v-if="hasData">
        <div class="table-container">
          <table>
            <thead>
              <tr>
                <th @dblclick="sortColumn('awayTeam')">Away Team</th>
                <th @dblclick="sortColumn('homeTeam')">Home Team</th>
                <th @dblclick="sortColumn('aaronPick')">Aaron's Pick</th>
                <th @dblclick="sortColumn('aaronWinProb')">Aaron's Win Probability</th>
                <th @dblclick="sortColumn('awayDKOdds')">Away DK Odds</th>
                <th @dblclick="sortColumn('dkAwayWinProb')">DK Away Win Probability</th>
                <th @dblclick="sortColumn('awayValue')">Away Value</th>
                <th @dblclick="sortColumn('awayValueGap')">Away Value Gap</th>
                <th @dblclick="sortColumn('homeDKOdds')">Home DK Odds</th>
                <th @dblclick="sortColumn('dkHomeWinProb')">DK Home Win Probability</th>
                <th @dblclick="sortColumn('homeValue')">Home Value</th>
                <th @dblclick="sortColumn('homeValueGap')">Home Value Gap</th>
                <th @dblclick="sortColumn('aaronCorrect')">Aaron Correct?</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(game, index) in filteredAndSortedGames" :key="index">
                <td>{{ game.awayTeam }}</td>
                <td>{{ game.homeTeam }}</td>
                <td>{{ game.aaronPick }}</td>
                <td>{{ game.aaronWinProb }}</td>
                <td>{{ game.awayDKOdds }}</td>
                <td>{{ game.dkAwayWinProb }}</td>
                <td>{{ game.awayValue }}</td>
                <td>{{ game.awayValueGap }}</td>
                <td>{{ game.homeDKOdds }}</td>
                <td>{{ game.dkHomeWinProb }}</td>
                <td>{{ game.homeValue }}</td>
                <td>{{ game.homeValueGap }}</td>
                <td>{{ game.aaronCorrect }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
      <div v-else>
        <p>No Data for Week {{ selectedWeek }}</p>
      </div>
    </div>
  </template>
  
  <script>
  import axios from 'axios';
  
  export default {
    data() {
      return {
        games: [],
        sortOrder: {
          column: '',
          direction: ''  // 'asc' or 'desc'
        },
        selectedWeek: 2,  // Default to Week 2
        weeks: Array.from({ length: 16 }, (v, i) => i + 1),  // Array of weeks from 1 to 16
        sheetId: process.env.VUE_APP_SHEET_ID,  // Google Sheet ID
        apiKey: process.env.VUE_APP_API_KEY,  // Google API Key
        hasData: true,  // Determines if data exists for the selected week
        pageTitle: "CFB Aaron's Picks - Week 2",  // Default title
      };
    },
    computed: {
      filteredAndSortedGames() {
        let sorted = this.games;
  
        // Apply sorting if a column is selected
        if (this.sortOrder.column) {
          sorted = sorted.sort((a, b) => {
            let valA = a[this.sortOrder.column];
            let valB = b[this.sortOrder.column];
  
            if (this.sortOrder.direction === 'asc') {
              return valA > valB ? 1 : -1;
            } else {
              return valA < valB ? 1 : -1;
            }
          });
        }
  
        return sorted;
      }
    },
    methods: {
      fetchData() {
        const range = `Week${this.selectedWeek}!A1:O100`;  // Use selected week in range
        const googleSheetAPIUrl = `https://sheets.googleapis.com/v4/spreadsheets/${this.sheetId}/values/${range}?key=${this.apiKey}`;
  
        axios.get(googleSheetAPIUrl)
          .then(response => {
            const rows = response.data.values;
            if (!rows || rows.length === 0) {
              this.hasData = false;  // No data found for this week
              this.games = [];
            } else {
              this.hasData = true;
              this.games = rows.slice(1).map(row => ({
                awayTeam: row[0],
                homeTeam: row[1],
                aaronPick: row[2],
                aaronWinProb: row[3],
                awayDKOdds: row[4],
                dkAwayWinProb: row[5],
                awayValue: row[6],
                awayValueGap: row[7],
                homeDKOdds: row[8],
                dkHomeWinProb: row[9],
                homeValue: row[10],
                homeValueGap: row[11],
                aaronCorrect: row[14]  // Adjusted to pull from column O
              }));
            }
          })
          .catch(() => {
            this.hasData = false;  // Handle cases where the sheet for the week doesn't exist
          });
      },
      sortColumn(column) {
        // Toggle sort direction if already sorting by this column
        if (this.sortOrder.column === column) {
          this.sortOrder.direction = this.sortOrder.direction === 'asc' ? 'desc' : 'asc';
        } else {
          this.sortOrder.column = column;
          this.sortOrder.direction = 'asc';  // Default to ascending order
        }
      },
      changeWeek() {
        this.pageTitle = `CFB Aaron's Picks - Week ${this.selectedWeek}`;
        this.fetchData();  // Fetch data for the new selected week
      }
    },
    mounted() {
      this.fetchData();  // Fetch data for the default week (Week 2)
    }
  };
  </script>
  
  <style scoped>
  /* General Dark Mode Styles */
  .dark-mode {
    background-color: #181818;
    color: #e0e0e0;
    font-family: 'Arial', sans-serif;
    padding: 20px;
    min-height: 100vh;
    margin: 0;  /* Remove margin */
    border: none; /* Ensure no borders */
    box-sizing: border-box; /* Ensure padding doesn't create extra space */
  }
  
  body, html {
    margin: 0;  /* Remove default margin from body and html */
    padding: 0; /* Remove default padding */
    box-sizing: border-box;
  }
  
  h1 {
    color: #ffffff;
    font-size: 1.5rem;
    text-align: center;
    margin-bottom: 20px;
  }
  
  /* Center the week dropdown */
  .week-dropdown {
    display: flex;
    justify-content: center;
    margin-bottom: 20px;
  }
  
  select {
    padding: 8px;
    background-color: #333333;
    color: #ffffff;
    border: 1px solid #555555;
    border-radius: 5px;
    font-size: 1rem;
  }
  
  /* Make the table scrollable on smaller screens */
  .table-container {
    overflow-x: auto;
  }
  
  table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 20px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  }
  
  th, td {
    padding: 12px 15px;
    text-align: center;
    border: 1px solid #333333;
    font-size: 0.9rem;
  }
  
  th {
    background-color: #2c2c2c;
    color: #ffffff;
    cursor: pointer;
  }
  
  th:hover {
    background-color: #404040;
  }
  
  tr:nth-child(even) {
    background-color: #202020;
  }
  
  tr:hover {
    background-color: #333333;
  }
  
  p {
    text-align: center;
    color: #e0e0e0;
    font-size: 1.1rem;
  }
  
  /* Mobile Responsive Adjustments */
  @media (max-width: 600px) {
    h1 {
      font-size: 1.2rem;
    }
    
    th, td {
      padding: 8px 10px;
      font-size: 0.8rem;
    }
  
    select {
      width: 100%;
      font-size: 1rem;
    }
  
    p {
      font-size: 1rem;
    }
  }
  </style>