# Description du projet
Ce dépôt contient les sources d'un site statique qui permet à des utilisateurs authentifiés d'accèder à un interface permttant de démarrer à distance des machines via un mécanisme de Wake On Lan (WOL).
## Expérience utilisateur
Le site contient un formulaire qui demande aux utilisateurs un mail et un mot de passe. 

<img width="500" alt="image" src="https://github.com/user-attachments/assets/695af3f6-d105-481e-83db-c08e1204dc62" />

Une fois entré, l'utilisateur a accès à la liste des machines qui sont paramétrées pour recevoir des Magic Packet

<img width="500" alt="image" src="https://github.com/user-attachments/assets/434c1a78-ff0c-4b70-8777-0541ecfd8b0c" />

- Si la requête fonctionne, le bouton se désactive puis affiche "Envoyé" pendant 30 secondes puis revient à son état initial
- Si la requête échoue, le bouton se déactive, puis affiche "Échec" pendant 10 sec puis revient à son état initial

## Communication avec la base de donnnées
Les demandes d'activations ainsi que les données des machines à réveiller son stockées dans une base de données hébergée chez Supabase.
Supabase fournit à la fois le stockage des tables Postgresql, ainsi que l'API accessible via Javascript.

### Bibliothèque javascript
La bibliothèque permettant au site de communiquer avec la base de données est appelée ici 
```
      import { createClient } from 'https://cdn.jsdelivr.net/npm/@supabase/supabase-js/+esm';
```
### Authentification
Puis l'authentification est gérée ici
```
      const supabase = createClient(
        'https://bmzijxtvhjbugcdkfswe.supabase.co',
        'sb_publishable_n13ojVtT79rG2WbTEq9S8w_zlOlH3H4'
      );

      window.login = async function () {
        const email = document.getElementById('email').value;
        const password = document.getElementById('password').value;

        const { data, error } = await supabase.auth.signInWithPassword({
          email,
          password
        });
```
### Affichage des données
Les machines sont énumérées
```
document.getElementById('login').style.display = 'none';
document.getElementById('machines').style.display = 'grid';

        await loadMachines();
      };

      async function loadMachines() {
        const { data, error } = await supabase.from('machines').select('*');

        if (error) {
          alert(error.message);
          return;
        }

        data.forEach((machine) => {
          const item = document.createElement('div');
          item.className = 'machine-item';
          item.innerHTML = `
            <strong>${machine.name}</strong>
            <button class="button machine-button" type="button" onclick="wake('${machine.id}', event)">
              Allumer
            </button>
          `;
          list.appendChild(item);
        });
      }
```
### Envoi des requêtes vers la base de donnée
Puis en fonction du clic de l'utilisateur, une requête WOL est émise vers Supabase pour passer le status de la requête en 'pending'.
```
      window.wake = async function (id, event) {
        const button = event?.target;

        if (button) {
          button.disabled = true;
          button.textContent = 'Envoi...';
        }

        const { error } = await supabase.from('wol_requests').insert({
          machine_id: id,
          status: 'pending'
        });
```
Charge ensuite au backend de récupérer cette ligne crée par l'utilisateur sur supabase, d'envoyer le magic packet sur le réseau, puis de mettre à jour le status de démarrage de la machine sur Supabase.
