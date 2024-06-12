document.addEventListener("DOMContentLoaded", function() {
    const pages = document.querySelectorAll('.page');
    const nextPageButton = document.getElementById('nextPage');
    const backPage1Button = document.getElementById('backPage1');
    const backPage1FromPage3Button = document.getElementById('backPage1FromPage3');
    const loginButton = document.getElementById('login');
    const sendTimeButton = document.getElementById('sendTime');
    const sendCalibButton = document.getElementById('sendCalib');
    const leftArrowButton = document.getElementById('leftArrow');
    const rightArrowButton = document.getElementById('rightArrow');

    let currentPageIndex = 0;
    let websocket;
    let macID, temperature;

    function showPage(index) {
        pages.forEach((page, i) => {
            page.classList.toggle('visible', i === index);
        });
    }

    function initWebSocket() {
        websocket = new WebSocket(`ws://${window.location.hostname}/ws`);
        websocket.onopen = onOpen;
        websocket.onclose = onClose;
        websocket.onmessage = onMessage;
    }

    function onOpen(event) {
        console.log("WebSocket connection opened.");
    }

    function onClose(event) {
        console.log("WebSocket connection closed. Reconnecting...");
        setTimeout(initWebSocket, 2000);
    }

    function onMessage(event) {
        const data = event.data;
        console.log(data);

        if (data.startsWith("macID:")) {
            macID = data;
            document.getElementById('macID1').textContent = macID;
            document.getElementById('macID2').textContent = macID;
            document.getElementById('macID3').textContent = macID;
        } else if (data.startsWith("temperature:")) {
            temperature = data.split(':')[1];
            document.getElementById('temperature').textContent = temperature;
        } else if (data === "hello") {
            document.getElementById('tempTitle').textContent = data;
        }
    }

    function sendTime() {
        const hour = document.getElementById('hour').value;
        const minute = document.getElementById('minute').value;
        const second = document.getElementById('second').value;
        const day = document.getElementById('day').value;
        const month = document.getElementById('month').value;
        const year = document.getElementById('year').value;

        const timeData = `time:${hour}:${minute}:${second}:${day}:${month}:${year}`;
        websocket.send(timeData);
    }

    function sendCalib() {
        const calib = document.getElementById('calib').value;
        websocket.send(`calib:${calib}`);
    }

    function handleLogin() {
        const username = document.getElementById('username').value;
        const password = document.getElementById('password').value;

        if (username === "admin" && password === "admin") {
            currentPageIndex = 2;
            showPage(currentPageIndex);
        } else {
            alert("Invalid username or password");
        }
    }

    function sendArrow(direction) {
        websocket.send(`arrow:${direction}`);
    }

    nextPageButton.addEventListener('click', () => {
        currentPageIndex = 1;
        showPage(currentPageIndex);
    });

    backPage1Button.addEventListener('click', () => {
        currentPageIndex = 0;
        showPage(currentPageIndex);
    });

    backPage1FromPage3Button.addEventListener('click', () => {
        currentPageIndex = 0;
        showPage(currentPageIndex);
    });

    loginButton.addEventListener('click', handleLogin);
    sendTimeButton.addEventListener('click', sendTime);
    sendCalibButton.addEventListener('click', sendCalib);
    leftArrowButton.addEventListener('click', () => sendArrow('left'));
    rightArrowButton.addEventListener('click', () => sendArrow('right'));

    showPage(currentPageIndex);
    initWebSocket();
});
