# ROS Demo Interface for Controlling the Tiago Robot Using Svelte-Based Web Application

This demo provides a web-based interface using Svelte to control a TIAGo robot and send Text-To-Speech (TTS) messages via ROS (Robot Operating System). The interface allows users to control the robot's movements and input text to be spoken by the robot.

## Prerequisites

- **ROS (Robot Operating System)**: Ensure ROS is installed and running on your robot.
- **ROS Bridge Suite**: For connecting ROS with web interfaces.
- **Svelte**: A front-end framework for building fast web applications.
- **ROSLIB.js**: A JavaScript library for ROS.

## Setup

### 1. Clone the Repository

First, clone the repository to your local machine:

```bash
git clone https://github.com/feliperussi/svelte-tiago-interface
cd svelte-tiago-interface
```

### 2. Install Dependencies

Install the necessary dependencies:

```bash
npm install
npm install roslib
```

### 3. Modify the IP Address

To ensure the web interface can communicate with the TIAGo robot, you need to modify the IP address in the Svelte script:

1. Connect the device running the Svelte application to the pmb2-58c (the robot's Wi-Fi).

2. Check the IP address of the TIAGo robot:
   ```bash
   hostname -I
   ```
3. Replace the IP address in the `url` field of the ROSLIB connection in your Svelte script (`src/routes/+page.svelte`) with the IP address obtained from the above command and use port `9090`.

## Running the Project

### 1. Start the Development Server

Once you've modified the IP address and installed dependencies, start a development server:

```bash
npm run dev -- --host
```

### 2. Open the Web Interface

Open your web browser and navigate to the address provided by the Svelte development server showed in the terminal.

## Usage

### Controlling the Robot

Use the arrow keys in the web interface to control the robot's movement. The interface sends velocity commands to the ROS topic `/input_joy/cmd_vel`.

### Sending TTS Messages

Enter the text you want the robot to speak in the input field and click the "Send TTS" button. The interface sends TTS commands to the ROS action server `/tts`. Feedback and results are displayed below the button.

## Code Explanation

### ROS Initialization

The ROS connection is initialized and monitored for connection status:

```javascript
let ros;
onMount(() => {
  ros = new ROSLIB.Ros({
    url: 'ws://10.68.0.1:9090'  // Replace with your robot's IP
  });

  ros.on('connection', () => {
    console.log('Connected to websocket server.');
    connectionStatus = 'Connected';
  });

  ros.on('error', (error) => {
    console.log('Error connecting to websocket server: ', error);
    connectionStatus = 'Error';
  });

  ros.on('close', () => {
    console.log('Connection to websocket server closed.');
    connectionStatus = 'Closed';
  });
});
```

### Robot Movement Controls

Functions to control the robot's movements using ROS topics:

```javascript
function sendCmdVel() {
  const cmdVelTopic = new ROSLIB.Topic({
    ros: ros,
    name: '/input_joy/cmd_vel',
    messageType: 'geometry_msgs/Twist'
  });

  const twist = new ROSLIB.Message({
    linear: { x: linearX, y: 0, z: 0 },
    angular: { x: 0, y: 0, z: angularZ }
  });

  cmdVelTopic.publish(twist);
}

function handlePressStart(direction) {
  switch (direction) {
    case 'up': linearX = 1; break;
    case 'down': linearX = -1; break;
    case 'left': angularZ = 1; break;
    case 'right': angularZ = -1; break;
  }
  sendCmdVel();
  intervalId = setInterval(sendCmdVel, 10); 
}

function handlePressEnd() {
  clearInterval(intervalId);
  linearX = 0;
  angularZ = 0;
  sendCmdVel();
}
```

### TTS Functionality

Functions to send TTS messages using ROS actions and update the feedback and result:

```javascript
function sendTTSMessage() {
  const ttsClient = new ROSLIB.ActionClient({
    ros: ros,
    serverName: '/tts',
    actionName: 'pal_interaction_msgs/TtsAction'
  });

  const goal = new ROSLIB.Goal({
    actionClient: ttsClient,
    goalMessage: {
      rawtext: { text: ttsText, lang_id: 'en_US' }
    }
  });

  goal.on('feedback', (feedback) => {
    ttsFeedback = JSON.stringify(feedback);
    console.log('Feedback: ', feedback);
  });

  goal.on('result', (result) => {
    ttsResult = JSON.stringify(result);
    console.log('Result: ', result);
  });

  goal.send();
}
```

### HTML Structure and Styles

```html
<style>
  .status {
    font-size: 1.5em;
    margin-bottom: 20px;
  }
  .keypad-container {
    display: flex;
    flex-direction: column;
    align-items: center;
  }
  .keypad-row {
    display: flex;
    justify-content: center;
    margin: 5px 0;
  }
  .key {
    width: 50px;
    height: 50px;
    margin: 5px;
    border: 1px solid #000;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    user-select: none;
  }
  .key:active {
    background-color: #ccc;
  }
  .tts-container {
    margin-top: 20px;
  }
  input {
    padding: 5px;
    margin-right: 10px;
  }
  button {
    padding: 5px 10px;
  }
</style>

<div>
  <p class="status">{connectionStatus}</p>
  <div class="keypad-container">
    <div class="keypad-row">
      <div class="key" on:mousedown={() => handlePressStart('up')} on:mouseup={handlePressEnd} on:touchstart={() => handlePressStart('up')} on:touchend={handlePressEnd}>▲</div>
    </div>
    <div class="keypad-row">
      <div class="key" on:mousedown={() => handlePressStart('left')} on:mouseup={handlePressEnd} on:touchstart={() => handlePressStart('left')} on:touchend={handlePressEnd}>◄</div>
      <div class="key" on:mousedown={() => handlePressStart('down')} on:mouseup={handlePressEnd} on:touchstart={() => handlePressStart('down')} on:touchend={handlePressEnd}>▼</div>
      <div class="key" on:mousedown={() => handlePressStart('right')} on:mouseup={handlePressEnd} on:touchstart={() => handlePressStart('right')} on:touchend={handlePressEnd}>►</div>
    </div>
  </div>
  <div class="tts-container">
    <input type="text" bind:value={ttsText} placeholder="Enter TTS text" />
    <button on:click={sendTTSMessage}>Send TTS</button>
    <p>Feedback: {ttsFeedback}</p>
    <p>Result: {ttsResult}</p>
  </div>
</div>
```
