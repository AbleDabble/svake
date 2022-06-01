<script>
    export let type = 'empty';
</script>

<div class={type}>

</div>


<style>
    div {
        height: 50px;
        width: 50px;
        margin: 0.2em;
    }
    .empty {
        background-color: rgb(160, 150, 150);
    }
    
    .snake {
        background-color: rgba(7, 87, 34, 0.74);
    }

    .food {
        background-color: chocolate;
    }

</style>