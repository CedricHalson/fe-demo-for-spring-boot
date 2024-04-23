<template>
    <h1>Connection status: {{ data.connectionStatus }}</h1>
    <button @click="onButtonClick">{{ data.buttonText }}</button>
</template>

<script setup lang="ts">
import { reactive } from "vue"
import { Client } from "@stomp/stompjs"

const data = reactive({
    connectionStatus: "Disabled",
    buttonText: "connect",
})

const stompClient = new Client({
    brokerURL: "",
    connectHeaders: {
        // Authorization: "Bearer",
    },
    onConnect: () => {
        data.connectionStatus = "Connected"
    },
    debug: (msg) => {
        console.debug("STOMP DEBUG", msg)
    },
    onWebSocketError: (e) => {
        console.info("WEBSOCKET ERROR", e)
        stompClient.deactivate()
    },
})

function onButtonClick() {
    if (data.buttonText === "connect") {
        stompClient.activate()
    } else {
        stompClient.deactivate()
    }
}
</script>
