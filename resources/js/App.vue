<script setup>
import { ref, onMounted } from "vue";
import Crisis from "./pages/Crisis.vue";
import Footer from "@/components/Footer.vue";
import GameCard from "@/components/GameCard.vue";

// Importez vos images
import crisisSimulatorImage from "../assets/images/crisis-simulator.png";
import lifeSimulatorImage from "../assets/images/life-simulator.png";
import labyrintheOmbresImage from "../assets/images/labyrinthe-ombres.png";
import dernierAppelImage from "../assets/images/dernier-appel.png";

// État
const user = ref({ name: "Invité" });
const isLoading = ref(true);
const currentView = ref("dashboard"); // 'dashboard' ou 'crisis-simulator'

// Liste des jeux
const games = ref([
    {
        id: "crisis-simulator",
        title: "Crisis Simulator",
        description: "Gérez une crise nationale en temps réel",
        available: true,
        image: crisisSimulatorImage,
    },
    {
        id: "life-simulator",
        title: "Life Simulator",
        description: "Contrôlez une vie",
        available: false,
        image: lifeSimulatorImage,
    },
    {
        id: "labyrinthe-ombres",
        title: "Labyrinthe des Ombres",
        description: "Évitez les pièges et trouvez la sortie",
        available: false,
        image: labyrintheOmbresImage,
    },
    {
        id: "dernier-appel",
        title: "Dernier Appel",
        description: "Évitez la guerre nucléaire",
        available: false,
        image: dernierAppelImage,
    },
]);

// Charger les données utilisateur
onMounted(async () => {
    try {
        const response = await fetch("/user");
        if (response.ok) {
            const userData = await response.json();
            user.value = userData;
        }
    } catch (error) {
        console.error("Erreur de chargement des données utilisateur:", error);
    } finally {
        isLoading.value = false;
    }
});

// Fonctions
function startGame(gameId) {
    if (gameId === "crisis-simulator") {
        currentView.value = "crisis-simulator";
    }
}

function returnToDashboard() {
    currentView.value = "dashboard";
}

function logout() {
    const csrfToken = document
        .querySelector('meta[name="csrf-token"]')
        ?.getAttribute("content");

    fetch("/logout", {
        method: "POST",
        headers: {
            "Content-Type": "application/json",
            "X-CSRF-TOKEN": csrfToken,
            Accept: "application/json",
        },
    })
        .then(() => {
            window.location.href = "/login";
        })
        .catch((error) => {
            console.error("Erreur de déconnexion:", error);
        });
}
</script>

<template>
    <!-- Dashboard view -->
    <div v-if="currentView === 'dashboard'" class="game-platform">
        <!-- Header -->
        <header class="app-header">
            <div class="logo">
                <h1>SIMULATION CENTER</h1>
            </div>
            <div class="user-info">
                <span v-if="!isLoading">{{ user.name }}</span>
                <button @click="logout" class="logout-btn">DÉCONNEXION</button>
            </div>
        </header>

        <!-- Main content -->
        <main class="game-selection">
            <h2>SÉLECTIONNEZ UNE SIMULATION</h2>

            <div class="games-grid">
                <GameCard
                    v-for="game in games"
                    :key="game.id"
                    :game="game"
                    @start-game="startGame"
                />
            </div>
        </main>
        <Footer />
    </div>

    <!-- Crisis Simulator view -->
    <Crisis
        v-else-if="currentView === 'crisis-simulator'"
        @return-to-dashboard="returnToDashboard"
        :user-data="user"
    />
</template>

<style>
/* Styles globaux */
:root {
    --primary: #1e40af;
    --primary-light: #3b82f6;
    --secondary: #0f172a;
    --accent: #f97316;
    --success: #10b981;
    --warning: #f59e0b;
    --danger: #ef4444;
    --bg-dark: #0f172a;
    --bg-light: #f8fafc;
    --text-light: #f8fafc;
    --text-dark: #1e293b;
    --border-color: #e2e8f0;
}

* {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
}

body {
    font-family: "Inter", sans-serif;
    background-color: var(--bg-light);
    color: var(--text-dark);
    line-height: 1.5;
}

/* Structure principale */
.game-platform {
    display: flex;
    flex-direction: column;
    min-height: 100vh;
}

/* Header */
.app-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 1.5rem 2rem;
    background-color: var(--primary);
    color: white;
}

.logo h1 {
    font-size: 1.5rem;
    font-weight: 700;
    letter-spacing: 1px;
}

.user-info {
    display: flex;
    align-items: center;
    gap: 1rem;
    font-weight: 500;
}

.logout-btn {
    padding: 0.5rem 1rem;
    background-color: rgba(255, 255, 255, 0.15);
    color: white;
    border: none;
    border-radius: 4px;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.2s ease;
}

.logout-btn:hover {
    background-color: rgba(255, 255, 255, 0.25);
}

/* Main content */
.game-selection {
    flex: 1;
    padding: 2rem;
    max-width: 1200px;
    margin: 0 auto;
    width: 100%;
}

.game-selection h2 {
    font-size: 1.8rem;
    margin-bottom: 2rem;
    text-align: center;
    color: var(--primary);
    border-bottom: 2px solid var(--primary);
    padding-bottom: 0.5rem;
    display: inline-block;
}

.games-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
    gap: 2rem;
}
@media (prefers-color-scheme: dark) {
    body {
        background-color: var(--bg-dark);
    }
    .game-selection h2 {
        color: var(--primary-light);
        border-bottom: 2px solid var(--primary-light);
    }
}
@media (max-width: 768px) {
    .app-header {
        flex-direction: column;
        gap: 1rem;
        padding: 1rem;
    }

    .games-grid {
        grid-template-columns: 1fr;
    }

    .game-selection {
        padding: 1rem;
    }
}
</style>
