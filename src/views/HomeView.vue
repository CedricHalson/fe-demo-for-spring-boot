<template>
    <div class="home-page">
        <header>
            <section class="home-page__status">
                Status: <span :class="statusColor">{{ statusText }}</span>
            </section>
            <section class="home-page__actions">
                <button
                    v-if="!data.connectionStatus"
                    @click="onConnect"
                >
                    Connect
                </button>
                <button
                    v-else
                    @click="onDisconnect"
                >
                    Disconnect
                </button>
            </section>
        </header>
        <main>
            <form @submit.prevent="onSubmit">
                <section>
                    <label>
                        <span>IP</span>
                        <input
                            type="text"
                            v-model="data.ip"
                        />
                    </label>
                    <label>
                        <span>User ID</span>
                        <input
                            type="text"
                            v-model="data.userId"
                        />
                    </label>
                    <label>
                        <span>Latitude</span>
                        <input
                            type="text"
                            v-model="data.latitude"
                        />
                    </label>
                    <label>
                        <span>Longitude</span>
                        <input
                            type="text"
                            v-model="data.longitude"
                        />
                    </label>
                    <label>
                        <span>Time Stamp</span>
                        <input
                            type="text"
                            v-model="data.timeStamp"
                        />
                    </label>
                </section>
                <button
                    v-if="data.connectionStatus"
                    type="submit"
                >
                    Send
                </button>
            </form>
            <h3>Input Data</h3>
            <pre>{{ requestData }}</pre>
            <h3>Output Data</h3>
            <pre>{{ data.messageList }}</pre>
        </main>
    </div>
</template>

<script setup lang="ts">
import { reactive, computed } from "vue"
import { Client, type StompSubscription } from "@stomp/stompjs"

const data = reactive({
    connectionStatus: false,

    ip: "",
    latitude: "",
    longitude: "",
    timeStamp: "",
    userId: "",

    subscription: {} as StompSubscription,

    messageList: [] as Array<string>,
})

const statusText = computed(() => {
    if (data.connectionStatus) {
        return "Connected"
    }
    return "Offline"
})

const statusColor = computed(() => ({
    "status-success": data.connectionStatus,
    "status-error": !data.connectionStatus,
}))

const requestData = computed(() =>
    JSON.stringify(
        {
            ip_address: data.ip,
            latitude: data.latitude,
            longitude: data.longitude,
            time_stamp: data.timeStamp,
            user_id: data.userId,
        },
        undefined,
        4
    )
)

const stompClient = new Client({
    brokerURL: "",
    connectHeaders: {
        // Authorization: "Bearer",
    },
    onConnect: () => {
        data.connectionStatus = true

        data.subscription = stompClient.subscribe("/topic/greetings", (msg) => {
            data.messageList.push(JSON.stringify(JSON.parse(msg.body), undefined, 4))
        })
    },
    onDisconnect: () => {
        data.connectionStatus = false
        stompClient.unsubscribe(data.subscription.id)
    },
    debug: (msg) => {
        console.debug("STOMP DEBUG", msg)
    },
    onWebSocketError: (e) => {
        console.info("WEBSOCKET ERROR", e)
        data.connectionStatus = false
        stompClient.deactivate()
    },
})

function onConnect() {
    stompClient.activate()
}

function onDisconnect() {
    stompClient.deactivate()
}

function onSubmit() {
    stompClient.publish({
        destination: "/update-location",
        body: requestData.value,
        headers: {},
    })
}
</script>

<style lang="sass">
.home-page
    padding: 0 24px

    .status-success
        color: var(--success)

    .status-error
        color: var(--error)

    header
        display: flex
        align-items: center
        justify-content: space-between

        padding: 12px 0

        font-weight: 600

    button
        border: 0
        background: var(--primary)
        color: white
        font-size: 16px

        padding: 8px
        border-radius: 4px
        cursor: pointer

    form
        margin: 12px 0

        section
            display: grid
            gap: 12px
            grid-template-columns: repeat(2, 1fr)
            margin-bottom: 12px

    label
        display: grid

        span
            margin-bottom: 4px

    input
        padding: 12px

    pre
        padding: 0
        margin: 0
        background: lightslategray
        padding: 12px
        color: linen
</style>
