<template>
    <div class="edit-actor">
        <h2>Modifier :</h2>
        <form class="form" @submit.prevent="submitForm">
            <label for="firstname">Prénom :</label>
            <input type="text" v-model="actor.firstname" />

            <label for="lastname">Nom :<span>*</span></label>
            <input type="text" v-model="actor.lastname" required />

            <label for="media">Media :</label>
            <input type="text" v-model="actor.media" placeholder="URL de l'image" />

            <label for="dob">Date de naissance :<span>*</span></label>
            <input type="date" v-model="actor.dob" required />

            <label for="deathDate">Date de décès :</label>
            <input type="date" v-model="actor.deathDate" />

            <label for="nationality">Pays :<span>*</span></label>
            <input type="text" v-model="actor.nationality" required />

            <label for="gender">Sexe :</label>
            <select v-model="actor.gender" required>
                <option value="M">M</option>
                <option value="F">F</option>
            </select>

            <label for="bio">Biographie :</label>
            <textarea v-model="actor.bio"></textarea>

            <label for="awards">Récompenses :</label>
            <input type="number" v-model="actor.awards" min="0" max="10" />

            <p>*ces champs doivent à tout prix être remplis</p>

            <div class="form__buttons">
                <button type="submit">Modifier</button>
                <button type="button" @click="$emit('cancel')">Annuler</button>
            </div>
        </form>
    </div>
</template>

<script>
import axios from 'axios';

export default {
    name: 'EditActor',
    props: {
        actor: {
            type: Object,
            required: true
        }
    },
    methods: {
        getToken() {
            return localStorage.getItem('jwt_token'); // Récupérer le token depuis le localStorage
        },
        redirectToLogin() {
            this.$router.push('/login'); // Redirection vers la page de login si pas de token
        },
        getCurrentDate() {
            const today = new Date();
            const yyyy = today.getFullYear();
            const mm = String(today.getMonth() + 1).padStart(2, '0');
            const dd = String(today.getDate()).padStart(2, '0');
            return `${yyyy}-${mm}-${dd}`;
        },
        async submitForm() {
            const actorId = this.actor.id;
            this.actor.updatedAt = this.getCurrentDate();

            const token = this.getToken();
            if (!token) {
                this.redirectToLogin(); // Redirection si pas de token
                return;
            }

            try {
                await axios.put(`http://symfony.mmi-troyes.fr:8319/api/actors/${actorId}`, this.actor, {
                    headers: {
                        'Authorization': `Bearer ${token}` // Ajout du token JWT dans les en-têtes
                    }
                });
                alert('Les détails de l\'acteur ont été mis à jour avec succès.');
                this.$emit('actor-updated', this.actor); // Émet l'événement pour notifier le parent
            } catch (error) {
                console.error('Erreur lors de la mise à jour de l\'acteur :', error);
                if (error.response && error.response.status === 401) {
                    this.redirectToLogin(); // Redirection si l'authentification échoue
                } else {
                    alert('Erreur lors de la mise à jour de l\'acteur.');
                }
            }
        }
    }
};
</script>
