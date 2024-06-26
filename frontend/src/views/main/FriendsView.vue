<template>
    <div class="container">
        <div class="requests" v-if="requests.length">
            <h3>Friend Requests</h3>
            <div class="request" v-for="(request, index) in requests" :key="index">
                <router-link class="profpic" :to="`/profile/${request.user_name}`">
                    <img :src="request.profile_picture" alt="">
                </router-link>
                <p>
                    <router-link :to="`/profile/${request.user_name}`">
                        {{ request.first_name }} {{ request.last_name }}
                    </router-link>
                    has sent a friend request
                </p>
                <button class="btn accept" @click="acceptRequest(request.requester_id)">Accept</button>
                <button class="btn reject" @click="rejectRequest(request.requester_id)">Delete</button>
            </div>
        </div>
        <h3>Find your friends on <span>Peralink</span></h3>
        <div class="search">
            <input type="text" placeholder="search people" v-model="search">
        </div>
        <div class="deck">
            <div class="card" v-for="(user, index) in users" :key="index">
                <router-link :to="`profile/${user.user_name}`">
                    <img :src="user.profile_picture" alt="" class="prof-pic">
                    <h4>{{ user.first_name }} {{ user.last_name }}</h4>
                    <p>@{{ user.user_name }}</p>
                </router-link>
                <button class="request" v-if="!user.is_friend && !user.sent_request" @click="sendRequest(user.u_id)">Send
                    Request</button>
                <button class="request cancel" v-else-if="!user.is_friend && user.sent_request"
                    @click="cancelRequest(user.u_id, `${user.first_name} ${user.last_name}`)">Cancel Request</button>
                <button class="reject" v-else
                    @click="unfriend(user.u_id, `${user.first_name} ${user.last_name}`)">Unfriend</button>
            </div>
        </div>
    </div>
</template>

<script setup>
import axios from "axios";
import { onMounted, ref, watch } from "vue";
import { useStore } from "vuex";

const search = ref("")
const loading = ref(false)
const store = useStore()

const users = ref([])
const requests = ref([])

watch(search, async () => {
    const list = await getUsers(search.value.length > 0 ? search.value : "%")
    users.value = list
})

const getUsers = async (query) => {
    try {
        const response = await axios.get("/search_global/", {
            params: {
                q: query,
            }
        })
        return response.data
    }
    catch (err) {
        store.commit("addError", err.response.data.error)
    }
}

const getRequest = async () => {
    try {
        const res = await axios.get("/friend_request/")
        return res.data
    }
    catch (err) {
        store.commit("addError", err.response.data.error)
    }
}

const sendRequest = async (id) => {
    try {
        await axios.post("/friend_request/send", {
            friend_id: id
        })
        store.commit("addSuccess", "Request sent")
        users.value = await getUsers(search.value.length > 0 ? search.value : "%")
    }
    catch (err) {
        store.commit("addError", err.response.data.error)
    }
}

const cancelRequest = async (id) => {
    try {
        await axios.delete("/friend_request/cancel", {
            data: { friend_id: id }
        })
        store.commit("addSuccess", "Request canceled")
        users.value = await getUsers(search.value.length > 0 ? search.value : "%")
    }
    catch (err) {
        store.commit("addError", err.response.data.error)
    }
}

const acceptRequest = async (id, name) => {
    try {
        await axios.post("/friend_request/accept", {
            friend_id: id
        })
        store.commit("addSuccess", `You and ${name} friends`)
        users.value = await getUsers(search.value.length > 0 ? search.value : "%")
        requests.value = await getRequest()
    }
    catch (err) {
        store.commit("addError", err.response.data.error)
    }
}

const rejectRequest = async (id) => {
    try {
        await axios.delete("/friend_request/reject", {
            data: { friend_id: id }
        })
        requests.value = await getRequest()
    }
    catch (err) {
        store.commit("addError", err.response.data.error)
    }
}

const unfriend = async (id, name) => {
    try {
        await axios.delete("/friend_request/unfriend", {
            data: { friend_id: id }
        })
        store.commit("addSuccess", `You and ${name} no longer friends`)
        users.value = await getUsers(search.value.length > 0 ? search.value : "%")
    }
    catch (err) {
        store.commit("addError", err.response.data.error)
    }
}

onMounted(async () => {
    users.value = await getUsers("%")
    requests.value = await getRequest()
})
</script>

<style scoped>
h3 {
    margin: 3rem 0 2rem;
    font-size: 1.5rem;
    font-weight: 500;
}

h3 span {
    color: #2FA634;
    font-family: 'Raleway', sans-serif;
    font-weight: 600;
}

.search {
    display: block;
    max-width: 500px;
    width: 100%;
    margin: auto;
    margin: 1rem auto;
}

.search input {
    width: 100%;
}

.deck {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
    gap: 1rem;
    margin-top: 2rem;
}

.deck .card {
    box-shadow: 0 0 10px #00000011;
    border-radius: 0.5rem;
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 2rem 1rem;
}

.deck .card a {
    text-decoration: none;
    color: #222;
    text-align: center;
}

.deck .card img {
    margin: 1rem 0;
    height: 100px;
    width: 100px;
    border-radius: 50%;
    object-fit: cover;
}

.deck .card h4 {
    font-weight: 500;
}

.deck .card p {
    font-size: 0.8rem;
    font-weight: 500;
    margin-top: 5px;
    color: #2FA634;
}

.deck .card .request,
.deck .card .reject {
    display: block;
    padding: 0.5rem 1rem;
    background: #2FA634;
    color: white;
    margin-top: 1rem;
    font-size: 1rem;
    border-radius: 10px;
}

.deck .card .request.cancel {
    background: #555;
}

.deck .card .reject {
    background: #555;
}

.requests {
    margin-top: 4rem;
}

.requests .request {
    width: 90%;
    margin: auto;
    box-shadow: 0 0 5px #00000033;
    padding: 1rem;
    display: grid;
    grid-template-columns: 40px auto max-content max-content;
    gap: 1rem;
    align-items: center;
}

.requests .request .profpic {
    height: 40px;
    width: 40px;
}

.requests .request .profpic img {
    height: 40px;
    width: 40px;
    border-radius: 50%;
    outline: 3px solid #2FA634;
    object-fit: cover;
}

.requests .request a {
    font-weight: 800;
    color: #222;
    text-decoration: none;
    font-family: 'Raleway', sans-serif;
}

.requests .request .btn {
    padding: 0.5rem 1rem;
    background: #555;
    color: white;
    border-radius: 5px;
}

.requests .request .btn.accept {
    background: #2FA634;
}

@media screen and (max-width:769px) {
    .deck .card {
        display: grid;
        grid-template-rows: 1rem 1fr 1fr 1rem;
        grid-template-columns: 1fr 2fr 2fr;
        min-height: 80px;
        padding: 0 1rem;
    }

    .deck .card img {
        height: 50px;
        width: 50px;
        grid-row: 1/5;
    }

    .deck .card h4 {
        grid-column: 2;
        grid-row: 2;
    }

    .deck .card p {
        grid-column: 2;
        grid-row: 3;
    }

    .deck .card button {
        grid-column: 3;
        grid-row: 2/4;
        font-size: 0.8rem;
    }
}
</style>