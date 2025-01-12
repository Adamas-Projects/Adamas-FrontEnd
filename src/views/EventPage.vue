<script>
import { cancelParticipation, formatEndDate, getEventFromID, sendProjectToApproval, subscribeToEvent, unsubscribeFromEvent } from '@/assets/scripts/event_scripts';
import SelectProjectModal from '@/components/modals/SelectProjectModal.vue';
import router from '@/router';
import { useAuthStore } from '@/stores/authentication';
import { format } from 'date-fns';
import { ptBR } from 'date-fns/locale';


export default{
    name: 'EventPage',

    components:{
        SelectProjectModal
    },

    data(){
        return{
            authStore: useAuthStore(),
            user: undefined,
            
            e_id: this.$route.params.id,
            event: undefined,
            subscribers_ID_array: [],

            data_inicial: undefined,
            data_final: undefined,

            showProjects: false,
            showParticipation: false,
            selected_project: undefined
        }
    },

    methods:{

        participation(){

            if (!this.authStore.isUserLogged){
                router.push({ path: '/tipo-de-conta' })
                return
            }
            if (this.authStore.getAccType == "institution"){
                alert("Instituições não podem se inscrever em eventos!")
                return
            }
            
            this.showParticipation = true

        },

        hasUserSubscribed(){

            if(!this.authStore.getToken){
                router.push({path: '/tipo-de-conta'})
                return
            }

            if (this.user && this.authStore.getAccType == 'common'){
                return this.subscribers_ID_array.includes(this.user.id)
            } else return false 
            
        },

        async inscrever(){

            if (!this.authStore.isUserLogged){
                router.push({ path: '/tipo-de-conta' })
                return
            }

            if (this.authStore.getAccType == "institution"){
                alert("Instituições não podem se inscrever em eventos!")
                return
            }

            try {
                subscribeToEvent(this.authStore.getToken, this.e_id)
                location.reload()
                
            } catch (error) {
                console.log(error)
            }
            
        },

        async desinscrever(){

            try {
                unsubscribeFromEvent(this.authStore.getToken, this.e_id)
                location.reload()

            } catch (error) {
                console.log(error)
            }
        },

        async mandarProjeto(){

            let u_token = this.authStore.getToken
            let p_id = parseInt(this.selected_project.project_id)

            sendProjectToApproval(
                u_token,
                p_id,
                this.e_id
            )
            this.showParticipation = false
        },

        async cancelarParticipacao(){
            let u_token = this.authStore.getToken
            let p_id = parseInt(this.selected_project.project_id)

            cancelParticipation(
                u_token,
                p_id,
                this.e_id
            )
            this.showParticipation = false
        }


    },



    async created(){
        await getEventFromID(this.e_id)
        .then(event => {
            this.event = event
        }).catch(error => console.log(error))

        this.user = this.authStore.getUser

        this.data_inicial = format(this.event.start_date, "d LLL 'às' HH:mm", {locale: ptBR})
        this.data_final = formatEndDate(this.event.start_date, this.event.end_date)

        if (this.event.subscribers){
            this.subscribers_ID_array = this.event.subscribers.map(subscriber => subscriber.id)
        }
    }

}

</script>

