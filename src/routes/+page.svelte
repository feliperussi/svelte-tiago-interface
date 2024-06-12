<script>
  import { onMount } from 'svelte';
  import ROSLIB from 'roslib';

  let ros;
  let linearX = 0;
  let angularZ = 0;
  let connectionStatus = 'Connecting...';
  let intervalId;
  let ttsText = '';
  let ttsFeedback = '';
  let ttsResult = '';

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

  function sendCmdVel() {
    const cmdVelTopic = new ROSLIB.Topic({
      ros: ros,
      name: '/input_joy/cmd_vel',
      messageType: 'geometry_msgs/Twist'
    });

    const twist = new ROSLIB.Message({
      linear: {
        x: linearX,
        y: 0,
        z: 0
      },
      angular: {
        x: 0,
        y: 0,
        z: angularZ
      }
    });

    cmdVelTopic.publish(twist);
  }

  function handlePressStart(direction) {
    switch (direction) {
      case 'up':
        linearX = 1;
        break;
      case 'down':
        linearX = -1;
        break;
      case 'left':
        angularZ = 1;
        break;
      case 'right':
        angularZ = -1;
        break;
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

  // TTS related functionality
  function sendTTSMessage() {
    const ttsClient = new ROSLIB.ActionClient({
      ros: ros,
      serverName: '/tts',
      actionName: 'pal_interaction_msgs/TtsAction'
    });

    const goal = new ROSLIB.Goal({
      actionClient: ttsClient,
      goalMessage: {
        rawtext: {
          text: ttsText,
          lang_id: 'en_US'
        }
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
</script>

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
      <div class="key" 
        on:mousedown={() => handlePressStart('up')} 
        on:mouseup={handlePressEnd} 
        on:touchstart={() => handlePressStart('up')} 
        on:touchend={handlePressEnd}>▲
      </div>
    </div>
    <div class="keypad-row">
      <div class="key" 
        on:mousedown={() => handlePressStart('left')} 
        on:mouseup={handlePressEnd} 
        on:touchstart={() => handlePressStart('left')} 
        on:touchend={handlePressEnd}>◄
      </div>
      <div class="key" 
        on:mousedown={() => handlePressStart('down')} 
        on:mouseup={handlePressEnd} 
        on:touchstart={() => handlePressStart('down')} 
        on:touchend={handlePressEnd}>▼
      </div>
      <div class="key" 
        on:mousedown={() => handlePressStart('right')} 
        on:mouseup={handlePressEnd} 
        on:touchstart={() => handlePressStart('right')} 
        on:touchend={handlePressEnd}>►
      </div>
    </div>
  </div>
  <div class="tts-container">
    <input type="text" bind:value={ttsText} placeholder="Enter TTS text" />
    <button on:click={sendTTSMessage}>Send TTS</button>
  </div>
</div>
