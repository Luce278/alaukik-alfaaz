---
{"dg-publish":true,"permalink":"/assets-chatbot/chatbot-js/"}
---



document.addEventListener('DOMContentLoaded', () => {
    const openChatBtn = document.getElementById('openChatBtn');
    const closeChatBtn = document.getElementById('closeChatBtn');
    const chatContainer = document.getElementById('chatContainer');
    const chatMessages = document.getElementById('chatMessages');
    const chatInput = document.getElementById('chatInput');
    const sendMessageBtn = document.getElementById('sendMessageBtn');

    openChatBtn.addEventListener('click', () => {
        chatContainer.style.display = 'flex';
        openChatBtn.style.display = 'none';
    });

    closeChatBtn.addEventListener('click', () => {
        chatContainer.style.display = 'none';
        openChatBtn.style.display = 'block';
    });

    sendMessageBtn.addEventListener('click', sendMessage);
    chatInput.addEventListener('keypress', (e) => {
        if (e.key === 'Enter') {
            sendMessage();
        }
    });

    async function sendMessage() {
        const userMessage = chatInput.value.trim();
        if (userMessage === '') return;

        addMessageToChat('user', userMessage);
        chatInput.value = '';

        addMessageToChat('bot', 'Typing...', true);

        try {
            // Find and extract the poem content from the current page
            const poemContent = document.querySelector('.poem-content') ? document.querySelector('.poem-content').innerText : 'No specific poem found on this page.';

            const response = await fetch('/api/chat', { // Call your Vercel serverless function
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({
                    userMessage: userMessage,
                    poemContext: poemContent
                }),
            });

            const data = await response.json();
            removeTypingIndicator();

            if (response.ok) {
                addMessageToChat('bot', data.message);
            } else {
                addMessageToChat('bot', data.message || 'An error occurred.');
            }
        } catch (error) {
            console.error('Error with chatbot:', error);
            removeTypingIndicator();
            addMessageToChat('bot', 'An error occurred while fetching the response.');
        } finally {
            chatMessages.scrollTop = chatMessages.scrollHeight;
        }
    }

    // Helper functions
    function addMessageToChat(sender, message, isTypingIndicator = false) {
        const messageElement = document.createElement('p');
        messageElement.classList.add(sender + '-message');
        messageElement.innerHTML = message;
        if (isTypingIndicator) {
            messageElement.id = 'typingIndicator';
        }
        chatMessages.appendChild(messageElement);
        chatMessages.scrollTop = chatMessages.scrollHeight;
    }

    function removeTypingIndicator() {
        const typingIndicator = document.getElementById('typingIndicator');
        if (typingIndicator) {
            typingIndicator.remove();
        }
    }
});
