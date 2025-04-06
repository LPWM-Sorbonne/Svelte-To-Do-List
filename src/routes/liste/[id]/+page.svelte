<script>
  // Import utilisé pour le fonctionnement de la page
  import { page } from "$app/stores";
  import TaskItem from "$lib/components/TaskItem.svelte";
  import { onMount } from "svelte";

  // Initialisation des variables
  let idListe = "";
  let titreListe = "";
  let tasks = [];
  let SelectionnerTacheIDs = new Set();
  let dataUtilisable = false;
  let message = { texte: "", type: "" };

  // Fonction qui retourne la cle de stockage de la liste actuelle
  function getStorageKey() {
    return `todolist-${idListe}`;
  }

   // Au chargement de la page on recup les informations depuis le localStorage
  onMount(() => {
    // On s’abonne aux parmaetre de l’URL pour recuperer l’id de la liste
    const unsubscribe = page.subscribe(({ params }) => {
      if (params.id && params.id.trim() !== "") {
        idListe = params.id;

        // On verifie que le localStorage est bien disponible
        if (typeof localStorage !== "undefined") {
          const data = localStorage.getItem("todolists");

          try {
            // Parse les donnee JSON et cherche la liste par ID
            const fouillerData = JSON.parse(data);
            const trouverListe = fouillerData.find((liste) => liste.id === idListe); 

            // Si une liste est trouvé on charge les tâches de la liste
            if (trouverListe) {
              titreListe = trouverListe.nom;
              tasks = trouverListe.tasks;
            }
            // Active l’affichage
            dataUtilisable = true;

          } catch (error) {
            // affiche l’erreur et vide les listes
            console.error("Erreur JSON corrompu :", error);
            tasks = [];
          }
        }
      }
    });
// On retourne l’unsubscribe pour nettoyer l’abonnement 
    return unsubscribe;
  });

  // Fonction pour sauvegarder les taches dans le localStorage
  function sauvegardeTaches() {
    // on check si toutes les ddonee qu'on a besoin sont disponibles
    if (!dataUtilisable || !idListe || typeof localStorage === "undefined") return;

    try {
      const data = JSON.parse(localStorage.getItem("todolists"));

      // Fonction pour mettre a jour les taches d'une liste par son id
      function ListeModifier(data, idListe, tasks) {
        return data.map((list) => list.id === idListe ? { ...list, tasks } : list);
      }

      // sauvegarde les taches dans le localstorage
      localStorage.setItem("todolists", JSON.stringify(ListeModifier(data, idListe, tasks)));
    } catch (e) {
      console.error("Erreur de sauvegarde localStorage :", e);
    }
  }

  // Fonction pour rechercher une tache par son id dans une liste
  function TrouverTacheParID(liste, id) {
    for (let tache of liste) {
      if (tache.id === id) return tache;
      // Recherche recursive 
      const trouver = TrouverTacheParID(tache.children, id);
      if (trouver) return trouver;
    }
    // Si aucune tache trouver on retourne null
    return null;
  }

   // Fonction pour rechercher un parent par une tache
  function TrouverParentParTache(liste, id, parent = null) {
    for (let tache of liste) {
      if (tache.id === id) return parent;
      // Recherche recursive 
      const trouver = TrouverParentParTache(tache.children, id, tache);
      if (trouver) return trouver;
    }
    // Si aucun parent trouver on retourne null
    return null;
  }

  // Fonction pour supprimer une tache par son id
  function SupprimeTacheParID(liste, ids) {
    return liste.filter((tache) => {
       // Si le id est dans 'ids' alors on le supprime 
      if (ids.has(tache.id)) return false;
      // sinon on verfie les sous  taches
      tache.children = SupprimeTacheParID(tache.children, ids);
      return true;
    });
  }

  // fonction pour dupliquer une tache
  function DupliquerTache(tache) {
    return {
      // Copie les prop de la tache de base
      ...tache,
      id: Date.now() + Math.random(),
      text: tache.text + " (copie)",
      // Duplique les sous taches
      children: tache.children.map(DupliquerTache),
    };
  }

  // Fonction pour changer l'etat de validation d'une tache
  function ChangerEtat(tache) {
    // Inverse l'état de la tache
    tache.completed = !tache.completed;
    if (SelectionnerTacheIDs.has(tache.id)) {
      SelectionnerTacheIDs.delete(tache.id);
    } else {
      SelectionnerTacheIDs.add(tache.id);
    }
    // cree un nouvel ensemble
    SelectionnerTacheIDs = new Set(SelectionnerTacheIDs);

    // Force la reactivite de Svelte
    tasks = [...tasks]; 
    sauvegardeTaches();
  }

  //fonction pour  selecctionner une tache par id
  function TacheSelectionner(id) {
    // Si la tache est selectionne, on la deselectionne
    if (SelectionnerTacheIDs.has(id)) {
      SelectionnerTacheIDs.delete(id);
    // Sinon, on la sélectionne
    } else {
      SelectionnerTacheIDs.add(id);
    }
    // cree un nouvel ensemble
    SelectionnerTacheIDs = new Set(SelectionnerTacheIDs);
  }

  // Fonction pour ajouter une sous tache
  function CreeSousCategorie(tache) {
    const sousTache = {
      id: Date.now(),
      text: "Nouvelle sous-tâche",
      completed: false,
      children: [],
    };

    // Force la reactivite de Svelte
    tache.children = [...tache.children, sousTache];
    tasks = [...tasks]; 
    sauvegardeTaches();
  }

  // Fonction permettant d'afficher des messages de façon modulaire
  function afficherMessage(texte, type = "success", duree = 4000) {
    message = { texte: texte, type };
    setTimeout(() => (message = { texte: "", type: "" }), duree);
  }

  // Fonction pour ajouter une nouvelle tache a la liste
  function Ajout() {
    const task = {
      id: Date.now(),
      completed: false,
      children: [],
    };

    const text = prompt("Modifiez le texte de la tâche :", task.text);
    // Vérifier si l'utilisateur a saisi un texte ou annulé
    if (text === null) {
      // L'utilisateur a annulé, on ne crée pas la tache
      return;
    }

    // Si l'utilisateur a saisi quelque chose, on met à jour la tache
    if (text.trim() !== "") {
      task.text = text;
    } else {
      // Si le texte est vide, on dedini un texte par defaut
      task.text = "Tâche sans nom";
    }

    // Si des taches sont selectionner, on ajoute la nouvelle tache comme sous-tache
    if (SelectionnerTacheIDs.size) {
      for (const id of SelectionnerTacheIDs) {
        const parent = TrouverTacheParID(tasks, id);
        if (parent) {
          parent.children = [...parent.children, task];
          afficherMessage("La tache est bien crée.", "success");

        }
      }
    } else {
      tasks = [...tasks, task];
      afficherMessage("La tache est bien crée.", "success");

    }

    // Force la reactivite de Svelte
    tasks = [...tasks];
    sauvegardeTaches();
  }

  // Fonction pour modifier une tache selectionner
  function Modification() {
    // Si plus d'une tache est selectionner on peut pas modifier
    if (SelectionnerTacheIDs.size !== 1) {
      alert("Sélectionne une seule tâche.");
      return;
    }
    const id = [...SelectionnerTacheIDs][0];
    const tache = TrouverTacheParID(tasks, id);
    if (tache) {
      // Demande à l'utilisateur de modifier le texte
      const text = prompt("Modifier :", tache.text);
      if (text) {
        tache.text = text;

        // Force la reactivite de Svelte
        tasks = [...tasks]; 
        sauvegardeTaches();
        afficherMessage("La tache est bien modifier.", "success");

      }
    }
  }


  function Duplication() {
    // Vérifier si plus d'une tâche est sélectionnée
    if (SelectionnerTacheIDs.size > 1) {
      afficherMessage("Veuillez sélectionner une seule tache à la fois pour la dupliquer.", "error");
      return;
    }

    const copies = [];

    // Pour chaque tache selectionner on la duplique
    for (const id of SelectionnerTacheIDs) {
      const tache = TrouverTacheParID(tasks, id);
      const parent = TrouverParentParTache(tasks, id);
      if (tache) {
        // cree une copie de la tache
        const clone = DupliquerTache(tache);
        if (parent) {
          // Ajoute la copie comme sous tache du parent
          parent.children = [...parent.children, clone];
          afficherMessage("La tache est bien dupliqué.", "success");
        } else {
          // sinon on ajoute la copie a la liste principale
          copies.push(clone);
          afficherMessage("La tache est bien dupliqué.", "success");

        }
      }
    }

    // Si des copies sont djea cree on les ajoute a la liste des taches
    if (copies.length) {
      tasks = [...tasks, ...copies];
    }

    // Vide l'ensemble
    SelectionnerTacheIDs = new Set();

    // Force la reactivite de Svelte
    tasks = [...tasks]; 
    sauvegardeTaches();
  }


  function Supression() {
    // supprimer une tache
    tasks = SupprimeTacheParID(tasks, SelectionnerTacheIDs);
    SelectionnerTacheIDs = new Set();

    // Force la reactivite de Svelte
    tasks = [...tasks]; 
    sauvegardeTaches();
    afficherMessage("La tache est bien supprimé.", "success");
  }


  

  
