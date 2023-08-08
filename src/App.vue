<template>
  <div id="app">
    <!-- Navbar -->
    <nav class="navbar">
      <button class="nav-button" :class="{ active: showTableContent }" @click="showTable">Tableau</button>
      <button class="nav-button" :class="{ active: showAPIPageContent }" @click="showAPIPage">Page de l'API</button>
    </nav>

    <!-- Table -->
    <div class="content" :class="{ show: showTableContent }">
      <h1>Liste des joueurs :</h1>
      <table>
        <!-- Table header -->
        <thead>
        <tr>
          <th>Monde</th>
          <th>Joueur</th>
          <th>Action</th>
          <th>Heure</th>
        </tr>
        </thead>
        <!-- Table body -->
        <tbody>
        <tr v-for="(row, index) in displayedData" :key="index">
          <td>{{ row.World }}</td>
          <td>{{ row.Player }}</td>
          <td>{{ row.Action }}</td>
          <td>{{ row.Timestamp }}</td>
        </tr>
        </tbody>
      </table>
      <!-- Pagination -->
      <div class="pagination">
        <button class="page-button" @click="prevPage" :disabled="currentPage === 1">&lt; Précédent</button>
        <button v-for="pageNumber in totalPages" :key="pageNumber" @click="gotoPage(pageNumber)" class="page-button">
          {{ pageNumber }}
        </button>
        <button class="page-button" @click="nextPage" :disabled="currentPage === totalPages">Suivant &gt;</button>
      </div>
    </div>

    <!-- API Page -->
    <div class="content" :class="{ show: showAPIPageContent }">
      <h1>Page de l'API :</h1>
      <div class="input-container">
        <input class="api-input" type="text" v-model="apiUrl" placeholder="Entrez l'URL de l'API..." />
        <button class="save-button" @click="saveAPIUrl">Enregistrer</button>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      tableData: [],
      itemsPerPage: 50,
      currentPage: 1,
      latestDataTimestamp: null,
      intervalId: null,
      showTableContent: true,
      showAPIPageContent: false,
      apiUrl: 'https://hungrycodes.alwaysdata.net/api.php', // URL de l'API par défaut
    };
  },
  computed: {
    totalPages() {
      return Math.ceil(this.tableData.length / this.itemsPerPage);
    },
    displayedData() {
      const start = (this.currentPage - 1) * this.itemsPerPage;
      const end = this.currentPage * this.itemsPerPage;
      return this.tableData.slice(start, end);
    },
  },
  created() {
    this.startPolling(); // Démarrer la vérification périodique des nouvelles données
  },
  beforeUnmount() {
    this.stopPolling(); // Arrêter la vérification périodique avant que le composant soit détruit
  },
  methods: {
    fetchTableData() {
      axios
          .get(this.apiUrl) // Utiliser l'URL de l'API actuelle
          .then((response) => {
            console.log('Nouvelles données reçues :', response.data);

            const newData = response.data;
            if (newData.length > 0) {
              // Vérifier si la dernière donnée reçue a le même timestamp que la dernière donnée existante
              const lastExistingData = this.tableData.length > 0 ? this.tableData[0] : null;
              const lastNewData = newData[0];

              if (!lastExistingData || lastNewData.Timestamp !== lastExistingData.Timestamp) {
                // Si les timestamps sont différents, remplacer les données actuelles par les nouvelles données
                this.tableData = newData;
              }
            } else {
              // Si l'API est vide, supprimer le contenu du tableau
              this.tableData = [];
              // Réinitialiser le numéro de page courant à 1
              this.currentPage = 1;
            }
            this.checkEmptyPages();
          })
          .catch((error) => {
            console.error('Erreur lors de la récupération des données :', error);
          });
    },
    checkEmptyPages() {
      const totalItems = this.tableData.length;
      const remainingItems = totalItems % this.itemsPerPage;

      if (remainingItems === 0 && totalItems > 0) {
        // Si toutes les pages sont pleines, supprimer la dernière page vide
        this.tableData.pop();
      }
    },
    startPolling() {
      this.fetchTableData(); // Appeler fetchTableData une fois pour récupérer les données au démarrage
      this.intervalId = setInterval(() => {
        this.fetchTableData(); // Appeler fetchTableData toutes les 5 secondes et mettre à jour le tableau
      }, 5000);
    },
    stopPolling() {
      clearInterval(this.intervalId); // Arrêter la vérification périodique lorsque le composant est détruit
    },
    nextPage() {
      if (this.currentPage < this.totalPages) {
        this.currentPage += 1;
      }
    },
    prevPage() {
      if (this.currentPage > 1) {
        this.currentPage -= 1;
      }
    },
    gotoPage(pageNumber) {
      this.currentPage = pageNumber;
    },
    showTable() {
      this.showTableContent = true;
      this.showAPIPageContent = false;
    },
    showAPIPage() {
      this.showTableContent = false;
      this.showAPIPageContent = true;
    },
    saveAPIUrl() {
      // Enregistrer l'URL de l'API saisie dans le champ de saisie
      this.apiUrl = this.apiUrl.trim();
      if (this.apiUrl) {
        // Vérifier si l'URL n'est pas vide
        this.startPolling(); // Redémarrer la vérification périodique avec la nouvelle URL
        this.showTable(); // Afficher le contenu du tableau après avoir enregistré l'URL
      }
    },
  },
};
</script>

<style>
/* Styles specific to the component go here */
body {
  font-family: Arial, sans-serif;
  margin: 0;
  padding: 0;
}

#app {
  padding: 20px;
}

.navbar {
  display: flex;
  justify-content: center;
  margin-bottom: 20px;
}

.nav-button {
  padding: 10px 20px;
  background-color: #f2f2f2;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  outline: none;
}

.nav-button.active {
  background-color: #ddd;
}

.content {
  display: none;
}

.content.show {
  display: block;
}

h1 {
  margin-bottom: 10px;
}

table {
  border-collapse: collapse;
  width: 100%;
}

th,
td {
  border: 1px solid #ddd;
  padding: 8px;
}

th {
  background-color: #f2f2f2;
}

.pagination {
  margin-top: 20px;
  display: flex;
  justify-content: center;
}

.page-button {
  padding: 5px 10px;
  background-color: #f2f2f2;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  outline: none;
  margin: 0 5px;
}

.page-button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.input-container {
  display: flex;
  align-items: center;
}

.api-input {
  flex: 1;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 5px 0 0 5px;
}

.save-button {
  padding: 8px 16px;
  background-color: #4caf50;
  color: white;
  border: none;
  border-radius: 0 5px 5px 0;
  cursor: pointer;
  outline: none;
}

.save-button:hover {
  background-color: #45a049;
}

.api-input:focus {
  border-color: #4caf50;
}
</style>
