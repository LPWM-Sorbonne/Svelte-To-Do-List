<script>
  import { onMount } from "svelte";

  // Initialisation de variable
  let listes = [];
  let message = "";

  function sauvegardeListes() {
    localStorage.setItem("todolists", JSON.stringify(listes));
  }

  function creationListe() {
  const nom = prompt("Nom de la nouvelle liste :");
  if (nom) {
    let id = nom.toLowerCase().replace(/\s+/g, "-");
    let suffixe = 1;
    let idUnique = id;

    while (listes.some((liste) => liste.id === idUnique)) {
      idUnique = `${id}-${suffixe}`;
      suffixe++;
    }

    const newList = {
      id: idUnique,
      nom: nom,
      tasks: []  // Ici, tu initialises les tâches de la liste avec un tableau vide.
    };

    listes = [...listes, newList];
    sauvegardeListes();
    message = "Liste créée avec succès !";

    setTimeout(() => (message = ""), 5000);
  } else {
    message = "Erreur : Nom invalide.";
    setTimeout(() => (message = ""), 5000);
  }
}

  function supprimerListe(id) {
    if (confirm("Souhaitez-vous supprimer cette liste ?")) {
      listes = listes.filter((liste) => liste.id !== id);
      sauvegardeListes();
    }
  }

  function modifierListe(liste) {
    const nouveauNom = prompt("Nouveau nom pour la liste :", liste.nom);
    if (nouveauNom && nouveauNom.trim() !== "") {
      // Mise à jour du nom et de l'id (à cause du slug)
      const newId = nouveauNom.toLowerCase().replace(/\s+/g, "-");
      let suffixe = 1;
      let idUnique = newId;

      while (listes.some((l) => l.id === idUnique && l !== liste)) {
        idUnique = `${newId}-${suffixe++}`;
      }

      liste.nom = nouveauNom;
      liste.id = idUnique;

      listes = [...listes]; // Re-trigger reactivité
      sauvegardeListes();
      message = "Liste renommée avec succès !";
      setTimeout(() => (message = ""), 4000);
    }
  }

  function dupliquerListe(liste) {
  const copieNom = prompt("Nom pour la copie :", liste.nom + " (copie)");
  if (copieNom && copieNom.trim() !== "") {
    let baseId = copieNom.toLowerCase().replace(/\s+/g, "-");
    let suffixe = 1;
    let idUnique = baseId;

    while (listes.some((l) => l.id === idUnique)) {
      idUnique = `${baseId}-${suffixe++}`;
    }

    // Dupliquer les tâches de la liste
    const dupliquerTaches = (taches) => {
      return taches.map((tache) => ({
        ...tache,
        id: Date.now() + Math.random(), // On génère un nouvel ID pour chaque tâche
        children: dupliquerTaches(tache.children), // Dupliquer les sous-tâches récursivement
      }));
    };

    // Créer la nouvelle liste avec les tâches dupliquées
    const nouvelleListe = {
      id: idUnique,
      nom: copieNom,
      tasks: dupliquerTaches(liste.tasks), // Dupliquer les tâches de la liste
    };

    listes = [...listes, nouvelleListe];
    sauvegardeListes();
    message = "Liste dupliquée avec succès !";
    setTimeout(() => (message = ""), 4000);
  }
}

  onMount(() => {
    const data = localStorage.getItem("todolists");
    if (data) {
      try {
        const parsedData = JSON.parse(data);
        listes = Array.isArray(parsedData) ? parsedData : [];
      } catch (error) {
        console.error("Erreur de recherche dans le localStorage :", error);
        listes = [];
      }
    }
  });
</script>

<div class="container">
  <h1>📋 Mes To‑Do Lists</h1>

  <button class="main-btn" on:click={creationListe}>➕ Créer une liste</button>

  {#if message}
    <div class="message" class:success={message === "Liste créée avec succès !"} class:error={message !== "Liste créée avec succès !"}>
      {message}
    </div>
  {/if}

  {#if listes.length === 0}
    <p class="info-text">Aucune liste pour le moment. Commencez par en créer une 👇</p>
  {:else}
    <p class="info-text">Appuyez sur une liste pour y accéder ou bien utilisez les actions ✏️ 🔁 🗑️</p>
  {/if}

  <div class="card-container">
    {#each listes as liste (liste.id)}
      <div class="card">
        <a href={`/liste/${liste.id}`} class="card-link">
          <h2>{liste.nom}</h2>
        </a>
        <div class="card-actions">
          <button on:click={() => modifierListe?.(liste)}>✏️</button>
          <button on:click={() => dupliquerListe?.(liste)}>🔁</button>
          <button on:click={() => supprimerListe(liste.id)}>🗑️</button>
        </div>
      </div>
    {/each}
  </div>
</div>

<style>
  @import url("https://fonts.googleapis.com/css2?family=Patrick+Hand+SC&family=Poppins:ital,wght@0,100;0,200;0,300;0,400;0,500;0,600;0,700;0,800;0,900;1,100;1,200;1,300;1,400;1,500;1,600;1,700;1,800;1,900&display=swap");

  :root {
    font-family: "Poppins", sans-serif;
    --primary-color: #007aff;
    --background-color: #f2f2f7;
    --text-color: #1c1c1e;
    --card-bg: #ffffff;
    --card-shadow: rgba(0, 0, 0, 0.05);
    --radius: 12px;
  }


  .container {
    padding: 2rem;
    max-width: 900px;
    margin: auto;
    text-align: center;
  }

  h1 {
    font-size: 2rem;
    font-weight: 700;
    margin-bottom: 1rem;
  }

  .main-btn {
    background-color: var(--primary-color);
    color: white;
    border: none;
    padding: 12px 20px;
    border-radius: var(--radius);
    font-size: 16px;
    cursor: pointer;
    margin-bottom: 2rem;
    transition: background-color 0.3s ease;
  }

  .main-btn:hover {
    background-color: #005ecb;
  }

  .info-text {
    font-size: 1rem;
    margin-bottom: 1rem;
    color: var(--text-color);
  }

  .message {
    font-weight: 600;
    margin-bottom: 1rem;
    padding: 12px 20px;
    border-radius: var(--radius);
    transition: all 0.3s ease;
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

  .card-container {
    display: flex;
    flex-wrap: wrap;
    gap: 20px;
    justify-content: center;
  }

  .card {
    background: var(--card-bg);
    border-radius: var(--radius);
    box-shadow: 0 4px 8px var(--card-shadow);
    padding: 20px;
    width: 220px;
    transition: transform 0.2s ease;
  }

  .card:hover {
    transform: translateY(-3px);
  }

  .card-link {
    text-decoration: none;
    color: var(--text-color);
  }

  .card h2 {
    font-size: 18px;
    margin-bottom: 1rem;
  }

  .card-actions {
    display: flex;
    justify-content: space-around;
    margin-top: 1rem;
  }

  .card-actions button {
    border: none;
    padding: 6px 10px;
    border-radius: 8px;
    font-size: 16px;
    background-color: #e5e5ea;
    cursor: pointer;
    transition: background 0.2s;
  }

  .card-actions button:hover {
    background-color: #d1d1d6;
  }

  @media (max-width: 600px) {
    .card {
      width: 100%;
    }

    .card-actions {
      flex-direction: column;
      gap: 8px;
    }
  }
</style>
