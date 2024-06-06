<template>
    <div class="chat card">
        <div class="scrollable card-body" ref="hasScrolledToBottom">
            <template v-for="message in messages" :key="message.id">
                <div class="message message-receive" v-if="user.id != message.user.id">
                    <p>
                        <strong class="primary-font">
                            {{ message.user.name }} :
                        </strong>
                        {{ message.message }}
                    </p>
                </div>
                <div class="message message-send" v-else>
                    <p>
                        <strong class="primary-font">
                            {{ message.user.name }} :
                        </strong>
                        {{ message.message }}
                    </p>
                </div>
            </template>
        </div>

        <div class="chat-form input-group">
            <input id="btn-input" type="text" name="message" class="form-control input-sm message-" placeholder="Type your message here..." v-model="newMessage" @keyup.enter="addMessage">

            <span class="input-group-btn">
                <button class="btn btn-primary" id="btn-chat" @click="addMessage">
                    Send
                </button>
            </span>
        </div>
    </div>
</template>

<script>
import { ref, onMounted, onUpdated } from 'vue';
import axios from 'axios';
import Echo from 'laravel-echo';

export default {
    props: ['user'],
    setup(props) {
        const messages = ref([]);
        const newMessage = ref('');
        const hasScrolledToBottom = ref(null);

        onMounted(() => {
            fetchMessages();
            setupEchoListener();
        });

        onUpdated(() => {
            scrollBottom();
        });

        const fetchMessages = async () => {
            try {
                const response = await axios.get('/messages');
                messages.value = response.data;
            } catch (error) {
                console.error("Error fetching messages:", error);
            }
        };

        const addMessage = async () => {
            const userMessage = {
                user: props.user,
                message: newMessage.value
            };
            messages.value.push(userMessage);
            try {
                const response = await axios.post('/messages', userMessage);
                console.log(response.data);
                newMessage.value = '';
            } catch (error) {
                console.error("Error sending message:", error);
            }
        };

        const scrollBottom = () => {
            if (messages.value.length > 1) {
                const el = hasScrolledToBottom.value;
                el.scrollTop = el.scrollHeight;
            }
        };

        const setupEchoListener = () => {
            Echo.private('chat-channel')
                .listen('SendMessage', (e) => {
                    messages.value.push({
                        message: e.message.message,
                        user: e.user
                    });
                });
        };

        return {
            messages,
            newMessage,
            addMessage,
            fetchMessages,
            hasScrolledToBottom
        };
    }
};
</script>