<template>

    <main v-if="this.event">
        
        <div class="container">
            <h1>{{ this.event.name }}</h1>

            <section class="event_info">
                <ul>
                    <li>
                        {{ this.event.institution_name }}
                    </li>

                    <li>
                        <p><span>Local:</span> {{ this.event.address }}</p>
                    </li>

                    <li>
                        <p><span>Data: </span> {{ this.data_inicial }} <span>até</span> {{ this.data_final }}</p>
                    </li>
                </ul>
            </section>

            <section class="event_content">
                <p>
                    {{ this.event.description }}
                </p>
            </section>

            <section class="buttons">

                <button v-if="hasUserSubscribed()" type="button" @click.prevent="desinscrever()">
                    Cancelar inscrição
                </button>
                <button v-else type="button" @click.prevent="inscrever()">
                    Inscrever-se
                </button>

                <button @click="this.participation()" type="button" v-if="!this.showParticipation">
                    Participar com projeto
                </button>
                <button @click="this.showParticipation = false" type="button" v-else-if="this.showParticipation == true">
                    Cancelar
                </button>

            </section>
        </div>

        <Transition name="participation">
            
            <div v-if="showParticipation == true" class="container">
                <h1>Participar com projeto</h1>
                <h3>Selecione um projeto:</h3>

                <div class="selection" v-if="this.selected_project" @click="showProjects = true">
                    <h3>{{ this.selected_project.title }}</h3>
                    <p>{{ this.selected_project.description }}</p>
                </div>
                
                <div class="selection" v-else @click="showProjects = true">
                    <h3>Selecionar projeto</h3>
                </div>

                <div class="buttons">
                    <button v-if="this.selected_project" type="button" @click="mandarProjeto()">
                        Participar
                    </button>
                    <button v-if="this.selected_project" type="button" @click="cancelarParticipacao(); this.showParticipation = false">
                        Cancelar participação
                    </button>
                </div>
                
            </div>

        </Transition>

        <Teleport to="body">
            <SelectProjectModal
            v-if="this.user"
            :userID="this.user.id"
            :token="this.authStore.getToken"
            :show="this.showProjects"
            @project="p => this.selected_project = p"
            @close="this.showProjects = false"
            />
        </Teleport>
        

    </main>

</template>

<style scoped>

main{
    display: flex;
    flex-direction: row;
    justify-content: space-around;
    padding: 10% 7% 7% 7%;
    min-height: 100vh;
    align-items: flex-start;
}

h1{
    margin-bottom: 2%;
}

.container{
    font-size: 1.25rem;
    width: 45%;
    height: fit-content;
    min-height: 45vh;
    padding: 2%;

    color: var(--Text2);
    background-color: var(--CardColor);

    border: 4px solid var(--ButtonColor);
    border-radius: 25px;

    display: flex;
    flex-direction: column;
}


/* Informações do evento */

.event_info ul{
    padding: 0;
    list-style: none;
    margin-bottom: 2%;
    color: var(--SubText);
    
}
.event_info span{
    font-weight: bold;
    color: var(--Text2);
}

.event_content{
    margin-bottom: 2%;
    line-break: anywhere;
}
.event_content p{
    height: fit-content;
}


/* Seleção de projetos */

.selection{
    margin: auto auto;

    background-color: var(--BackgroundColor);
    border: 4px solid var(--ButtonColor);
    border-radius: 25px;
    color: var(--TextHighlight);
    width: 100%;

    padding: 2% 0;
    height: fit-content;
    min-height: 10vh;
    text-align: center;
    
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
}
.selection{cursor: pointer;}



/* Botões */

.buttons{
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    margin: auto 0 0 0;
    width: 100%;
    height: 100%;
}
.buttons button{
    color: var(--Text2);
    font-weight: bold;
    font-size: 1.5rem;
    text-align: center;
    
    padding: 1%;
    width: 45%;
    height: auto;

    background-color: var(--ButtonColor);
    border: none;
    border-radius: 25px;
}
.buttons button:hover{
    background-color: var(--ButtonHoverColor);
    cursor: pointer;
}
@media screen and (max-width: 800px){
    .container{
        width: 80%;
    }
}
@media screen and (max-width: 600px){
    main{
        flex-direction: column;
        align-items: center;
        justify-content: center;
    }
    .container{
        margin-top: 10px;
        width: 100%;
    }
    h1{
        font-size: 1.5rem;
    }

}




.participation-enter-active {
  transition: all 0.3s ease-out;
}

.participation-leave-active {
  transition: all 0.3s cubic-bezier(1, 0.5, 0.8, 1);
}

.participation-enter-from,
.participation-leave-to {
  transform: translateX(20px);
  opacity: 0;
}



</style>