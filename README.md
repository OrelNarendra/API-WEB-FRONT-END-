<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Food Recipe</title>

<link rel="preconnect" href="https://www.themealdb.com">
<link rel="dns-prefetch" href="https://www.themealdb.com">

<style>
:root {
    --primary: #c65d3b;
    --secondary: #4f6f52;
    --card: #ffffff;
}

* { box-sizing: border-box; }

body {
    margin: 0;
    font-family: 'Segoe UI', sans-serif;
    background-image: url("https://img.freepik.com/free-vector/arabesque-decorative-brocade_1217-1560.jpg");
    background-size: cover;
    background-position: center;
    background-attachment: fixed;
    color: #333;
}

body::before {
    content: "";
    position: fixed;
    inset: 0;
    backdrop-filter: blur(6px);
    -webkit-backdrop-filter: blur(6px);
    background: rgba(249, 246, 242, 0.55);
    z-index: -1;
}

header {
    background: linear-gradient(to right, var(--primary), var(--secondary));
    color: white;
    text-align: center;
    padding: 25px 15px;
}

.search-section {
    display: flex;
    justify-content: center;
    gap: 10px;
    padding: 20px;
}

.search-section input {
    width: 60%;
    max-width: 300px;
    padding: 10px;
    border-radius: 25px;
    border: 1px solid #ccc;
}

.search-section button {
    padding: 10px 20px;
    border-radius: 25px;
    border: none;
    background: var(--primary);
    color: white;
    cursor: pointer;
}

section { padding: 20px; }

h2 { color: var(--secondary); }

.grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(180px, 1fr));
    gap: 15px;
}

.grid.small {
    grid-template-columns: repeat(auto-fill, minmax(120px, 1fr));
}

.card {
    background: var(--card);
    border-radius: 15px;
    overflow: hidden;
    box-shadow: 0 4px 10px rgba(0,0,0,0.15);
    cursor: pointer;
    transition: transform .3s;
}

.card:hover { transform: translateY(-5px); }

.card img {
    width: 100%;
    display: block;
}

.card h3 {
    padding: 10px;
    font-size: 15px;
    text-align: center;
}

.recipe {
    background: var(--card);
    padding: 20px;
    border-radius: 15px;
    box-shadow: 0 4px 10px rgba(0,0,0,0.15);
}

.recipe img {
    width: 100%;
    max-width: 350px;
    border-radius: 15px;
}

.skeleton {
    height: 220px;
    border-radius: 15px;
    background: linear-gradient(90deg,#eee 25%,#f5f5f5 37%,#eee 63%);
    background-size: 400% 100%;
    animation: shimmer 1.4s infinite;
}

@keyframes shimmer {
    0% { background-position: 100% 0 }
    100% { background-position: -100% 0 }
}

@media (max-width:600px) {
    .search-section { flex-direction: column; }
    .search-section input { width: 100%; }
}
</style>
</head>

<body>

<header>
    <h1>🍽️ Food Recipe</h1>
    <p>Modern Taste with Indonesian Soul</p>
</header>

<section class="search-section">
    <input type="text" id="searchInput" placeholder="Cari menu makanan...">
    <button onclick="searchMeal()">Search</button>
</section>

<section>
    <h2>🔥 Latest Meals</h2>
    <div id="latestMeals" class="grid"></div>
</section>

<section>
    <h2>⭐ Popular Ingredients</h2>
    <div id="ingredients" class="grid small"></div>
</section>

<section>
    <h2>📖 Recipe Detail</h2>
    <div id="recipeDetail" class="recipe"></div>
</section>

<script>
const latestMeals = document.getElementById("latestMeals");
const ingredientsDiv = document.getElementById("ingredients");
const recipeDetail = document.getElementById("recipeDetail");

for (let i = 0; i < 6; i++) {
    latestMeals.innerHTML += `<div class="skeleton"></div>`;
    ingredientsDiv.innerHTML += `<div class="skeleton" style="height:80px"></div>`;
}

const cachedMeals = sessionStorage.getItem("latestMeals");

if (cachedMeals) {
    renderMeals(JSON.parse(cachedMeals));
} else {
    fetch("https://www.themealdb.com/api/json/v1/1/search.php?s=")
        .then(res => res.json())
        .then(data => {
            const meals = data.meals.slice(0, 8);
            sessionStorage.setItem("latestMeals", JSON.stringify(meals));
            renderMeals(meals);
        });
}

function renderMeals(meals) {
    latestMeals.innerHTML = "";
    meals.forEach(meal => {
        latestMeals.innerHTML += mealCard(meal);
    });
}

fetch("https://www.themealdb.com/api/json/v1/1/list.php?i=list")
    .then(res => res.json())
    .then(data => {
        ingredientsDiv.innerHTML = "";
        data.meals.slice(0, 8).forEach(item => {
            ingredientsDiv.innerHTML += `
                <div class="card">
                    <h3>${item.strIngredient}</h3>
                </div>
            `;
        });
    });

function searchMeal() {
    const keyword = document.getElementById("searchInput").value.trim();
    if (!keyword) return;

    latestMeals.innerHTML = "<p>⏳ Mencari...</p>";

    fetch(`https://www.themealdb.com/api/json/v1/1/search.php?s=${keyword}`)
        .then(res => res.json())
        .then(data => {
            latestMeals.innerHTML = "";
            if (!data.meals) {
                latestMeals.innerHTML = "<p>Menu tidak ditemukan.</p>";
                return;
            }
            data.meals.forEach(meal => {
                latestMeals.innerHTML += mealCard(meal);
            });
        });
}

function showRecipe(id) {
    fetch(`https://www.themealdb.com/api/json/v1/1/lookup.php?i=${id}`)
        .then(res => res.json())
        .then(data => {
            const meal = data.meals[0];

            let ingredients = "";
            for (let i = 1; i <= 20; i++) {
                const ing = meal[`strIngredient${i}`];
                const measure = meal[`strMeasure${i}`];
                if (ing && ing.trim()) {
                    ingredients += `<li>${ing} - ${measure}</li>`;
                }
            }

            let stepsHTML = "";
            meal.strInstructions.split(".")
                .filter(s => s.trim())
                .forEach((step, i) => {
                    stepsHTML += `<p><b>Step ${i+1}:</b> ${step.trim()}.</p>`;
                });

            recipeDetail.innerHTML = `
                <h3>${meal.strMeal}</h3>
                <img src="${meal.strMealThumb}">
                <h4>Bahan-bahan</h4>
                <ul>${ingredients}</ul>
                <h4>Cara Memasak</h4>
                ${stepsHTML}
            `;
            recipeDetail.scrollIntoView({ behavior: "smooth" });
        });
}

function mealCard(meal) {
    return `
        <div class="card" onclick="showRecipe(${meal.idMeal})">
            <img src="${meal.strMealThumb}" loading="lazy">
            <h3>${meal.strMeal}</h3>
        </div>
    `;
}
</script>

</body>
</html>
