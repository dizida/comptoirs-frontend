<script setup>
// Importation des méthodes nécessaires depuis des modules externes
import {doAjaxRequest} from "@/api"; // Importation de la fonction pour faire des requêtes Ajax
import {onMounted, reactive} from "vue"; // Importation des fonctions onMounted et reactive de Vue.js

// Définition d'un objet réactif pour stocker les données de l'application
let data = reactive({
  categories: [], // Liste des catégories de produits
  selectedCategory: null, // Catégorie sélectionnée
  listOfProducts: [], // Liste des produits
  page: 0, // Numéro de la page actuelle
  size: 5, // Nombre d'éléments par page
  totalPages: 0, // Nombre total de pages
  totalElements: 0, // Nombre total d'éléments
});

// Fonction pour charger les catégories de produits depuis l'API
function chargeCategories() {
  doAjaxRequest("/api/categories")
      .then((json) => {
        data.categories = json._embedded.categories; // Met à jour la liste des catégories
      })
      .catch(showError); // Gère les erreurs en affichant un message d'erreur
}

// Fonction pour charger les produits depuis l'API
function chargeProducts() {
  let url = `/api/produits?page=${data.page}&size=${data.size}`;
  if (data.selectedCategory) {
    url = `/api/categories/${data.selectedCategory}/produits?page=${data.page}&size=${data.size}`;
  }
  doAjaxRequest(url)
      .then((json) => {
        data.listOfProducts = json._embedded.produits; // Met à jour la liste des produits
        if (json.page) {
          data.page = json.page.number; // Met à jour le numéro de page actuelle
          data.totalPages = json.page.totalPages; // Met à jour le nombre total de pages
          data.totalElements = json.page.totalElements; // Met à jour le nombre total d'éléments
        } else {
          data.page = 0;
          data.totalPages = 1;
          data.totalElements = data.listOfProducts.length; // S'il n'y a pas de pagination, met à jour le nombre total d'éléments
        }
      })
      .catch(showError); // Gère les erreurs en affichant un message d'erreur
}

// Fonction pour afficher les erreurs
function showError(error) {
  console.log("Erreur : status %d", error.status);
  console.log(error.body);
  alert(error.message);
}

// Fonctions de navigation entre les pages
function goToFirstPage() {
  data.page = 0;
  chargeProducts();
}

function goToPreviousPage() {
  if (data.page > 0) {
    data.page--;
    chargeProducts();
  }
}

function goToNextPage() {
  if (data.page < data.totalPages - 1) {
    data.page++;
    chargeProducts();
  }
}

function goToLastPage() {
  data.page = data.totalPages - 1;
  chargeProducts();
}

// Fonction appelée lors du changement de catégorie
function onCategoryChange() {
  data.page = 0; // Réinitialise la page à 0
  chargeProducts(); // Charge les produits de la nouvelle catégorie
}

// Fonction exécutée une fois que le composant est monté
onMounted(() => {
  chargeCategories(); // Charge les catégories de produits
  chargeProducts(); // Charge les produits
});
</script>

<template>
  <div class="table-container">
    <h1>Categories de Produits</h1>

    <!-- Sélecteur de catégories -->
    <div class="category-filter">
      <label for="category-select">Sélectionner une catégorie :</label>
      <select id="category-select" v-model="data.selectedCategory" @change="onCategoryChange">
        <option value="">Toutes les catégories</option>
        <option v-for="category in data.categories" :key="category.code" :value="category.code">
          {{ category.libelle }}
        </option>
      </select>
    </div>

    <!-- Tableau de produits -->
    <div class="table-content">
      <table>
        <caption>Liste des produits</caption>
        <tr>
          <th>Nom</th>
          <th>Prix</th>
          <th>Stock</th>
          <th>Commandés</th>
        </tr>
        <!-- Affichage d'un message pendant le chargement des produits -->
        <tr v-if="data.listOfProducts.length === 0">
          <td colspan="4">Veuillez patienter, chargement des produits...</td>
        </tr>
        <!-- Affichage des produits -->
        <tr v-for="product in data.listOfProducts" :key="product.reference">
          <td>{{ product.nom }}</td>
          <td>{{ product.prixUnitaire }}</td>
          <td>{{ product.unitesEnStock }}</td>
          <td>{{ product.unitesCommandees }}</td>
        </tr>
      </table>
    </div>

    <!-- Pagination -->
    <div class="pagination">
      <button @click="goToFirstPage" :disabled="data.page === 0">Début</button>
      <button @click="goToPreviousPage" :disabled="data.page === 0">Précédent</button>
      <span>Page {{ data.page + 1 }} sur {{ data.totalPages }}</span>
      <button @click="goToNextPage" :disabled="data.page === data.totalPages - 1">Suivant</button>
      <button @click="goToLastPage" :disabled="data.page === data.totalPages - 1">Fin</button>
    </div>
  </div>
</template>

<!-- Styles CSS spécifiques au composant -->
<style scoped>
.table-container {
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: center;
}

.table-content {
  flex: 1;
  overflow-y: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

td,
th {
  border: 1px solid #ddd;
  padding: 8px;
}

th {
  background-color: #232623;
  color: #fff;
}

.pagination {
  margin-top: 1rem;
}

.pagination button {
  padding: 0.5rem 1rem;
  background-color: #007bff;
  color: white;
  border: none;
  cursor: pointer;
  margin: 0 0.5rem;
}

.pagination button:disabled {
  background-color: #c0c0c0;
  cursor: not-allowed;
}

.category-filter {
  margin-bottom: 1rem;
}
</style>
