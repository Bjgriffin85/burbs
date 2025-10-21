<template>
  <div>
    <h1>Burbs</h1>
  </div>
  <div>

    <h2 v-if="suburb">{{ suburb }}'s Properties</h2>
    
    <div v-if="houses.length > 0">
      <h3>Properties</h3>
      
      <div class="sort-buttons">
        <button @click="toggleSort('price')" :class="{ active: sortBy === 'price' }">
          Sort by Price ({{ sortBy === 'price' ? (sortDirection === 'desc' ? 'High to Low' : 'Low to High') : 'High to Low' }})
        </button>
        <button @click="toggleSort('land_size')" :class="{ active: sortBy === 'land_size' }">
          Sort by Land Size ({{ sortBy === 'land_size' ? (sortDirection === 'desc' ? 'High to Low' : 'Low to High') : 'High to Low' }})
        </button>
        <button @click="toggleSort('price_per_sqm')" :class="{ active: sortBy === 'price_per_sqm' }">
          Sort by Price/m² ({{ sortBy === 'price_per_sqm' ? (sortDirection === 'desc' ? 'High to Low' : 'Low to High') : 'High to Low' }})
        </button>
      </div>
      
      <div v-for="house in sortedHouses" :key="house.id">
        <h4>{{ house.address.street }} {{ house.address.sal }} {{ house.address.state }}</h4>
        <p v-if="house.price">Asking Price: ${{ formatPrice(house.price) }}</p>
        <div v-if="house.attributes.land_size && house.price">
          <p v-if="house.attributes.land_size">Land Size: {{ house.attributes.land_size}} m²</p>
          <p v-if="house.attributes.land_size && house.price">Price per square meter: $
            <b>{{ pricePerSquareMeter(house.price, house.attributes.land_size) }}</b> / m²
          </p>
        </div>
      </div>
    </div>

  </div>
</template>

<script setup>
import { onMounted, ref, computed } from 'vue';

const suburb = ref('');
const houses = ref([]);
const sortBy = ref('price');
const sortDirection = ref('desc'); // 'desc' = High to Low, 'asc' = Low to High

const sortedHouses = computed(() => {
  if (!houses.value || houses.value.length === 0) return [];
  
  // Filter out units, only keep houses
  const housesOnly = houses.value.filter(item => item.property_type !== 'Unit');
  const housesCopy = [...housesOnly];
  const multiplier = sortDirection.value === 'desc' ? -1 : 1;
  
  return housesCopy.sort((a, b) => {
    let comparison = 0;
    
    if (sortBy.value === 'price') {
      const priceA = a.price || 0;
      const priceB = b.price || 0;
      comparison = (priceB - priceA) * multiplier;
    } else if (sortBy.value === 'land_size') {
      const landSizeA = parseFloat(String(a.attributes?.land_size || '0').replace(/[^\d.]/g, '')) || 0;
      const landSizeB = parseFloat(String(b.attributes?.land_size || '0').replace(/[^\d.]/g, '')) || 0;
      comparison = (landSizeB - landSizeA) * multiplier;
    } else if (sortBy.value === 'price_per_sqm') {
      const getPricePerSqm = (house) => {
        if (!house.price || !house.attributes?.land_size) return 0;
        const landSize = parseFloat(String(house.attributes.land_size).replace(/[^\d.]/g, ''));
        if (!landSize) return 0;
        return house.price / landSize;
      };
      comparison = (getPricePerSqm(b) - getPricePerSqm(a)) * multiplier;
    }
    return comparison;
  });
});

function toggleSort(sortType) {
  if (sortBy.value === sortType) {
    // Toggle direction if clicking the same button
    sortDirection.value = sortDirection.value === 'desc' ? 'asc' : 'desc';
  } else {
    // Set new sort type and reset to High to Low
    sortBy.value = sortType;
    sortDirection.value = 'desc';
  }
}

async function checkSuburb(suburbName) {
  const data = await getProperties(suburbName);
  houses.value = data.houses.results;

  suburb.value = suburbName.replace(/\+/g, ' ');
  console.log('Houses:', houses.value);

}

function formatPrice(price) {
  if (!price) return '0';
  return price.toLocaleString('en-US');
}

function pricePerSquareMeter(price, land_size) {
  if (!price || !land_size) return '0';
  // Extract numeric value from land_size string (remove m2, m², spaces, etc.)
  const numericLandSize = parseFloat(String(land_size).replace(/[^\d.]/g, ''));
  if (!numericLandSize || numericLandSize === 0) return '0';
  return (price / numericLandSize).toLocaleString('en-US', { minimumFractionDigits: 2, maximumFractionDigits: 2 });
}

async function getProperties(suburb) {
  const response_houses = await fetch(`/api/suburb/properties?suburb=${suburb}&property_type=house`, {
    method: "GET",
    headers: {
        "Authorization": "Bearer test",
        "Content-Type": "application/json"
    }
  })
  const text_houses = await response_houses.text();
  // Replace NaN with null to make it valid JSON
  const sanitized_houses = text_houses.replace(/:\s*NaN/g, ': null');
  const houses = JSON.parse(sanitized_houses);

  return { houses };
} 

onMounted(async () => {
  checkSuburb('Belmont+North');
})

</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.sort-buttons {
  margin: 20px 0;
  display: flex;
  gap: 10px;
  justify-content: center;
}

.sort-buttons button {
  padding: 10px 20px;
  background-color: #f0f0f0;
  border: 2px solid #ddd;
  border-radius: 5px;
  cursor: pointer;
  font-size: 14px;
  transition: all 0.3s ease;
}

.sort-buttons button:hover {
  background-color: #e0e0e0;
}

.sort-buttons button.active {
  background-color: #42b983;
  color: white;
  border-color: #42b983;
}
</style>
