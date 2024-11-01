<script setup>
    defineProps(["active", "color", "name", "selectedMove"]);
</script>

<template>
    <div id="player" :style="{'--color': color}">
        <div id="profile">
            <img src="../assets/guest.png" :class="{active: active}">
            <h1 :class="{active: active}">{{ name }}</h1>
        </div>
        <div id="moveSelection">
            <button id="drop" @click="$emit('selectMove', 'drop')" :class="{selected: selectedMove == 'drop'}"></button>
            <button id="rotate" @click="$emit('selectMove', 'rotate')" :class="{selected: selectedMove == 'rotate'}"></button>
            <button id="revolve" @click="$emit('selectMove', 'revolve')" :class="{selected: selectedMove == 'revolve'}"></button>
            <button id="shift" @click="$emit('selectMove', 'shift')" :class="{selected: selectedMove == 'shift'}"></button>
        </div>
    </div>
</template>

<style scoped>
    div {
        display: flex;
        align-items: center;
    }
    #player {
        justify-content: space-between;
        width: calc(100% - 1rem);
        padding: 0.5rem;
    }
    img {
        width: 3rem;
        height: 3rem;
        box-shadow: var(--color);
        transition: box-shadow 0.5s;
    }
    img.active {
        box-shadow: 0 0 0.25rem 0.25rem var(--color);
    }
    h1 {
        margin: 0;
        margin-left: 1rem;
        transition: color 0.5s;
    }
    h1.active {
        color: var(--color);
    }
    #moveSelection {
        width: fit-content;
        gap: 1rem;
    }   
    button {
        width: 3rem;
        height: 3rem;
        background-color: white;
        background-size: contain;
        transition: background-color 0.5s;
        border-radius: 50%;
        border: 0.3rem solid rgba(0, 0, 0, 0.25);
        color: black;
        cursor: pointer;
    }
    #drop {
        background-image: url(../assets/drop.png);
    }
    #rotate {
        background-image: url(../assets/rotate.png);
    }
    #revolve {
        background-image: url(../assets/revolve.png);
    }
    #shift {
        background-image: url(../assets/shift.png);
    }
    .selected {
        background-color: var(--color);
    }
</style>