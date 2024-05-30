<script setup>
import {doAjaxRequest} from "@/api";
import {onMounted, reactive} from "vue";

let data = reactive({
  // Liste des produits de la page actuelle
  listOfProducts: [],
  page: 0, // Initialise la page à 0
  size: 5, // Initialise la taille de la page à 5 (modifiable selon vos besoins)
  totalPages: 0,
  totalElements: 0
});

function chargeProduct() {
  // Appel à l'API pour avoir la liste des catégories
  // Trié par code, descendant
  // Verbe HTTP GET par défaut
  doAjaxRequest(`/api/produits?page=${data.page}&size=${data.size}`)
      .then((json) => {
        data.listOfProducts = json._embedded.produits;
        data.page = json.page.number;
        data.totalPages = json.page.totalPages;
        data.totalElements = json.page.totalElements;
      })
      .catch(showError);
}

function showError(error) {
  console.log("Erreur : status %d", error.status)
  console.log(error.body);
  alert(error.message);
}

function goToFirstPage() {
  chargeProduct();
}

function goToPreviousPage() {
  if (data.page > 0) {
    data.page--; // Décrémentez simplement la valeur de la page
    chargeProduct();
  }
}

function goToNextPage() {
  if (data.page < data.totalPages - 1) {
    data.page++; // Incrémente simplement la valeur de la page
    chargeProduct();
  }
}

function goToLastPage() {
  data.page = data.totalPages - 1; // Allez à la dernière page
  chargeProduct();
}


// Charge les produits lorsque le composant est monté
onMounted(chargeProduct);
</script>

<template>
  <div class="table-container">
    <h1>Product</h1>

    <div class="table-content">
      <table>
        <caption>List of products</caption>
        <tr>
          <th>Nom</th>
          <th>Prix</th>
          <th>Stock</th>
          <th>Commandés</th>
        </tr>
        <!-- Si le tableau des produits est vide -->
        <tr v-if="data.listOfProducts.length === 0">
          <td colspan="4">Veuillez patienter, chargement des catégories...</td>
        </tr>
        <!-- Si le tableau des produits n'est pas vide -->
        <tr v-for="product in data.listOfProducts" :key="product.reference">
          <td>{{ product.nom }}</td>
          <td>{{ product.prixUnitaire }}</td>
          <td>{{ product.unitesEnStock }}</td>
          <td>{{ product.unitesCommandees }}</td>
        </tr>
      </table>
    </div>

    <div class="pagination">
      <button @click="goToFirstPage" :disabled="data.page === 0">Début</button>
      <button @click="goToPreviousPage" :disabled="data.page === 0">Précédent</button>
      <span>Page {{ data.page + 1 }} sur {{ data.totalPages }}</span>
      <button @click="goToNextPage" :disabled="data.page === data.totalPages - 1">Suivant</button>
      <button @click="goToLastPage" :disabled="data.page === data.totalPages - 1">Fin</button>
    </div>
  </div>
</template>

<style scoped>
.table-container {
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: center;
}

.table-content {
  flex: 1; /* Permet au contenu du tableau de prendre autant d'espace que possible */
  overflow-y: auto; /* Ajoute une barre de défilement verticale si nécessaire */
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
  margin-top: 1rem; /* Espacement entre la table et la pagination */
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
</style>
