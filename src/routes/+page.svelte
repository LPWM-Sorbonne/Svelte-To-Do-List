<script>
  import { onMount } from "svelte";

  // Initialisation des variables
  let listes = [];
  let message = { texte: "", type: "" };

   // Au chargement de la page on recup les listes depuis le localStorage
   onMount(() => {
    const data = localStorage.getItem("todolists");
    if (data) {
      try {
        // Parse les données JSON et vérifie que c’est bien un tableau
        const fouillerData = JSON.parse(data);
        listes = Array.isArray(fouillerData) ? fouillerData : [];
      } catch (error) {
        // affiche l’erreur et vide les listes
        console.error("Erreur de recherche dans le localStorage :", error);
        listes = [];
      }
    }
  });

  // Sauvegarde toutes les listes dans le localstorage en ayant comme cle "todolists"
  function sauvegardeListes() {
    localStorage.setItem("todolists", JSON.stringify(listes));
  }

  //Fonction qui cree une nouvelle liste après avoir demande un nom a l'utilisateur
  function creationListe() {
    const nom = prompt("Nom de la nouvelle liste :");

    // Si le nom n'est pas vide on nettoie l'entree pour generer un id  sous la forme minuscule et tiret
    if (nom && nom.trim() !== "") {
      let id = nom.toLowerCase().replace(/\s+/g, "-");
      let suffixe = 1;
      let idUnique = id;

      // Verifie si l'id existe et si oui on ajoute un suffixe pour le rendre unique
      while (listes.some((liste) => liste.id === idUnique)) {
        idUnique = `${id}-${suffixe}`;
        suffixe++;
      }

      // On cree un objet liste avec un tableau vide pour les taches
      const nouvelleListe = {
        id: idUnique,
        nom: nom,
        tasks: [],
      };

      //Ajout de la nouvelle liste dans le tableau et ensuite on sauvegarde
      listes = [...listes, nouvelleListe];
      sauvegardeListes();

      // Affiche un message temporaire de succès
      afficherMessage("Liste créée avec succès !", "success");
    } else {
      return;
    }
  }

  // fonction qui modifie le nom et l'id d'une liste existante
  function modifierListe(liste) {
    const nouveauNom = prompt("Nouveau nom pour la liste :", liste.nom);
    if (nouveauNom && nouveauNom.trim() !== "") {
      // generation d'un nouvel id en fonction du nouveau nom
      const nouveauId = nouveauNom.toLowerCase().replace(/\s+/g, "-");
      let suffixe = 1;
      let idUnique = nouveauId;

      // check que le nouvel id est unique
      while (listes.some((l) => l.id === idUnique && l !== liste)) {
        idUnique = `${nouveauId}-${suffixe++}`;
      }

      // Met a jour les valeurs dans la liste
      liste.nom = nouveauNom;
      liste.id = idUnique;

      // Force la reactivite de Svelte
      listes = [...listes];
      sauvegardeListes();

      // Message temporaire
      afficherMessage("Liste renommé avec succès !", "success");
    } else {
      return;
    }
  }

  // fonction qui duplique une liste et ses taches associe
  function dupliquerListe(liste) {
    const copieNom = prompt("Nom pour la copie :", liste.nom + " (copie)");
    if (copieNom && copieNom.trim() !== "") {
      // generation d'un nouvel id en fonction du nom de copie
      let idDeBase = copieNom.toLowerCase().replace(/\s+/g, "-");
      let suffixe = 1;
      let idUnique = idDeBase;

      // verifie que l'id de la copie n'existe pas
      while (listes.some((l) => l.id === idUnique)) {
        idUnique = `${idDeBase}-${suffixe++}`;
      }

      // autre fonction qui permet de  dupliquer les taches et sous tache de facon recursive
      function dupliquerTaches(taches) {
        return taches.map((tache) => ({
          ...tache,
          // creation du nouvelle id en fonction de la date
          id: Date.now() + Math.random(), 
          // récursivite de la duplication des taches
          children: dupliquerTaches(tache.children || []), 
        }));
      }

      // Créer la nouvelle liste avec les tâches dupliquées
      const nouvelleListe = {
        id: idUnique,
        nom: copieNom,
        tasks: dupliquerTaches(liste.tasks),
      };

      // Force la reactivite de Svelte
      listes = [...listes, nouvelleListe];
      sauvegardeListes();

      // Message temporaire
      afficherMessage("Liste dupliqué avec succès !", "success");
    } else {
      return;
    }
  }

  // Supprime une liste après confirmation
  function supprimerListe(id) {
    if (confirm("Souhaitez-vous supprimer cette liste ?")) {
      // Filtre la liste pour exclure celle avec l'id
      listes = listes.filter((liste) => liste.id !== id);
      sauvegardeListes();

      //Message temporaire
      afficherMessage("Liste supprimer avec succès !", "success");
    } else {
      return;
    }
  }

  // Fonction permettant d'afficher des messages de façon modulaire
  function afficherMessage(texte, type = "success", duree = 4000) {
    message = { texte: texte, type };
    setTimeout(() => (message = { texte: "", type: "" }), duree);
  }
</script>

<div class="container">
  <h1>📋 Mes To‑Do Lists</h1>

  <button class="main-btn" on:click={creationListe}>➕ Créer une liste</button>

  {#if message.texte}
    <div class="message" class:success={message.type === "success"} class:error={message.type === "error"}>
      {message.texte}
    </div>
  {/if}

  {#if listes.length === 0}
    <p class="info-text"> Aucune liste pour le moment. Commencez par en créer une. </p>
  {:else}
    <p class="info-text"> Appuyez sur une liste pour y accéder ou bien utilisez les actions ✏️ 🔁 🗑️ </p>
  {/if}

  <div class="card-container">
    {#each listes as liste (liste.id)}
      <div class="card" >
        <a href={`/liste/${liste.id}`} class="card-link">
          <h2 class="titre_text">{liste.nom}</h2>
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

  .titre_text:hover{
    cursor: pointer;
    text-decoration: underline;
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
