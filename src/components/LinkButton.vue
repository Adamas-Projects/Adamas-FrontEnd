<script>
    export default {
        name: "LinkButton",
        props: {
            link: String,
            icon: String,
        }
    }

</script>

<template>
    
   <RouterLink :to="link">
            <img v-if="icon != null" :src="icon" alt="Ícone">
            <slot></slot>
   </RouterLink> 

</template>

<style scoped>

a{
    display: flex;
    flex-direction: row;

    justify-items: center;
    justify-content: center;
    align-items: center;
    align-content: center;

    background-color: var(--ButtonColor);
    padding: 2% ;
    border-radius: 15px;
    text-decoration: none;
    font-size: 2rem;
    color: var(--Text2);
    text-align: center;
}

a:hover{ background-color: var(--ButtonHoverColor);}

a:visited{color: var(--Text2);}

img{
    width: 50px;
    margin-right: 2%;
}

@media screen and (max-width: 600px){
    img{
        margin: 5px;
        width: 8vw;
    }
}

</style>