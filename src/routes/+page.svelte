<script>
    import { onMount } from "svelte";

    // Initialisation des variables
    let listes = [];
    let message = "";

    // Sauvegarder les listes dans le localStorage
    function sauvegardeListes() {
        localStorage.setItem("todolists", JSON.stringify(listes));
    }

    // Créer une nouvelle liste
    function creationListe() {
        const nom = prompt("Nom de la nouvelle liste :");
        if (nom) {

            //Création de l'id de la liste + vérification si l'id existe déjà ou pas
            let id = nom.toLowerCase().replace(/\s+/g, "-");
            let suffixe = 1;
            let idUnique = id;

            while (listes.some((liste) => liste.id === idUnique)) {
                idUnique = `${id}-${suffixe}`;
                suffixe++;
            }

            // Mettre à jour 'listes' en réassignant un nouveau tableau
            listes = [...listes, { id: idUnique, nom }];
            sauvegardeListes();
            message = "Liste créée avec succès !";

            // Cacher le message après 5 secondes
            setTimeout(() => {
                message = "";
            }, 5000);
        } else {
            message = "Erreur : Nom invalide.";
            setTimeout(() => {
                message = "";
            }, 5000);
        }
    }

    // Supprimer une liste
    function supprimerListe(id) {
        if (confirm("Souhaitez-vous supprimer cette liste ?")) {
            listes = listes.filter((liste) => liste.id !== id);
            sauvegardeListes();
        }
    }

    // Charger les données depuis le localStorage
    onMount(() => {
        const data = localStorage.getItem("todolists");
        if (data) {
            try {
                const parsedData = JSON.parse(data);
                if (Array.isArray(parsedData)) {
                    listes = parsedData;
                } else {
                    console.error("Les données récupérées ne sont pas un tableau.");
                    listes = [];
                }
            } catch (error) {
                console.error("Erreur lors du parsing des données de localStorage :", error);
                listes = []; // Réinitialiser les listes en cas d'erreur
            }
        } else {
            listes = []; // Aucune donnée dans localStorage, initialiser un tableau vide
        }
    });
</script>

<h1>Mes to-do lists</h1>
<button on:click={creationListe}>➕ Nouvelle liste</button>

<!-- Affichage d'utilisation' -->
{#if listes.length === 0}
    <h2>Veuillez créer une ou plusieurs ToDo List</h2>
{/if}

{#if listes.length > 0}
    <h2>
        Appuyez sur une liste pour y accéder ou bien sur les boutons d'action
    </h2>
{/if}

<!-- Affichage du message de création -->
{#if message}
    <div
        class="message"
        class:success={message === "Liste créée avec succès !"}
        class:error={message !== "Liste créée avec succès !"}
    >
        {message}
    </div>
{/if}

<div class="card-container">
    {#each listes as liste (liste.id)}
        <div class="card">
            <a href={`/liste/${liste.id}`} class="card-link">
                <h2>{liste.nom}</h2>
            </a>
            <div class="card-actions">
                <button on:click={() => modifierListe(liste)}>✏️</button>
                <button on:click={() => dupliquerListe(liste)}>🔁</button>
                <button on:click={() => supprimerListe(liste.id)}>🗑️</button>
            </div>
        </div>
    {/each}
</div>


<!-- Mise en style (on peut utiliser tailwind / Bootstrap pour reduire tout ce code inbuvable) -->
<style>
    .card-container {
        display: flex;
        flex-wrap: wrap;
        gap: 20px;
    }

    .card {
        width: 200px;
        padding: 15px;
        border: 1px solid #ccc;
        border-radius: 8px;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
        text-align: center;
    }

    .card-link {
        text-decoration: none;
        color: #333;
    }

    .card-actions {
        margin-top: 10px;
    }

    .card-actions button {
        margin: 5px;
        background-color: #f0f0f0;
        border: none;
        border-radius: 4px;
        padding: 5px 10px;
        cursor: pointer;
    }

    .card-actions button:hover {
        background-color: #ddd;
    }

    .message {
        margin: 15px 0;
        padding: 10px;
        border-radius: 5px;
        font-weight: bold;
        text-align: center;
    }

    .message.success {
        background-color: #d4edda;
        color: #155724;
        border: 1px solid #c3e6cb;
    }

    .message.error {
        background-color: #f8d7da;
        color: #721c24;
        border: 1px solid #f5c6cb;
    }
</style>
