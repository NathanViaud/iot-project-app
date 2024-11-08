<script setup lang="ts">
import Paho from 'paho-mqtt';
import useConfetti from '~/composables/confetti';

const distance = ref('N/A');
const collisions = ref('N/A');
const ledState = ref('N/A');
let client: any = null;

const broker = 'broker.hivemq.com';
const port = 8000;
const topic = 'bat-radar/distance';


function connectToMQTT() {
    const clientId = `vueClient-${Math.random().toString(16).substring(2, 8)}`;
    client = new Paho.Client(broker, port, clientId);

    client.onConnectionLost = onConnectionLost;
    client.onMessageArrived = onMessageArrived;

    client.connect({
        onSuccess: () => {
            console.log('Connected to MQTT');
            client.subscribe(topic);
        },
        onFailure: (error: any) => {
            console.error('Failed to connect to MQTT');
        }
    })
}

function onConnectionLost(responseObject: any) {
    if (responseObject.errorCode !== 0) {
        console.error('Connection lost:', responseObject.errorMessage);
    }
}

function onMessageArrived(message: any) {
    console.log('Message arrived:', message.payloadString);
    try {
        const data = JSON.parse(message.payloadString);
        distance.value = data.distance;
        collisions.value = data.collisions;
        ledState.value = data.ledState;
    } catch (e) {
        console.error('Error parsing message:', e);
    }
}

onMounted(() => {
    connectToMQTT();
})

onBeforeUnmount(() => {
    client.disconnect();
    console.log('Disconnected from MQTT');
})

const progress = computed(() => 100 - (Number(distance.value) / 150 * 100));
const color = computed(() => {
    switch (true) {
        case Number(distance.value) < 25: return 'red'
        case Number(distance.value) < 50: return 'amber'
        default: return 'emerald'
    }
})

// watch if collision is changed
watch(collisions, () => {
    useConfetti({ particle: 200, spread: 360, origin: { y: 0.3 } });
})
</script>

<template>
    <div class="p-4 text-center flex flex-col gap-5">
        <h1 class="text-4xl font-bold">Bat Radar Dashboard</h1>
        <UCard>
            <h2 class="text-xl font-semibold mb-4">Distance: {{ distance }} cm</h2>
            <UProgress :value="progress" :color="color" />
        </UCard>

        <UCard>
            <h2 class="text-xl font-semibold">Collisions: {{ collisions }}</h2>
        </UCard>
    </div>
</template>