</script>

<div class="container">
  <a href="/">← Retour</a>
  <h1>{titreListe}</h1>

  <div class="buttons">
    <button on:click={Ajout} disabled={!dataUtilisable}>Ajouter</button>
    <button
      on:click={Modification}
      disabled={SelectionnerTacheIDs.size !== 1 || !dataUtilisable}>Modifier</button
    >
    <button
      on:click={Duplication}
      disabled={SelectionnerTacheIDs.size === 0 || !dataUtilisable}>Dupliquer</button
    >
    <button
      on:click={Supression}
      disabled={SelectionnerTacheIDs.size === 0 || !dataUtilisable}>Supprimer</button
    >
  </div>

  {#if message.texte}
    <div class="message" class:success={message.type === "success"} class:error={message.type === "error"}>
      {message.texte}
    </div>
  {/if}

  <div class="task-list">
    {#if dataUtilisable}
      {#each tasks as task (task.id)}
        <TaskItem {task} level={0} {SelectionnerTacheIDs} {TacheSelectionner} {ChangerEtat} {CreeSousCategorie}/>
      {/each}
    {/if}
  </div>
</div>

<style>
  :root {
    --primary-color: #007aff;
    --background-color: #f2f2f7;
    --text-color: #1c1c1e;
    --secondary-text-color: #636366;
  }

  .container {
    padding: 16px;
    max-width: 850px;
    margin: auto;
  }

  .buttons {
    display: flex;
    gap: 8px;
    margin-bottom: 16px;
    flex-wrap: wrap;
    justify-content: center;
  }

  .message {
    font-weight: 600;
    margin-bottom: 1rem;
    padding: 12px 20px;
    border-radius: 15px;
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

  .task-list {
    background: #fff;
    border-radius: 12px;
    padding: 16px;
    border: 1px solid #ddd;
    margin: auto;
    align-items: center;
    max-width: 100%;
    overflow-x: scroll;
  }

  button {
    background-color: var(--primary-color);
    color: white;
    border: none;
    border-radius: 8px;
    padding: 10px 16px;
    font-size: 16px;
    cursor: pointer;
    transition: background-color 0.3s ease;
  }

  button:hover {
    background-color: #005ecb;
  }

  button:disabled {
    background-color: #d1d1d6;
    cursor: not-allowed;
  }

  h1 {
    font-size: 1.8rem;
    margin-bottom: 1rem;
    text-align: center;
  }

  a {
    display: block;
    margin-bottom: 1rem;
    color: var(--primary-color);
    text-decoration: none;
  }

  @media (max-width: 768px) {
    .buttons {
      flex-direction: column;
    }

    button {
      width: 100%;
    }
  }
</style>
